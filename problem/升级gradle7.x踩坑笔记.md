# 升级Gradle7.x版本踩坑记录

最近公司为了解决第三方组建的安全性问题，所以升级了gradle的版本。谁懂啊，上午构建项目还正常，一觉醒来版本升级了，全是机器人报错提醒的消息。无奈只能一个一个项目的改，说多了都是泪，我们进入正题。

之前的gradle版本一直使用的是6.9.1，目前升级到了7.6.3。可谓是大版本升级，一步一步踩坑得到了如下问题的处理方式。

## 安全性问题

7.x版本以上的gradle默认是不允许使用http链接的，需要使用安全的https进行访问。如果确实需要使用http进行访问，需要进行如下处理：

1. maven仓库的配置位置增加`allowInsecureProtocl = true`，如下代码段所示：

```groovy
// build.gradle 顶部插件仓库使用
buildscript {
    repositories {
        maven {
            url mavenUrl
            credentials {
                username mavenUsername
                password mavenPassword
            }
            allowInsecureProtocol = true     // 新增
        }
    }
    dependencies {
        // ...此处省略若干字
    }
}

// gradle publish 发布制品到仓库使用
publishing {
    // ...此处省略若干字
    repositories {
        maven {
            url = mavenReleaseUrl
            if (version.endsWith('-SNAPSHOT')){
                url = mavenSnapshotUrl
            }
            credentials {
                username = mavenUsername
                password = mavenPassword
            }
            allowInsecureProtocol = true     // 新增
        }
    }
}

// gradle build 构建拉取制品使用
repositories {
    maven {
        url mavenUrl
        credentials {
            username mavenUsername
            password mavenPassword
        }
        allowInsecureProtocol = true     // 新增
    }
}

```

2. `apply from`中使用http，需要进行如下配置：

```groovy
apply from: resources.text.fromInsecureUri('http://abd.com/xxxx.gradle')
```

## 废弃compile和runtime相关方法

7.x版本废弃了compile和runtime相关api方法，可以参考如下表格进行相应的关键字替换

| 被废弃             | 新的                      |
| ------------------ | ------------------------- |
| compile            | api or implementation     |
| runtime            | runtimeOnly               |
| testRuntime        | testRuntimeOnly           |
| testCompile        | testImplementation        |
| <sourceSet>Runtime | <sourceSet>RuntimeOnly    |
| <sourceSet>Compile | <sourceSet>Implementation |

## 废弃旧的maven插件

maven插件已经被移除，需要使用maven-publish插件代替

```
修改前：
apply plugin: 'maven'

修改后：
apply plugin: 'maven-publish'
```

## 废弃uploadArchives任务

uploadArchives 是针对旧版本的 lvy 和 maven 发布机制强关联的，随着两种发布机制的废弃，uploadArchives也被废弃了；如果你是和我一样也是使用的maven进行版本发布，可以参考如下代码段：

```groovy
//修改前：
uploadArchives {
    repositories {
        mavenDeployer {
            snapshotRepository(url: mavenSnapshotUrl) {
                authentication(userName: nexusUsername, password: nexusPassword)
            }
            repository(url: mavenReleaseUrl) {
                authentication(userName: nexusUsername, password: nexusPassword)
            }
        }
    }
}
//修改后：
publishing {
    repositories {
        maven {
            url = mavenReleaseUrl
            if (version.endsWith('-SNAPSHOT')) {
                url = mavenSnapshotUrl
            }
            credentials {
                username = nexusUsername
                password = nexusPassword
            }
        }
    }
}
```

因为本身公司使用的旧版本已经是6.9.4的版本，所以没有太多的改动点。如果你碰见了其他问题，可以在gradle官网的升级手册中查找对应的问题以及解决办法

[gradle upgrade version 6.x to 7.x](https://docs.gradle.org/7.0/userguide/upgrading_version_6.html)