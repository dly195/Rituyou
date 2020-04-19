---
layout: post
title:  "将自己的framework上传至CocoaPods"
date:   2020-03-24 20:30:00
categories: coding
tags: [cocoapods, iOS]

---

#### CocoaPods: The Cocoa dependency manager
CocoaPods是开发iOS或者Mac应用时使用的依赖管理工具，它可以帮助我们很方便地找到第三方框架，并且省去一些额外的工程配置。

我们知道使用cocoapods的一般流程是：

```
1. 安装cocoapod
2. 在需要使用cocoapod的工程目录下使用pod init命令生成podfile文件
3. 在podfile文件中添加需要使用的第三方框架
4. 执行‘pod install’命令，生成.xcworkspace文件和Pods目录
5. 使用.xcworkspace文件打开工程
6. 当有新的框架需要集成时，我们只需要在podfile文件中添加新的框架名称，然后执行‘pod install’或者‘pod update’命令
```
当在我们的设备上第一次执行‘pod install’命令时，你会发现‘终端’会停顿很久，这是因为第一次使用pod的时候，需要从github上拉取所有发布在cocoapods上第三方框架的repo文件。你可以在[github-Specs](https://github.com/cocoapods/Specs)上查看这些文件。

随着第三方框架的不断增多，这些文件时越来越大的，目前已经达到了1个多G。你可以在Finder的`~/.cocoapods/repo/`目录下查看这些文件。

我们的设备在本地储存了这么一份索引文件，当然不可能是没有用的。在我们执行‘pod install’命令添加一个新的框架时，实际上cocoapods干了这些事

```
1. 查看 ~/.cocoapods/repo/master/Specs 是否存在此框架
2. 如果存在，则在其spece文件找到其源文件的git地址
3. 如果不存在，则在[github-Specs]更新spece目录，然后找到对应的spece文件，获取其源文件git地址
4. 通过git地址拉取其相对应版本的release源码
```

所以，如果我们需要在cocoapods上发布我们自己的framework，那么我们只需要一个发布源码的git仓库和一个上传到[github-Specs]的spece文件。
 
### 管理一个git仓库

怎么获取一个git仓库我们就不详述了，你可以使用github，oschina，codingchina等等。
一定不要忘记：
1. 这个仓库必须是公有仓库，否则cocoapods无法访问你的仓库
2. 你需要具备一定的权限，因为你需要push你的代码并且打上对应的tag

怎么生成自己的framework这里也不做详述了，我们直接从上传framework开始。

```text
1. 将你的git仓库克隆到本地，并且将你要上传的framework文件拷贝到该目录下。 
2. 依次执行'git add .' , 'git commit -m'' ', 'git push' 命令将你的framework push到git仓库 
3. 使用'git tag 1.0.0'命令给你的仓库添加tag 
4. 使用'git push --tags'命令将tag push到远程仓库
```
 
### 生成Podspec索引文件

framework源码已经准备好了，我们开始生成Podspec文件。
1. 依然还在你新建的仓库路径下，使用‘pod spec create xxxx’命令就可以本地生成podspec文件
2. 编辑podspec文件，你可以使用任何文本编辑器，甚至是Xcode。

```text
Pod::Spec.new do |s|
  spec.name             = 'xxx'
  spec.version          = '1.0.0'
  spec.summary      = 'xxxxx'
  spec.homepage    = 'xxxxx'
  spec.license          = { :type => 'MIT', :file => 'LICENSE' }
  spec.author           = { 'xxx' => 'xxxx' }
  spec.source           = { :git => '//https://github.com/xxxx.git', :tag => 1.0.0 }
  spec.iospec.deployment_target = '9.0'
  spec.source_files = 'Lib/**/*'
  spec.frameworks = 'UIKit', 'Foundation'

end
```
你看到的podspec文件可能是上面这个样子的，你只需要配置好一些重要信息即可。你可以参照下面的例子

```text
Pod::Spec.new do |spec|
    //框架名称
    spec.name          = ‘dlyLib’
    //版本号，这是我们podfile文件指定的版本号。 框架每次发布版本时都需要打tag标签，tag标签与此版本号必须一致。
    spec.version       = ‘1.0.0'
    //许可证，除非源代码包含了LICENSE.*或者LICENCE.*文件，否则必须指定许可证文件。文件扩展名可以没有，或者是.txt,.md,.markdown。
    spec.license          = { :type => 'MIT', :file => 'LICENSE' }
    //主页
    spec.homepage      = 'https://github.com/dly195’      
    //框架作者信息
    spec.authors       = { 'dly' => 'dly195@gmail.com’ } 
         //pod简介
    spec.summary = 'dlyLib'
    //详细描述
    spec.description = <<-DESC
      dlyLib是我的dly的framework。
                        DESC    
    //获取库的地址
    spec.source = { :git => 'https://github.com/dly195/dlyLib',:tag => spec.version.to_s }
    //支持iOS版本
    spec.iospec.deployment_target = '9.0'
    //你要上传的源文件的路径
    spec.source_files = 'Lib/Classes/**/*'
    //头文件目录
    spec.public_header_files = 'Lib/Classes/**/*.h'    
    //你要所依赖的第三方框架，在这里也就是你的frameworouk。如果你只上传framework并且，你的framework已经暴露了足够的头文件，你就不需要再spec.source_files或spec.public_header_files重新暴露这些头文件
    spec.vendored_frameworks = 'lib/dlyLib.framework'
    //你所依赖的系统框架
    spec.frameworks = 'UIKit', 'QuartzCore', 'ImageIO', 'CoreVideo', 'CoreMedia', 'CoreGraphics', 'AVFoundation', 'AssetsLibrary'
     //你所依赖的静态库，注意在Xcode中他们都是Lib开头的，在这里并不需要
    spec.libraries = 'sqlite3','c++abi',  'stdc++', 'z', 'c++', 'resolv'
    //你所依赖的第三方静态库，会帮你一起上传上去
    spec.vendored_libraries = 'lib/libcrypto/libcrypto.a'
    //Xcode需要的其他设置
    spec.xcconfig = { 'OTHER_LDFLAGS' => '-ObjC' }

```   
你可能会需要的一些路径说明

```text
*   匹配所有文件
c*  匹配所有以c开头的文件
*c  匹配所有以c结尾的文件
*c* 匹配所有包含c的文件
**  递归匹配所有子文件夹
{h, m} 匹配h或m
？ 匹配任何一个字符
```  
如果你需要更详细的podspec文件，你可以在网上找到详细的教程

### push到cocoapods

完成podspec文件之后，我们就可以着手把它发布出去了。
首先我们需要验证我们的podspec文件。
你可以使用‘pod lib lint’命令进行本地验证，当出现‘xxxx.podspec passed validation’就说明验证通过了。

如果你发现出现了很多waring，但它们不影响使用。你可以使用‘pod lib lint --allow-warnings’忽略这些waring。
如果出现了错误，你想看错误的详情，可以使用‘pod spec lint --verbose’来查看详情。

然后我们使用‘pod spec lint’命令来进行远程验证，这一步有可能会花费一些时间，请耐心等待。

等我们都验证通过了，就可以着手将podspec文件push到cocoapods上了。

```
1. 首先我们需要为设备添加trunk。 使用‘pod trunk register 邮箱 名称 --description="描述"’命令进行注册
2. 到你的邮箱中点击验证链接。
3. 可以使用‘pod trunk me’查看设备的trunk信息
```
当你更换设备之后可以使用相同的邮箱和名称重新注册trunk，并且需要重新验证邮箱。

然后我们使用‘pod trunk push xxxx.podspec --allow-warnings’来进行podspec文件push到cocoapod。这一步也可能会花费一些时间，请耐心等待。
当终端出现xxxx successfully published 的时候，恭喜你已经成功将你的framework push到cocoapods了。


##### some问题

* 这时候如果你马上使用‘pod search xxx’可能会找不到你自己的库，这时候你可以尝试删除~/Library/Caches/CocoaPods目录下的search_index.json文件，不要担心，pod search之后cocoapods会重建这个文件

```
1. 终端输入rm ~/Library/Caches/CocoaPods/search_index.json
2. 删除成功后再执行pod search
```
* 如果使用 pod install 无法找到你发布的框架,那么使用 pod update 更新本地库
* 如果你已经使用pod install成功集成了你的框架，但是pod search还是搜索不到，删除search_index.json文件也不起作用，那么你可以到[cocoapods.org](https://cocoapods.org/)上查看是否已经收录了你的框架，有时新发布的框架可能需要一天时间来被收录

##### 更新你的框架
如果你需要更新你在cocoapods上的框架，那么你只需要做以下几步

```
1. 更新的你框架并push到远程仓库
2. 打一个新的tag 并push
3. 修改podspec文件的版本号和tag一致
4. 验证podspec文件文件
5. push podspec文件到cocoapods
```






