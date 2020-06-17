# flutter app开发流程
## 环境安装
具体依照中文官网  [安装和环境配置](https://flutter.cn/docs/get-started/install) 


## 开发
```bash
flutter packages get # 安装依赖
Flutter run # 启动 要先启动模拟器 或者链接usb
```




## 打包
### Android

```bash
flutter build apk

```

### 参考  Flutter打包

* [Flutter打包](https://www.jianshu.com/p/888ac3b7df01)


### IOS

Flutter：实现iOS无证书打包ipa

```bash
flutter build iOS —release
```

参考

	* [Flutter：实现iOS无证书打包ipa - 云+社区 - 腾讯云](https://cloud.tencent.com/developer/article/1394918)


## 开发中遇到的问题
* 1. 内网环境下运行命令需要挂代理
`bash export proxy=http://x.x.x.x:8080/ export https_proxy=http://x.x.x.x:8080/`

* 2.  打包或者下载依赖包时，需要替换依赖包中的gradle 配置
	* 首先是项目本身 `android\build.gradle` 中替换 `maven` 源
    
```
 maven { url ‘https://maven.aliyun.com/repository/google’ }
        maven { url ‘https://maven.aliyun.com/repository/jcenter’ }
        maven { url ‘http://maven.aliyun.com/nexus/content/groups/public’}
        maven { url “https://storage.flutter-io.cn/download.flutter.io” }

        // google()
        // jcenter()
```


	* 第三方依赖包时有可能也需要换源， 以  `shared_preferences` 为例 我的flutter 安装目录为 `C:\flutter`. 那么的安装路径为 `C:\flutter\.pub-cache\hosted\pub.flutter-io.cn\shared_preferences-0.5.7+3\android\build.gradle` 同样替换

```
        maven { url ‘https://maven.aliyun.com/repository/google’ }
        maven { url ‘https://maven.aliyun.com/repository/jcenter’ }
        maven { url ‘http://maven.aliyun.com/nexus/content/groups/public’}
        maven { url “https://storage.flutter-io.cn/download.flutter.io” }

        // google()
        // jcenter()

```


# 参考资料
* [api文档 - 英文](https://api.flutter-io.cn/)
* [图标](https://fluttericon.com/)
* [dart 语言概览](https://dart.cn/guides/language/language-tour)