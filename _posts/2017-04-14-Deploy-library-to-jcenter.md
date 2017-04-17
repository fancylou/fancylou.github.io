---
layout: post
title: "发布android的library包到Jcenter"
categories: kotlin,android
date: 2017-04-17 23:33:00

---

Android开发的时候很会使用到很多优秀的开源库，gradle引入这些包的时候，大部分都是从Jcenter这个仓库来的，这个Jcenter库是由Bintray在维护，如果想自己开发一个开源库或者开源组件，可以打包发布到Jcenter供大家使用，方便自己也方便大家。

想发布包到Jcenter，先的去[Bintray](https://bintray.com/)注册账户，这里注意注册一个开源账户，前面那个免费账户要什么组织什么的，我也没搞懂，反正就是一直没传成功！

![](http://muliba.u.qiniudn.com/blog/post/bintray.jpg)

注册登录之后，看下图，两个事情，知道API key在哪里、创建一个maven仓库

![](http://muliba.u.qiniudn.com/blog/post/bintray_index.png)

## API Key

点击用户名下面的Edit按钮，可以进入个人信息界面，左边菜单的最后一个选项API Key就能查到你的key，这个key后续上传你的包的时候会需要用到的。

![](http://muliba.u.qiniudn.com/blog/post/bintray_api_key.png)

## 创建maven仓库

点击上上图圈出来的按钮**Add New Repository** ，进入创建界面，类型选择maven，填写你的库名称，然后保存

![](http://muliba.u.qiniudn.com/blog/post/bintray_new_repository.png)

这样Bintray网站上的事情就告一段落了。



## Gradle 插件

在你要发布的项目的根gradle文件中添加两个插件：

```json
classpath 'com.github.dcendents:android-maven-gradle-plugin:1.5'
        classpath 'com.jfrog.bintray.gradle:gradle-bintray-plugin:1.6'
```

一个是为了给Android的项目生成maven标准用的，一个是上传文件到bintray服务器用的

建一个**bintrayUpload.gradle**文件，内容：

```json
apply plugin: 'com.github.dcendents.android-maven'
apply plugin: 'com.jfrog.bintray'

// load properties 
Properties properties = new Properties()
properties.load(project.rootProject.file('local.properties').newDataInputStream())

// read properties
//项目名称
def projectName = properties.getProperty("project.name")
//项目的groupId
def projectGroupId = properties.getProperty("project.groupId")
//项目的artifactId，这里需要跟项目的library名称一致，不然传到bintray的包和名字对不上会找不到
def projectArtifactId = properties.getProperty("project.artifactId")
//library的版本号
def projectVersionName = properties.getProperty("project.version")
//网站地址
def projectSiteUrl = properties.getProperty("project.siteUrl")
//开源的git地址
def projectGitUrl = properties.getProperty("project.gitUrl")
//项目库的描述
def projectDesc = properties.getProperty("project.desc")
//开发者标识ID
def developerId = properties.getProperty("developer.id")
//开发者名称
def developerName = properties.getProperty("developer.name")
//开发者邮箱
def developerEmail = properties.getProperty("developer.email")
//这个就是刚才注册的bintray用户名
def bintrayUser = properties.getProperty("bintray.user")
//这个是前面提到的bintray上面的API Key
def bintrayApikey = properties.getProperty("bintray.apikey")

group = projectGroupId

// This generates POM.xml with proper parameters
install {
    repositories.mavenInstaller {
        pom {
            project {
                packaging 'aar'
                groupId projectGroupId
                artifactId projectArtifactId

                name projectName
                description projectDesc
                version projectVersionName
                url projectSiteUrl
                licenses {
                    license {
                        name 'The Apache Software License, Version 2.0'
                        url 'http://www.apache.org/licenses/LICENSE-2.0.txt'
                    }
                }
                developers {
                    developer {
                        id developerId
                        name developerName
                        email developerEmail
                    }
                }
                scm {
                    connection projectGitUrl
                    developerConnection projectGitUrl
                    url projectSiteUrl
                }
            }
        }
    }
}


version = projectVersionName

// 命令 java -Djava.ext.dirs=. -jar dokka-fatjar.jar ./src/main/java -format javadoc -output ./build/doc

task dokkaJavadoc(type: org.gradle.api.tasks.Exec) {
    workingDir '.'
    println "WorkingDir: $workingDir"
    ext.toolJarPath = "$workingDir"
    ext.dokkaJarPath = "$workingDir"+File.separator+"dokka-fatjar.jar"
    ext.sourceDirs = "$workingDir"+File.separator+"src"+File.separator+"main"+File.separator+"java"
    ext.outputFormat = 'javadoc'
    ext.outputDirectory = "$buildDir" + File.separator+ "doc"

    /**
     *
     * 这里输出格式可以为: html , markdown, jekyll, javadoc
     * 如果是 javadoc 格式, 他会用到 javadoc 的库,
     * 如果你的 PATH 没有包含 JDK x.x.x/lib 路径的话, 就会报 'java.lang.ClassNotFoundException: com.sun.javadoc.DocErrorReporter' 异常
     * 所以需要你主动将这个路径加进来, 或者将 JDK x.x.x/lib/tools.jar 文件拷贝出来, 下面这个命令我就是拷贝到了当前目录
     *
     * dokka-fatjar.jar 这个jar就是从 dokka 项目上下载下来的
     */
    commandLine "cmd" , "/c", "java -Djava.ext.dirs=$toolJarPath -jar $dokkaJarPath $sourceDirs -format $outputFormat -output $outputDirectory"

    /**
     * 如果你是 Linux 系统, 就用这个
     */
//    commandLine "java -Djava.ext.dirs=$toolJarPath -jar $dokkaJarPath $sourceDirs -format $outputFormat -output $outputDirectory"

}

/**
 * 这里将生成的文档打包成 xxxx-javadoc.jar
 */
task kotlinDocJar(type: Jar, dependsOn: dokkaJavadoc) {
    classifier = 'javadoc'
    from dokkaJavadoc.outputDirectory
}

/**
 * 这里将源码打包成 xxx-sources.jar
 */
task sourcesJar(type: Jar) {
    classifier = 'sources'
    from android.sourceSets.main.java.srcDirs
}


artifacts {
    archives kotlinDocJar
    archives sourcesJar
}

// bintray configuration
bintray {
    user = bintrayUser
    key = bintrayApikey
    configurations = ['archives']
    pkg {
        repo = "maven"
        name = projectName
        desc = projectDesc
        websiteUrl = projectSiteUrl
        vcsUrl = projectGitUrl
        licenses = ["Apache-2.0"]
        publish = true
        publicDownloadNumbers = true
    }
}
```

然后把这个文件引入到你要打包的模块的gradle文件内：

```json
apply from: "../bintrayUpload.gradle"
```

然后用Android Studio上gradle窗口执行两个任务

#### install

![](http://muliba.u.qiniudn.com/blog/post/gradle1.png)

#### bintrayUpload

![](http://muliba.u.qiniudn.com/blog/post/gradle2.png)

这样就传到bintray的服务器上了。 

最后一步是到bintray的网站上，将你maven仓库里面的包同步到Jcenter，等Jcenter通过了审核后，就可以在其他项目中使用你的包了。

![](http://muliba.u.qiniudn.com/blog/post/bintray_sync_jcenter.png)





**PS：**这里重点说下bintrayUpload.gradle中的关于`kotlinDocJar`任务，因为我的library是用kotlin写的，于是就一直生成步了javadoc，没有javadoc，Jcenter就不肯通过审核。这里要感谢[Kejin](https://liungkejin.github.io/)，在网上搜到了他的博客，才知道如何生成这个javadoc，[文章链接](https://liungkejin.github.io/2016/04/12/Publish-Kotlin-Lib-To-Jcenter.html)

