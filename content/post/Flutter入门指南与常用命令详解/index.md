---
title: Flutter入门指南与常用命令详解
description: 本文介绍Flutter入门指南与常用命令。
date: 2025-04-25T18:34:25+08:00
draft: false
categories:
- 编程
tags:
- Flutter
---

# Flutter 入门指南与常用命令详解

## 前言

本文介绍三个Flutter入门项目的基本操作及Flutter常用命令，帮助开发者快速上手Flutter开发。

- **创建项目命令**：`flutter create project_name`
- **运行项目命令**：`flutter run`（先要执行`cd project_name`切换到项目目录）
- **热重启命令**：`r` 或 `R`
- **终止项目命令**：`q`

---

## 一、Flutter 官方资源

- 官方网站：[https://flutter.dev/](https://flutter.dev/)
- 中文官网：[https://flutter.cn/](https://flutter.cn/)
- 官方博客：[https://flutter.cn/posts](https://flutter.cn/posts)
- Flutter CLI 文档：[https://flutter.cn/docs/reference/flutter-cli](https://flutter.cn/docs/reference/flutter-cli)

**官方与中文站点的对应规则**：

| 官方网址                                                     | 中文网址                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [https://flutter.dev/xxx](https://flutter.dev/xxx)           | [https://flutter.cn/xxx](https://flutter.cn/xxx)             |
| [https://docs.flutter.dev/xxx](https://docs.flutter.dev/xxx) | [https://flutter.cn/docs/xxx](https://flutter.cn/docs/xxx)   |
| [https://storage.googleapis.com/xxx](https://storage.googleapis.com/xxx) | [https://storage.flutter-io.cn/xxx](https://storage.flutter-io.cn/xxx) |

---

## 二、Flutter 常用 CLI 命令详解

Flutter 命令支持多种格式（参数位置、简写、单杠/双杠），并具备智能提示常见错误命令。

### 1. 基础命令

| 命令                 | 说明                                                     |
| -------------------- | -------------------------------------------------------- |
| `flutter -h`         | 显示帮助信息，支持多种帮助格式如 `-h xx` 或 `help xx`    |
| `flutter doctor`     | 环境检测，只需解决项目相关问题，不必全部处理             |
| `flutter --version`  | 查看 Flutter SDK、Framework、Engine 和 Tools 版本信息    |
| `where flutter dart` | 查看 Flutter 和 Dart 安装路径，确保 flutter\bin 在最前面 |

### 2. 频道管理

| 命令                        | 说明                                                         |
| --------------------------- | ------------------------------------------------------------ |
| `flutter channel`           | 查看所有发行渠道（分支）及当前使用渠道                       |
| `flutter channel <channel>` | 切换至指定渠道，如 master、dev、beta、stable，推荐使用 stable |

### 3. 配置与更新

| 命令              | 说明                                         |
| ----------------- | -------------------------------------------- |
| `flutter config`  | 基本配置，如平台启用、SDK/IDE 路径、默认设备 |
| `flutter upgrade` | 更新 Flutter SDK 和依赖包                    |

### 4. 依赖管理

| 命令                    | 说明                                                       |
| ----------------------- | ---------------------------------------------------------- |
| `flutter pub <command>` | 依赖包管理，包括 upgrade, add, remove, get, deps, outdated |

### 5. 项目与构建

| 命令                                               | 说明                                                         |
| -------------------------------------------------- | ------------------------------------------------------------ |
| `flutter devices`                                  | 列出可用设备（Windows、Chrome、Android 设备等）              |
| `flutter clean`                                    | 清理构建缓存，删除 build/.dart_tool，释放空间约至 5M         |
| `flutter create .`                                 | 创建项目，或修复现有项目。支持 app/module/package/plugin 类型 |
| `flutter create --platforms=windows,macos,linux .` | 给现有项目添加桌面平台支持                                   |
| `flutter build <type>`                             | 构建指定类型产物（aar、apk、appbundle、bundle、web、windows） |
| `flutter run -d <device> --release/debug`          | 构建并启动应用，指定设备及构建类型                           |
| `flutter attach`                                   | 连接到已运行的 Flutter 应用，实现调试（热重载、热启动等功能） |

> **注意**：Web 应用启动完成后，需要启动 Web 服务器（如 `npm serve`）才能访问。

---

## 三、构建 Android 包相关命令详解

### 1. 构建 AAR 包（Android Archive）

命令示例：

```bash
flutter build aar --no-debug --no-profile --target-platform=android-arm64 --build-number=2.0
flutter build aar -h    # 显示详细帮助
flutter build aar -v    # 显示详细日志
```

- 默认入口为 `lib/main.dart`，暂不可配置。
- 主要参数说明：
  - `--[no-]debug`、`--[no-]release`、`--[no-]profile`：构建不同版本（调试、发布、性能分析）。
  - `--build-number`：内部版本号。
  - `--target-platform`：支持平台，包括 `android-arm`（armeabi-v7a）、`android-arm64`（arm64-v8a）、`android-x64`（x86-64），目前不支持 `android-x86`。
  - `--output-dir`：生成AAR存放路径，默认为当前目录的 `android/build`。
  - `--flavor`：支持 Android 的 `product flavors`。
  - 其他参数包括代码混淆、分割调试信息等，详见官方文档。

详见：[StackOverflow - How to set Flutter build android aar version](https://stackoverflow.com/questions/60597182/how-to-set-flutter-build-android-aar-version)

---

### 2. 构建 APK 包

命令示例：

```bash
flutter build apk --release --target-platform=android-arm64 --build-number=88 --build-name=8.8
flutter build apk -h  # 帮助
flutter build apk -v  # 详细日志
```

- 仅在 application 项目中支持 `--build-number`, `--build-name` 参数，module项目中无效。
- 参数与 `flutter build aar` 大致相同，新增：
  - `--target=<path>`：入口文件，默认为 `lib/main.dart`。
  - `--[no-]analyze-size`：生成构建文件大小分析。
  - `--code-size-directory`：分析文件存放目录，默认 `build/`。
  - `--[no-]multidex`：是否启用 MultiDex，默认启用。
  - `--ignore-deprecation`：忽略弃用 API 警告。
  - `--split-per-abi`：分 ABI 构建多个独立 APK。

---

## 四、Flutter 调试命令 —— flutter attach

`flutter attach` 命令用于连接一个已运行的 Flutter 应用，实现调试功能，命令行内可以执行热重载、热重启等操作，与 IDE 中的快捷命令功能相同。

### 常用交互命令：

| 快捷键 | 功能说明                                 |
| ------ | ---------------------------------------- |
| `r`    | 热重载（Hot reload）                     |
| `R`    | 热重启（Hot restart）                    |
| `d`    | 取消连接（Detach），断开调试但应用仍运行 |
| `q`    | 终止程序（Quit）                         |
| `c`    | 清屏                                     |
| `s`    | 截图，保存为 `flutter.png`               |
| `v`    | 打开 Flutter DevTools                    |
| `h`    | 显示所有可用命令列表                     |
| `w`    | 打印 Widget 层级结构                     |
| `t`    | 打印渲染树（Rendering tree）             |
| `L`    | 打印布局树（Layer tree）                 |
| `o`    | 切换模拟操作系统                         |
| `b`    | 切换亮度模式（浅色/暗色）                |

> 工具会搜索正在运行的 Flutter app 或 module，如果未检测到，会等待应用启动后自动连接。

---

## 五、三个入门项目详解与代码注释

下面对三个基础Flutter项目的完整代码提供详细注释，帮助理解Flutter框架的核心概念、Dart语法和常见用法。

---

### 1. Hello Flutter

**目标**：学习Flutter项目的基本结构与Dart基础语法。

```dart
import 'package:flutter/material.dart';  // 引入Flutter的Material组件库

// 应用入口函数，开始执行Flutter App
void main() {
  runApp(const MyApp());  // 运行MyApp组件，const表示不可变
}

// 自定义无状态组件MyApp，继承StatelessWidget
class MyApp extends StatelessWidget {
  const MyApp({super.key});  // 构造函数，传入key用于组件唯一标识

  @override
  Widget build(BuildContext context) {
    // 返回一个Material风格的应用，包含主题和首页结构
    return MaterialApp(
      title: '皇极经世',  // 应用标题
      theme: ThemeData(
        primarySwatch: Colors.yellow,  // 主题主色调为黄色
      ),
      home: Scaffold(  // 脚手架，搭建页面基本结构
        appBar: AppBar(  // 应用栏
          title: Text('皇极经世'),  // 标题文字
        ),
        body: Center(  // 主体内容居中显示
          child: Text('算命吗？', 
            style: Theme.of(context).textTheme.headlineMedium,  // 使用主题中的中大型标题样式
          ),
        ),
      ),
    );
  }
}
```

---

### 2. 点击计数器

**目标**：理解Stateless与Stateful组件的区别，学习事件处理和状态更新。

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Counter',  // 应用名
      theme: ThemeData(
        primarySwatch: Colors.green,  // 主题色为绿色
      ),
      home: HomePage(title: 'TianHanCounter'),  // 主页，传入标题参数
    );
  }
}

// 有状态组件HomePage，支持状态管理
class HomePage extends StatefulWidget {
  const HomePage({super.key, required this.title});
  final String title;  // 组件接收标题字符串

  @override
  State<StatefulWidget> createState() {
    return _HomePageState();  // 创建状态对象
  }
}

// HomePage的状态实现类
class _HomePageState extends State<HomePage> {
  int _counter = 0;  // 计数器初始值为0

  // 增加计数器值
  void _incrementCounter(int n) {
    setState(() {  // 调用setState触发UI刷新
      _counter += n;
    });
  }

  // 递减计数器
  void _decrementCounter() {
    setState(() {
      _counter--;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title),  // 使用widget中的标题
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,  // 垂直居中排列
          children: <Widget>[
            Text('You have pushed the button this many times:'),
            Text('$_counter', 
              style: Theme.of(context).textTheme.headlineMedium,  // 大号字体显示计数
            ),
          ],
        ),
      ),
      floatingActionButton: Row(
        mainAxisAlignment: MainAxisAlignment.end,  // 按钮右对齐
        children: <Widget>[
          FloatingActionButton(
            onPressed: _decrementCounter,  // 点击时递减计数
            tooltip: 'Decrement',  // 悬浮提示
            child: Icon(Icons.remove),  // 减号图标
          ),
          SizedBox(width: 10),  // 按钮间距
          FloatingActionButton(
            onPressed: () => _incrementCounter(3),  // 点击增加3
            tooltip: 'Increment',
            child: Icon(Icons.add),  // 加号图标
          )
        ],
      ),
    );
  }
}
```

---

### 3. 心爱配对 (Love Match)

**目标**：学习`ListView`展示列表、页面导航`Navigator`与较复杂的状态管理。

> 依赖包安装：需要安装`english_words`，可以通过修改`pubspec.yaml`文件或在命令行使用`flutter pub add english_words`安装。

```dart
import 'package:flutter/material.dart';
import 'package:english_words/english_words.dart';  // 引入英文单词对生成库

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: '爱心配对',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(
          seedColor: Colors.deepPurple,  // 主题种子颜色深紫色
        ).copyWith(surfaceTint: Colors.transparent),  // 取消表面颜色阴影
        appBarTheme: AppBarTheme(
          backgroundColor: Colors.green,  // AppBar背景绿色
          elevation: 4.0,  // 阴影高度
          shadowColor: Theme.of(context).colorScheme.shadow,  // 阴影颜色
        ),
      ),
      home: HomePage(),  // 主页组件
    );
  }
}

