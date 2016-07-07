---
title: 使用cocoapods向第三方提供静态库
date: 2016-07-07 13:33:11
tags: ios
---

最近在尝试将项目的静态SDK库打包成cocoapods，以方便第三方用户使用。由于是第一次打包cocoapods，并且项目自身的代码和编译都不能改变，所以只能通过项目编译出的静态库.a文件和头文件.h文件直接提供，与cocoapods官方的文档上有很大的差异（在这里吐槽下，官方文档实在太简陋了，这么多人使用的项目，上手难度还是觉得挺大的）。所以在这里记录下提供静态库所遇到的问题，方便以后使用。

1. 准备静态库文件`libTest.a` (此处必须是`lib*.a`这样的文件名格式，经过测试，xcode在编译的时候只能链接以`lib`作为prefix的静态库文件，否则会报链接失败)和`Test.h`文件
2. 将文件放入文件夹，文件结构如下

  ```
Test
├── Test.h
└── libTest.a
  ```
3. 新建podspec文件，这里你可以自己手动新建，也可以使用cocoapods提供的命令来新建一个模板，再手动修改
     
 ```
 pod spec create Test
 ```
 新建完成后的文件路径如下
 
 ```
.
├── Test
│   ├── Test.h
│   └── libTest.a
└── Test.podspec
 ```
 修改`Test.podspec`文件，然后将此文件转为json文件（我也不知道为什么需要转成json文件，但是cocoapods的官方库里的spec文件都已经转成了json）， 转成json文件的命令可以通过cocoapods的一个命令完成：
 
 ```
 pod ipc spec Test.podspec
 ```
 最后的结果如下
     
 ``` json
 {
  "name": "Test",
  "version": "1.0.0",
  "summary": "Test",
  "description": "Test",
  "homepage": "https://yeluolei.github.io/",
  "license": {
    "type": "Copyright",
    "text": "Copyright 2016. All rights reserved."
  },
  "authors": {
    "yeluolei": "yeluolei@gmail.com"
  },
  "platforms": {
    "ios": "7.0"
  },
  "source": {
    "git" : "~/Test/"
  },
  "source_files": "Test/Test.h",
  "public_header_files": "Test/Test.h",
  "vendored_libraries": [
    "Test/libTest.a"
  ],
  "frameworks": [
    "JavaScriptCore",
    "Security",
    "CoreLocation",
    "SystemConfiguration",
    "CoreTelephony",
    "CoreGraphics",
    "UIKit",
    "Foundation"
  ],
  "libraries": [
    "z",
    "stdc++"
  ],
  "requires_arc": true
}
 ```
 其中：
 * `source` 指定文件的来源，可以支持git,zip包等格式
 * `source_files` 指定文件需要引入的源代码文件，这里注意只需要包括源代码文件，不需要包括.a文件
 * `public_header_files` 第三方项目可以访问的公开的头文件
 * `vendored_libraries` 我们提供的静态库文件放在此处
 * `frameworks` 静态库依赖的动态库文件
 * `libraries` 静态库依赖的系统库文件
 * `verdered_frameworks` 如果需要的不是静态库，而是动态库.framework文件，则同理可以加在这里
 
4. 将上面的文件全部放入git内，由于是测试使用，所以不需要将git push到远程服务器也可以

 ```
 git init
 git add -A
 git commit -am 'init'
 ```

5. 验证缩写的Test.podsspec.json文件是正确的,这里可以使用cocoapods官方提供的验证命令

 ```
 pod spec lint --verbose
 ```

6. 创建一个项目来测试
 
 *  新建一个xcode项目
 *  在项目文件夹新建一个Podfile文件,内容如下
 
       ```
       platform :ios, '9.0'
       target 'SampleProject' do
         pod 'Test',:path => '../Test/Test.podspec.json'
       end
       ```
 *  运行 `pod install` 命令
 *  打开xcode项目，可以看到我们的静态库和头文件已经被引入到项目内
 
    {% asset_img "1.png" "xcode 项目" %}
  

7. 上传到cocoapods，这里要注意cocoapods现在需要先注册才能使用

 ```
 pod trunk register 你的邮箱 '用户名' --description='简单描述'
 ```

8. 完成邮箱验证后既可以把自己的代码库push到trunk上

 ```
 pod trunk push Test.podsspec.json
 ``` 
