---
layout: post
title:  "maven与gradle互转"
categories: build
tags:  build maven gradle
---

* content
{:toc}

## maven to gradle
> gradle init --type pom

## gradle to maven

gradle转maven，在build.gradle中添加maven插件
> apply plugin: 'maven' 

然后在gradle窗口的任务（Tasks）下的other中双击install，或者直接在Terminal中敲命令gradle install完成转换

## gradle使用镜像
在 USER_HOME/.gradle/ 下面创建新文件 init.gradle，输入下面的内容并保存。
```groovy
allprojects{
    repositories {
        def REPOSITORY_URL = 'http://maven.aliyun.com/nexus/content/groups/public/'
        all { ArtifactRepository repo ->
            if(repo instanceof MavenArtifactRepository){
                def url = repo.url.toString()
                if (url.startsWith('https://repo1.maven.org/maven2') || url.startsWith('https://jcenter.bintray.com/')) {
                    project.logger.lifecycle "Repository ${repo.url} replaced by $REPOSITORY_URL."
                    remove repo
                }
            }
        }
        maven {
            url REPOSITORY_URL
        }
    }
}

```