class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<StatefulWidget> createState() {
    return _HomePageState();
  }
}

class _HomePageState extends State<HomePage> {
  final _words = <WordPair>[];      // 存储生成的单词对列表
  final _favorites = <WordPair>[];  // 存储用户收藏的单词对

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('爱心配对'),
        actions: [
          // 右上角收藏按钮，点击进入收藏页面
          IconButton(icon: Icon(Icons.list), onPressed: _goToFavorites),
        ],
      ),
      body: Center(child: _buildList()),  // 主体为单词列表
    );
  }

  // 跳转到收藏页面，展示已收藏的单词对列表
  void _goToFavorites() {
    Navigator.of(context).push(
      MaterialPageRoute<void>(
        builder: (BuildContext context) {
          // 构造收藏单词对的列表项
          final Iterable<ListTile> tiles = _favorites.map((WordPair pair) {
            return ListTile(
              title: Text(pair.asPascalCase, style: TextStyle(fontSize: 18.0)),
            );
          });
          return Scaffold(
            appBar: AppBar(title: Text('收藏')),
            body: ListView(children: tiles.toList()),  // 使用ListView展示收藏列表
          );
        },
      ),
    );
  }

  // 构建主列表，动态生成单词对列表项
  Widget _buildList() {
    // 每次调用添加10个单词对
    _words.addAll(generateWordPairs().take(10));
    return ListView.builder(
      itemBuilder: (context, item) {
        if (item.isOdd) return Divider();  // 奇数行插入分割线
        int index = item ~/ 2;  // 计算实际单词对索引
        if (index >= _words.length) {
          // 超出列表长度时自动加载更多
          _words.addAll(generateWordPairs().take(10));
        }
        return _buildRow(index);  // 构建单个列表项
      },
    );
  }

  // 单个列表项构建，包含单词对文字和收藏图标
  Widget _buildRow(int index) {
    return ListTile(
      title: Text(_words[index].asPascalCase),  // Pascal命名格式展示单词对
      trailing: _changeIcon(index),              // 右侧收藏图标
      onTap: () => {_changeFavorites(index)},   // 点击切换收藏状态
    );
  }

  // 根据是否收藏显示不同图标
  Widget _changeIcon(int index) {
    if (_favorites.contains(_words[index])) {
      return Icon(Icons.favorite, color: Colors.red);  // 已收藏红色爱心
    } else {
      return Icon(Icons.favorite_border);  // 未收藏空心爱心
    }
  }

  // 切换收藏状态
  void _changeFavorites(int index) {
    setState(() {
      if (_favorites.contains(_words[index])) {
        _favorites.remove(_words[index]);  // 取消收藏
      } else {
        _favorites.add(_words[index]);     // 添加收藏
      }
    });
  }
}
```

