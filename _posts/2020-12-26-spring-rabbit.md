---
layout: post
title: springboot集成rabbitmq
categories: springboot rabbitmq
tags: spring
author: leihtg
---



* content
{:toc}

springboot集成了各种第三方组件，让我们使用起来特别方便，于是想一探究竟，看看它是怎么集成的。





springboot在集成各种组件时都会有一个`xxxxAutoConfiguration`，今天要看的是rabbitmq，所以是`RabbitAutoConfiguration`这个类里面配置了连接rabbitmq的工厂和rabbitTemplate等必要组件

```java
@Configuration
@ConditionalOnClass({ RabbitTemplate.class, Channel.class })
@EnableConfigurationProperties(RabbitProperties.class)
@Import(RabbitAnnotationDrivenConfiguration.class)
public class RabbitAutoConfiguration {
	@Configuration
	@ConditionalOnMissingBean(ConnectionFactory.class)
	protected static class RabbitConnectionFactoryCreator{}
    
   	@Configuration
	@Import(RabbitConnectionFactoryCreator.class)
	protected static class RabbitTemplateConfiguration{}
    
   	@Configuration
	@ConditionalOnClass(RabbitMessagingTemplate.class)
	@ConditionalOnMissingBean(RabbitMessagingTemplate.class)
	@Import(RabbitTemplateConfiguration.class)
	protected static class MessagingTemplateConfiguration
}
```

这个类没有什么特别之处，就是利用springboot的自动配置，初始化这个类，接着初始化类里面的方法。

比较重要的是类上面 `@Import(RabbitAnnotationDrivenConfiguration.class)`

```java
@Configuration
@ConditionalOnClass(EnableRabbit.class)//这个比较重要
class RabbitAnnotationDrivenConfiguration {
    //这里面依旧是各种配置
}
```

`EnableRabbit`类里面 `@Import(RabbitBootstrapConfiguration.class)` 导入的这个类配置了两个后置处理器：

```java
@Configuration
public class RabbitBootstrapConfiguration {

	@Bean(name = RabbitListenerConfigUtils.RABBIT_LISTENER_ANNOTATION_PROCESSOR_BEAN_NAME)
	@Role(BeanDefinition.ROLE_INFRASTRUCTURE)
	public RabbitListenerAnnotationBeanPostProcessor rabbitListenerAnnotationProcessor() {
		return new RabbitListenerAnnotationBeanPostProcessor();
	}

	@Bean(name = RabbitListenerConfigUtils.RABBIT_LISTENER_ENDPOINT_REGISTRY_BEAN_NAME)
	public RabbitListenerEndpointRegistry defaultRabbitListenerEndpointRegistry() {
		return new RabbitListenerEndpointRegistry();
	}

}
```

类`RabbitListenerAnnotationBeanPostProcessor`实现了不少接口，因为是后置处理器所以肯定也有`BeanPostProcessor`，看一下实现的方法

```java
	@Override
	public Object postProcessAfterInitialization(final Object bean, final String beanName) throws BeansException {
		Class<?> targetClass = AopUtils.getTargetClass(bean);
		final TypeMetadata metadata = this.typeCache.computeIfAbsent(targetClass, this::buildMetadata);
		for (ListenerMethod lm : metadata.listenerMethods) {
            //找出带有processAmqpListener注解的方法
			for (RabbitListener rabbitListener : lm.annotations) {
				processAmqpListener(rabbitListener, lm.method, bean, beanName);
			}
		}
		if (metadata.handlerMethods.length > 0) {
			processMultiMethodListeners(metadata.classAnnotations, metadata.handlerMethods, bean, beanName);
		}
		return bean;
	}
```

把标记有`RabbitListener`注解的方法封装到类`MethodRabbitListenerEndpoint`里面，然后通过方法`registrar.registerEndpoint(endpoint, factory)`注册到`RabbitListenerEndpointRegistrar`里面。

```java
	public void registerEndpoint(RabbitListenerEndpoint endpoint, RabbitListenerContainerFactory<?> factory) {
//封装endpoint
		AmqpListenerEndpointDescriptor descriptor = new AmqpListenerEndpointDescriptor(endpoint, factory);
		synchronized (this.endpointDescriptors) {
			if (this.startImmediately) { // Register and start immediately
                //把endpoint注册到RabbitListenerEndpointRegistry中，在这个方法里面会把endpoint封装成SimpleMessageListenerContainer
				this.endpointRegistry.registerListenerContainer(descriptor.endpoint,
						resolveContainerFactory(descriptor), true);
			}
			else {
				this.endpointDescriptors.add(descriptor);
			}
		}
	}
```

