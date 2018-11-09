---
layout: post
title: "CocoaPods 发布自定义的开源组件"
categories: IOS
date: 2018-11-06 20:00:00
---


### pod lib create 的命令说明

我们开发应用的时候经常会用到别人开发好的组件，比如Moya网络请求库、HandyJSON阿里巴巴的JSON解析库等等很多很多。我们只需要一个Podfile文件和敲一个`pod install`就可以使用这些开源库了，非常方便。程序员的这种开源精神一直都是受到大家好评的，那既然我们也是程序员，就应用为开源做贡献的嘛。可能你有啥好点子也想开源出来，发布到CocoaPods上供大家使用，那该如何发布自己的库呢？CocoaPods提供了很简单方便的模式。
官方提供了pod lib create这个命令来生成一个cocoapods库的模版，这个是官方的说明：[Using Pod Lib Create](https://guides.cocoapods.org/making/using-pod-lib-create.html) 。

<!-- more -->



```shell
pod lib create MyLib
```
敲了这个命令它会询问几个基本参数：
```shell
What platform do you want to use?? [ iOS / macOS ]  # 你的库用在那个平台上
 > iOS

What language do you want to use?? [ Swift / ObjC ] # 你的库用啥语言开发
 > Swift

Would you like to include a demo application with your library? [ Yes / No ] # 是否包含demo应用
 > Yes

Which testing frameworks will you use? [ Quick / None ] # 是否用测试库
 > None

Would you like to do view based testing? [ Yes / No ] 
 > No

Running pod install on your new library.
```
### 组件库的项目结构
完成了上面操作后就会生成如下结构的的项目：
```
MyLib
  ├── .travis.yml
  ├── _Pods.xcproject
  ├── Example
  │   ├── MyLib
  │   ├── MyLib.xcodeproj
  │   ├── MyLib.xcworkspace
  │   ├── Podfile
  │   ├── Podfile.lock
  │   ├── Pods
  │   └── Tests
  ├── LICENSE
  ├── MyLib.podspec
  ├── MyLib
  │   ├── Assets
  │   └── Classes
  │     └── RemoveMe.swift
  └── README.md
```
你的源代码都放在`MyLib/Classes`目录下，资源放在`MyLib/Assets`下，`Example`就是Demo应用可以测试你写的组件库。
### podspec文件
还有一个很重要的文件：MyLib.podspec，这个文件是配置你的组件库的具体信息的，为了最后CocoaPods管理上传你的组件库用的。
大概内容如下：
```ruby
#
# Be sure to run `pod lib lint MyLib.podspec' to ensure this is a
# valid spec before submitting.
#
# Any lines starting with a # are optional, but their use is encouraged
# To learn more about a Podspec see https://guides.cocoapods.org/syntax/podspec.html
#

Pod::Spec.new do |s|
  s.name             = 'MyLib' #库名称
  s.version          = '0.1.0' #版本号
  s.summary          = '组件库的摘要信息 '

# This description is used to generate tags and improve search results.
#   * Think: What does it do? Why did you write it? What is the focus?
#   * Try to keep it short, snappy and to the point.
#   * Write the description between the DESC delimiters below.
#   * Finally, don't worry about the indent, CocoaPods strips it!

  s.description      = <<-DESC
这里写组织库的具体描述                    
  DESC

  s.homepage         = '组件库的网址'
  # s.screenshots     = 'www.example.com/screenshots_1', 'www.example.com/screenshots_2'
  s.license          = { :type => 'MIT', :file => 'LICENSE' }
  s.author           = { '作者' => '作者邮箱' }
  s.source           = { :git => '组件库git地址.git', :tag => s.version.to_s }
  # s.social_media_url = 'https://twitter.com/<TWITTER_USERNAME>'

  s.ios.deployment_target = '8.0'
  s.swift_version = '4.2' # 这里填写你使用的swift版本
  s.source_files = 'MyLib/Classes/**/*'
  
  # s.resource_bundles = {
  #   'MyLib' => ['MyLib/Assets/*.png']
  # }

  # s.public_header_files = 'Pod/Classes/**/*.h'
  # s.frameworks = 'UIKit', 'MapKit'
  # s.dependency 'AFNetworking', '~> 2.3' # 如何你的库还依赖其他的库就用dependency 这个可以写多个
end

```
### 发布
编写好上面的podspec文件后就可以发布到cocoapods了，发布上去就可以给别人使用了！
按照惯例我们会使用git的tag来作为版本发布，所以我们用版本号来做tag名称。
我们先上传代码到git，然后push到开源仓库。

```shell
git add .
git commit -m "注释"
git push
# 然后打tag
git tag -m "注释" 0.1.0
git push --tags
```
如果是第一次发布组件库到cocoapods，那就先注册一个用户。

```shell
pod trunk register yourmail@gmail.com 'yourName' 
```
注册后会发送一个验证网址到你的邮箱，需要去点击验证一下。
检查是否注册成功：

```shell
pod trunk me
```
最后发布之前用下面的命令检查一下我们的配置文件是否都正确，配置文件里的网址啥的都要能访问到，不然验证不通过：

```shell
pod lib lint
# 后面加上 --no-clean 就能看到那些告警啥的具体信息，方便你改正
```
最后就是发布你的库到cocoapods：
```shell
 pod trunk push MyLib.podspec
```
ok，现在可以给别人使用了。在你的项目Podfile文件里面加入：

```ruby
pod 'MyLib'
```
然后敲一下`pod install` 就可以了。

The End ！