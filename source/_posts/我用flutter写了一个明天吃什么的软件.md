---
title: 我用flutter写了一个明天吃什么的软件
date: 2021-11-22
tags:
- flutter

author: YYY
---


# 我用flutter写了一个明天吃什么的软件


<img src="./assets/效果.jpeg" width = "300"  alt="图片名称" align=center />

## **准备工作**
- 开发工具 vscode
- 调试工具 一部Android手机 一条数据线

- 安装Dart、Flutter扩展

![](./assets/tools.jpg)

- 创建项目

1. Cmd+Shift+P
2. Flutter: New Project
3. Application

![](./assets/create.jpg)

- 运行

运行 -> 启动调试

至此第一个hello world运行在了手机上

## 开发必备知识

**状态管理**

- Widget 管理自己的状态

``` js
class TapboxA extends StatefulWidget {
  TapboxA({Key? key}) : super(key: key);

  @override
  _TapboxAState createState() => _TapboxAState();
}

class _TapboxAState extends State<TapboxA> {
  bool _active = false;

  void _handleTap() { // 点击该盒子时更新_active，并调用setState()更新UI
    setState(() {
      _active = !_active;
    });
  }

  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: _handleTap,
      child: Container(
        child: Center(
          child: Text(
            _active ? 'Active' : 'Inactive',
            style: TextStyle(fontSize: 32.0, color: Colors.white),
          ),
        ),
        width: 200.0,
        height: 200.0,
        decoration: BoxDecoration(
          color: _active ? Colors.lightGreen[700] : Colors.grey[600],
        ),
      ),
    );
  }
}
```

> StatefulWidget类必须有一个对应的State类

- Widget 管理子 Widget 状态

``` js
// ParentWidget 为 TapboxB 管理状态.

//------------------------ ParentWidget --------------------------------

class ParentWidget extends StatefulWidget {
  @override
  _ParentWidgetState createState() => _ParentWidgetState();
}

class _ParentWidgetState extends State<ParentWidget> {
  bool _active = false;

  void _handleTapboxChanged(bool newValue) {
    setState(() {
      _active = newValue;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Container(
      child: TapboxB(
        active: _active,
        onChanged: _handleTapboxChanged,
      ),
    );
  }
}

//------------------------- TapboxB ----------------------------------

class TapboxB extends StatelessWidget {
  TapboxB({Key? key, this.active: false, required this.onChanged})
      : super(key: key);

  final bool active;
  final ValueChanged<bool> onChanged;

  void _handleTap() {
    onChanged(!active);
  }

  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: _handleTap,
      child: Container(
        child: Center(
          child: Text(
            active ? 'Active' : 'Inactive',
            style: TextStyle(fontSize: 32.0, color: Colors.white),
          ),
        ),
        width: 200.0,
        height: 200.0,
        decoration: BoxDecoration(
          color: active ? Colors.lightGreen[700] : Colors.grey[600],
        ),
      ),
    );
  }
}
```


**路由管理**

首先需要一个路由表

``` js
import 'package:flutter/material.dart';
import 'package:lunch_tomorrow/pages/index/index.dart';
import 'package:lunch_tomorrow/pages/collect/collect.dart';
import 'package:lunch_tomorrow/pages/choose/choose.dart';
import 'package:lunch_tomorrow/pages/about/about.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: '首页',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      routes: {
        '/': (context) => ListPage(),
        'collect': (context) => CollectPage(),
        'choose': (context) => ChoosePage(),
        'about': (context) => AboutPage(),
      },
    );
  }
}

```

路由跳转

``` js
Navigator.of(context).pushNamed('choose');
```

> 了解了状态管理和路由管理，接下来就可以正式开发了


**布局文件**

``` js
@override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        //导航栏
        title: Text(_title),
        actions: <Widget>[
          //导航栏右侧菜单
          IconButton(icon: Icon(Icons.share), onPressed: () {}),
        ],
      ),
      body: _buildWidgetBody(),
      // drawer: MyDrawer(), //抽屉
      bottomNavigationBar: BottomAppBar(
        color: Colors.white,
        shape: CircularNotchedRectangle(), // 底部导航栏打一个圆形的洞
        child: Row(
          children: [
            IconButton(icon: Icon(Icons.home), onPressed: _onHome),
            SizedBox(), //中间位置空出
            IconButton(icon: Icon(Icons.person), onPressed: _onSelf),
          ],
          mainAxisAlignment: MainAxisAlignment.spaceAround, //均分底部导航栏横向空间
        ),
      ),
      floatingActionButtonLocation: FloatingActionButtonLocation.centerDocked,
      floatingActionButton: FloatingActionButton(
          //悬浮按钮
          child: Icon(Icons.add),
          onPressed: _onAdd),
    );
  }
```

> 值得注意的是 mainAxisAlignment 控制了底部tab的切角

**依赖文件**

在pubspec.yaml 文件中，维护了app需要的依赖 以及静态资源的引用

**网络请求**

``` js
import 'package:dio/dio.dart';
import 'package:lunch_tomorrow/api/http_config.dart';

class HttpRequest {
  static final BaseOptions options = BaseOptions(
      baseUrl: HTTPConfig.baseURL, connectTimeout: HTTPConfig.timeout);
  static final Dio dio = Dio(options);

  static Future<T> request<T>(String url,
      {String method = 'get',
      Map<String, dynamic>? params,
      Interceptor? inter}) async {
    // 1.请求的单独配置
    final options = Options(method: method);

    // 2.添加第一个拦截器
    dio.interceptors.add(InterceptorsWrapper(onRequest: (options, handler) {
      // Do something before request is sent
      print(
          "\n================================= 请求数据 =================================");
      print("method = ${options.method.toString()}");
      print("url = ${options.uri.toString()}");
      print("headers = ${options.headers}");
      print("params = ${options.queryParameters}");
      return handler.next(options); //continue
      // If you want to resolve the request with some custom data，
      // you can resolve a `Response` object eg: `handler.resolve(response)`.
      // If you want to reject the request with a error message,
      // you can reject a `DioError` object eg: `handler.reject(dioError)`
    }, onResponse: (response, handler) {
      // Do something with response data
      print(
          "\n================================= 响应数据开始 =================================");
      print("code = ${response.statusCode}");
      print("data = ${response.data}");
      print(
          "================================= 响应数据结束 =================================\n");
      return handler.next(response); // continue
      // If you want to reject the request with a error message,
      // you can reject a `DioError` object eg: `handler.reject(dioError)`
    }, onError: (DioError e, handler) {
      // Do something with response error
      print(
          "\n=================================错误响应数据 =================================");
      print("type = ${e.type}");
      print("message = ${e.message}");
      print("stackTrace = ${e.stackTrace}");
      print("\n");
      return handler.next(e); //continue
      // If you want to resolve the request with some custom data，
      // you can resolve a `Response` object eg: `handler.resolve(response)`.
    }));

    // 3.发送网络请求
    try {
      Response response =
          await dio.request<T>(url, queryParameters: params, options: options);
      return response.data;
    } on DioError catch (e) {
      return Future.error(e);
    }
  }
}

```

> 后端接口是我用koa写的，数据库用的mongodb 感兴趣的同学请移步 [server](https://github.com/JustReactY/myservers)


**github**

[项目地址](https://github.com/JustReactY/lunch_tomorrow)