此时，后置处理器的工作已经做完，静待spring初始化实现完成后调用`SimpleMessageListenerContainer`里面的`start`方法，这个方法里面完成了绑定queue，以及注册回调方法到rabbitmq的channel中。

```java
//SimpleMessageListenerContainer的dostart方法
Set<AsyncMessageProcessingConsumer> processors = new HashSet<AsyncMessageProcessingConsumer>();
//初始化consumer，用于封装rabbitmq传来的消息
for (BlockingQueueConsumer consumer : this.consumers) {
    AsyncMessageProcessingConsumer processor = new AsyncMessageProcessingConsumer(consumer);
    processors.add(processor);
    //线程池内运行
    getTaskExecutor().execute(processor);
    if (getApplicationEventPublisher() != null) {
        getApplicationEventPublisher().publishEvent(new AsyncConsumerStartedEvent(this, consumer));
    }
}
```

我们可以看一下 `AsyncMessageProcessingConsumer`内的run方法

```java
//此方法完成了对channel消息回调的绑定
this.consumer.start();
//进入start方法
for (String queueName : this.queues) {
    if (!this.missingQueues.contains(queueName)) {
        consumeFromQueue(queueName);
    }
}
//consumeFromQueue中
String consumerTag = this.channel.basicConsume(queue, this.acknowledgeMode.isAutoAck(),(this.tagStrategy != null ? this.tagStrategy.createConsumerTag(queue) : ""), this.noLocal,
                                               this.exclusive, this.consumerArgs,
//对consumer的封装      
new ConsumerDecorator(queue, this.consumer, this.applicationEventPublisher));


```

这个方法后面有一个死循环，一直会去队列拿消息

```java
while (isActive(this.consumer) || this.consumer.hasDelivery() || this.consumer.cancelled()) {
    try {
        //一直请求
        boolean receivedOk = receiveAndExecute(this.consumer); // At least one message received
        if (SimpleMessageListenerContainer.this.maxConcurrentConsumers != null) {
            if (receivedOk) {
                if (isActive(this.consumer)) {
                    consecutiveIdles = 0;
                    if (consecutiveMessages++ > SimpleMessageListenerContainer.this.consecutiveActiveTrigger) {
                        considerAddingAConsumer();
                        consecutiveMessages = 0;
                    }
                }
            }
        }
    }
    //.........
}
//如果RabbitListener方法出现异常或者channel关闭，会重启consumer.start
if (!isActive(this.consumer) || aborted) {
    this.consumer.stop();
	restart(this.consumer);
}
```



到此，对rabbitmq的绑定都已完成。

现在看看怎么接收消息的。由于将consumer注入channel中，所以消息来了会第一时间调用consumer中的`handleDelivery`方法，会将消息提交到`BlockingQueueConsumer`队列中

```java
BlockingQueueConsumer.this.queue.offer(new Delivery(consumerTag, envelope, properties, body),BlockingQueueConsumer.this.shutdownTimeout, TimeUnit.MILLISECONDS)
```

在此等着的方法会立即获取到数据并返回

```java
	public Message nextMessage(long timeout) throws InterruptedException, ShutdownSignalException {
		if (logger.isTraceEnabled()) {
			logger.trace("Retrieving delivery for " + this);
		}
		checkShutdown();
		if (this.missingQueues.size() > 0) {
			checkMissingQueues();
		}
        //获取到来的消息
		Message message = handle(this.queue.poll(timeout, TimeUnit.MILLISECONDS));
		if (message == null && this.cancelled.get()) {
			throw new ConsumerCancelledException();
		}
		return message;
	}
```

然后再返回，真正的调用添加`RabbitListener`注解的方法

```java
Message message = consumer.nextMessage(this.receiveTimeout);
if (message == null) {
    break;
}
try {
    //调用我们写的方法
    executeListener(channel, message);
}
```

现在算是对rabbitmq绑定和接收消息的流程走完，其中还有好多细节，我还没有仔细看，以后发现有问题再来补充吧。