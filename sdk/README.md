### 引入SDK

使用git或者引入源码的方式

```dart
fair_pushy:
   git:
   	url: https://github.com/wuba/FairPushy.git
	path: FairPushy/sdk
```



#### 初始化SDK

main()函数中调用初始化方法

```dart
FairPushy.init(
      appID: '1001',
      updateUrl: "https://fangfe.58.com/fairapp/module_patch_bundle",
      debug: true);
```

> appid：web可视化平台中的项目id
>
> updateUrl：接入方获取补丁config文件的服务器地址
>
> debug：运行环境



#### 热更新api

1. ##### 单module更新工程

   进入flutter工程就需要下载热更新文件，两种方式：

   a. 直接调用updateBundle接口：

   ```dart
   FairPushy.updateBundle(bundleid: "6005")
   ```

   > bundleid可视化平台中模块对应的补丁唯一标识

   

   b. 使用loading中间件，用FairPushyWidget替代根widget

   ```dart
   MaterialApp(
       title: 'Flutter Demo',
       theme: ThemeData(
         primarySwatch: Colors.blue,
       ),
       home: FairPushyWidget(
         bundleid: '6005',
         targetWidgetBuilder: (context) => HomePage(),
   		));
   ```

   

2. ##### 多module工程

   分为进模块独立更新和进app全部更新的方式。

   a. 进模块独立更新：

   如果只使用更新API，业务方自己实现目标页loading过程，且需要目标页调用updateBundle：

   ```dart
   FairPushy.updateBundle(bundleid: "6005")
   ```

   

   调用了loading中间件，需要在module间跳转的时候，可以先跳SDK中提供的中间件，中间件中实现了补丁的更新逻辑，走完更新补丁逻辑，会把目标页替换成传入的targetWidget。

   ```dart
   Navigator.push(context, MaterialPageRoute(builder: (context) {
         return FairPushyWidget(
             bundleid: BundleConst.car,
           // targetPageName: "car_cate"
             targetWidgetBuilder: (context) => CarCatePage());
       }));
   ```

   > targetPageName：module间跳转时的目标界面pagename，传入pagename需要在FairPushy中进行界面的注册，详见example示例工程
   >
   > targetWidgetBuilder：module间跳转时的目标界面widget

   ​	

   b. 进app全部更新

   如果接入方在多module的工程结构下，想进app就下载所有的补丁文件，可以调用getConfigs接口获取appid对应项目下的所有补丁config信息，然后调用downloadConfig方法进行下载。

   ```dart
   FairPushy.getConfigs("https://fangfe.58.com/fairapp/module_patch_app").then((value) {
     if (null != value && value.isNotEmpty) {
       for (var i = 0; i < value.length; i++) {
         FairPushy.downloadConfig(value[i]);
       }
     }
   });
   ```

   > url：业务方获取app所有资源文件的服务器地址
   
 #### Fair调用
   调用FairPushy.getFilePath获取热更新文件
   ```dart
   FairWidget(
    name: 'carcate',
    path: FairPushy.getFilePath(bundleId: '6005', filename: 'car_cate'));
   ```
   > filename：widget对应的文件名