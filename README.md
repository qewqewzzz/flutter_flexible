# flexible脚手架介绍
基础环境版本

• Flutter version 1.12.13+hotfix.9

• Dart version 2.7.2

<br>
内置集成功能：

1、状态管理：集成Provider在Flutter项目中，任何页面声明好store，注入providers_config.dart文件内即可使用。

2、页面组件支持别名路由跳转传参（参数接收更短更便捷），无需任何插件支持！简单易用，无学习成本。
```
Navigator.pushNamed(
  context,
  '/testDemo',
  arguments: {'data': 'hello world'},
);

// 子组件使用及接收
class testDemo extends StatefulWidget {
  testDemo({Key key, this.params}) : super(key: key);
  final params;

  @override
  _testDemoState createState() => _testDemoState();
}
class _testDemoState extends State<testDemo>{
  @override
  void initState() {
    super.initState();
    print(widget.params); // 别名参数接收
  }
}
```

3、页面路由跳转容错处理，未声明路由跳转错误，指定跳转到错误页面。能让你第一时间发现低级错误bug，友好提示页面清晰明了。

4、内置全局主题一键换色，只需要配置你的主题颜色，调用方法即可。

5、内置全局浮动调试组件，让你在真机上也能便利的获取错误捕获，在我的页面》右下按钮 查看效果。


## 文件夹结构

这是项目中一直会使用的结构
    lib/
    |- constants/ # 常量文件夹
    |- config/ # 配置参数
    |- components/ # 共用widget组件封装
    |- provider/ # 全局状态
    |- pages/ # 页面ui层，每个独立完整的页面
    |- utils/ # 公共方法抽离
      |- dio/ # dio底层请求封装safeRequest
    |- service/ # 请求接口抽离层
    |- routes/
      |- routesInit.dart # 定义路由页面
    |- main.dart # 入口文件

<br/><br/>

# 快速上手

## 启动项目
下载此仓库文件后，进入项目目录文件夹

初始化安装依赖包以及启用APP（记的开启你的模拟器）
输入以下命令：
```
npm run initApp // 方式一

// 方式二：手动输入flutter命令
flutter pub get
flutter run
```
<br/>

----------

## 打包项目方式
你可以使用flutter原生命令，或是使用脚手架内置好的直接使用。

同时打包android和ios二个APP文件，输入以下命令：

```
npm run build
```

<br/>
单独打包某一个平台的文件命令如下：

```
npm run build:apk // 打包安卓的APK文件

npm run build:ios // 打包IOS的文件

npm run build:web // 打包web的文件
```

<br>

# 命令行参数说明

|       命令        |                         说明                         |
| :---------------: | :--------------------------------------------------: |
|     npm start     |       启动APP项目，请提前开好模拟器或连接真机        |
|   npm run build   |       打包项目生成APP，会打包安卓和IOS二个版本       |
| npm run build:apk |                打包生成安卓的APP文件                 |
| npm run build:ios |                 打包生成IOS的APP文件                 |
| npm run build:web |               打包生成纯前端web的文件                |
|   npm run upsdk   | 更新sdk版本，全局的flutter和dart版本将更新为最新版本 |
|  npm run appkey   |             验证打包后的安卓apk签名信息              |
<br>

# 功能介绍

## 获取全局context

全局Key和全局context都注入存放在IOC容器当中，而IOC容器实现是使用了get_it实现。

使用方式引入ioc/locator.dart 容器实例文件，直接使用你之前已经注入的类方法。

PS：你可以把一些全局的类都可以注入到IOC容器中使用，从而实现页面更加简洁，不需要在某个组件或页面中导入更多的import

```dart
import 'package:flexible/ioc/locator.dart' show locator, CommonService; // 引入容器实例
CommonService _commonIoc = locator.get<CommonService>(); // 获取指定IOC容器的方法
_commonIoc.getGlobalContext; // 全局context对象
```

## dio请求底层封装使用
已经抽离请求组件dio，可直接使用

```
import 'package:flexible/utils/dio/safeRequest.dart';
// get请求使用方法，同dio组件request方法
getHomeData() async {
  Map resData = await safeRequest(
    'http://url',
    queryParameters: {'version': version},
    options: Options(method: 'GET'),
  );
}
// post请求
getHomeData() async {
  Map resData = await safeRequest(
    'http://url',
    data: {'version': version}, // 传递参数
    options: Options(method: 'POST'),
  );
}
```


## 更新APP版本组件

1、添加安卓的存储权限申请标签(默认已添加, 可跳过此步)，如有删除安卓目录生成过的，请自行添加一下。

安卓权限配置文件 android\app\src\main\AndroidManifest.xml

```
<manifest xmlns:android="http://schemas.android.com/apk/res/android" package="com.example.flutter_flexible">
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
    <application>...其它配置忽略</application>
</manifest>
```

2、在lib\components\UpdateAppVersion\getNewAppVer.dart文件中，getNewAppVer方法直接运行更新APP版本，但有少部份需要自己实现，已标注TODO位置，指定APP下载地址和获取新版本的接口替换。
```dart
// TODO:替换成自己的获取新版本APP的接口
Map resData = await getNewVersion();
// 模拟参数结构如下  {"code":"0","message":"success","data":{"version":"1.1.0","info":["修复bug提升性能","增加彩蛋有趣的功能页面","测试功能"]}}

UpdateAppVersion(
    // TODO: 传入新版本APP相关参数、版本号、更新内容、下载地址等
    version: resData['version'] ?? '', // 版本号
    info: (resData['info'] as List).cast<String>() ?? [], // 更新内容介绍
    // ios是苹果应用商店地址
    iosUrl: 'itms-apps://itunes.apple.com/cn/app/id414478124?mt=8',
    // 安卓APK下载地址
    androidUrl:
        'https://b6.market.xiaomi.com/download/AppStore/08fee50a2945783f419a5945f8e89707f2640c6b0/com.ss.android.ugc.aweme.apk',
  ),
)
```

3、在指定页面运行 检查APP版本函数，默认在lib\pages\HomeBarTabs\HomeBarTabs.dart中，运行检查更新APP函数，你可以指定其它位置运行检查新版本。

```dart
import 'package:flexible/components/UpdateAppVersion/UpdateAppVersion.dart' show getNewAppVer;

getNewAppVer(); // 执行更新检查
```

### 全局主题更换

把你的主题配置参数文件放入lib\config\themes文件夹中，然后part到index_theme.dart文件中统一管理，另外还有灰度模式。

案例内容如下：

```dart
import 'package:flutter/material.dart';
// 以下你配置的全局主题颜色参数
part 'themeBlueGrey.dart';
part 'themeLightBlue.dart';
part 'themePink.dart';

```

主题配色具体可以参考是关配色文件 themeBlueGrey.dart等。

在需要替换主题的页面中调用如下：

```dart
import 'package:flexible/config/themes/index_theme.dart' show themeBlueGrey; // 主题文件
import 'package:flexible/provider/themeStore.p.dart'; // 全局主题状态管理
ThemeStore _theme = Provider.of<ThemeStore>(context);
_theme.setTheme(themeBlueGrey); // 替换主题，注入主题配置即可
```

#### 灰度模式

首页灰度模式不需要单独配置主题文件，使用方式如下：

```dart
import 'package:flexible/pages/HomeBarTabs/provider/homeBarTabsStore.p.dart';
HomeBarTabsStore homeBarStore = Provider.of<HomeBarTabsStore>(context);
homeBarStore.setGrayTheme(true); // 设置灰度模式
```

