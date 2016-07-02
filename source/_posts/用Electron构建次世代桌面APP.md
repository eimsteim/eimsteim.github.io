title: 用Electron构建次世代桌面APP
date: 2016-02-01 21:31:58
tags: [Node.js,Electron]
---
** 什么是Electron？**

Electron是基于Node.js的桌面应用开发框架。他的出现是为了解决传统桌面应用程序开发中，一直存在着的老大难问题：跨平台一致性。

Electron的工作原理是模拟一个简化版的Chrome内核浏览器，通过编写HTML和CSS程序来渲染界面，而在底层调用Node.js来和操作系统交互，从而另辟蹊径地解决了跨平台一致性的问题。
<!--more-->
***
** 怎样学习Electron？**

因为Eelectron是基于Node.js的，所以掌握一定的JavaScript语法是必须的。

另外，Electron的文档是非常丰富的，重要的资料网站包括：

* [Electron官方](http://electron.atom.io/)
* [GitHub主页](https://github.com/atom/electron)
* [GitHub中文文档](https://github.com/atom/electron/tree/master/docs-translations/zh-CN)
***
** 安装Electron **

在安装Electron之前，你可能需要先安装`Node.js`环境和`npm`，如果不知道怎样安装，请先耐心地学习[这里](https://nodejs.org/en/)。

通过npm来安装Electron应该是世界上最快的方式了：

```python
# Install the `electron` command globally in your $PATH
npm install electron-prebuilt -g
```
你也可以选择不安装在global环境下：
```python
# Install as a development dependency
npm install electron-prebuilt --save-dev
```
这里需要注意的是，有些网络环境下在安装时可能出现超时，如果发生这种情况，你可以尝试使用[淘宝NPM镜像](http://npm.taobao.org/)。
***
> *** 说明： ***
> 本文并不会完全按照官方文档的顺序来讲解，而是提及一些重要的知识点，并通过一个Demo来讲解如何实践开发，以及我在这个开发这个Demo的中遇到过的问题。
> ***
> *** 重要概念：***
>
> 1. Electron的界面渲染分为两个部分：主进程（Main Process）和渲染进程（Render Process），主进程是指应用程序的系统原生UI部分.主进程包括浏览器窗体（BrowserWindow）和菜单（Menu），以及与操作系统调用相关的操作（例如：文件IO）；渲染进程则是在窗体中通过HTML和CSS进行渲染呈现的部分。
>
> 2. 如果需要在渲染进程中调用主进程的相关功能，必须通过remote的方式调用，具体做法后面会讲到，需要谨记于心。

***
** 开始编码！**

一般来说，一个最简单的Electron应用程序的文档结构如下：
```
your-app/
├── package.json
├── main.js
└── index.html
```
其中package.json文件是用来定义应用程序的基本信息，包括名称、版本号、程序入口和依赖。一个最简单的package.json可能形如：
```json
{
  "name"    : "your-app",
  "version" : "0.1.0",
  "main"    : "main.js"
}
```
其中name是你的应用程序名称，version为当前版本号，而main则标识了程序入口。而实际上这个文件就是Node.js的打包脚本，类似Maven的pom.xml。如果希望了解更全面的配置，可以参考[这里](https://docs.npmjs.com/files/package.json)。

** 注意：** 如果不设置main参数，程序会默认尝试从根目录加载 index.js 脚本。

下面是main.js的基本实例，它将负责创建浏览器窗体，并负责处理系统事件：
```javascript
'use strict';

const electron = require('electron');
const app = electron.app;  // Module to control application life.
const BrowserWindow = electron.BrowserWindow;  // Module to create native browser window.

// Keep a global reference of the window object, if you don't, the window will
// be closed automatically when the JavaScript object is garbage collected.
var mainWindow = null;

// Quit when all windows are closed.
app.on('window-all-closed', function() {
  // On OS X it is common for applications and their menu bar
  // to stay active until the user quits explicitly with Cmd + Q
  if (process.platform != 'darwin') {
    app.quit();
  }
});

// This method will be called when Electron has finished
// initialization and is ready to create browser windows.
app.on('ready', function() {
  // Create the browser window.
  mainWindow = new BrowserWindow({width: 800, height: 600});

  // and load the index.html of the app.
  mainWindow.loadURL('file://' + __dirname + '/index.html');

  // Open the DevTools.
  mainWindow.webContents.openDevTools();

  // Emitted when the window is closed.
  mainWindow.on('closed', function() {
    // Dereference the window object, usually you would store windows
    // in an array if your app supports multi windows, this is the time
    // when you should delete the corresponding element.
    mainWindow = null;
  });
});
```
- 从上述代码中可以看到，引入模块的方式与Node.js基本一致，都是使用require('模块名')。
- 我曾尝试通过将new BrowserWindow()中的width和height改为100%以实现默认全尺寸的效果，但是很可惜失败了，从[官方API文档](http://electron.atom.io/docs/v0.36.5/api/browser-window/)中得知：在这里Electron并不支持非Int类型的赋值。

最后是index.html，这个文件是应用程序的主界面，也就是刚才所说的“渲染进程”的载体。从这里开始，我的代码开始变得与[atom/electron-quick-start](https://github.com/atom/electron-quick-start)大相径庭，这是因为我想要的功能更丰富，界面也更复杂。
```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8"/>
    <title>Find Word</title>

    <style type="text/css">
    body {
      padding: 5px;
    }
    </style>
    <!-- 新 Bootstrap 核心 CSS 文件 -->
    <link rel="stylesheet" href="bootstrap/css/bootstrap.min.css">
    <!-- <link href="flatui/css/vendor/bootstrap/css/bootstrap.min.css" rel="stylesheet"> -->
    <!-- <link href="flatui/css/flat-ui.css" rel="stylesheet"> -->
    <link rel="stylesheet" href="bootstrap/css/buttons.css">

    <!-- rainbow for highlight code -->
    <!-- <link href="rainbow/themes/blackboard.css" rel="stylesheet" type="text/css" media="screen">
    <script src="rainbow/js/rainbow.js"></script>
    <script src="rainbow/js/language/generic.js"></script>
    <script src="rainbow/js/language/python.js"></script> -->

    <!-- jQuery文件。务必在bootstrap.min.js 之前引入 -->
    <!-- <script src="bootstrap/js/jquery.min.js"></script> -->
    <script type="text/javascript">
        window.$ = window.jQuery = require(__dirname+'/bootstrap/js/jquery.min.js');
    </script>

    <!-- 最新的 Bootstrap 核心 JavaScript 文件 -->
    <script src="bootstrap/js/bootstrap.min.js"></script>

    <script type="text/javascript">
    function line2br(data){
      return data.split("\n").join("<br />");
    }
    function check(fileName, lookingForString){
      var data = readFile(fileName);
      var exc = new RegExp(lookingForString);
      if(exc.test(data)) {
        return true;
      } else {
        return false;
      }
    }
    function readFile(fileName){
      if(fs.existsSync(fileName)) return fs.readFileSync(fileName,"utf-8");
    }
    /**
    * 根据关键字检索文件，目前只能查找当前目录，无法查找子目录
    */
    function find(path, keyword){
      //去掉当前已经存在的样式
      $('#file_list > .list-group-item-danger').removeClass('list-group-item-danger');
      //开始遍历查找
      files.forEach(function(file){
        var full_path = path + '/' + file;
        fs.stat(full_path, function(err, stats){
          if(stats.isFile()) { //如果是文件，则读取，并筛选
            if(check(full_path, keyword)){
              //匹配到file_list中的值，在file_content中显示其内容
              $('#file_list > .list-group-item').each(function(i, n) {
                if($(n).text() === file) {
                  $(n).addClass('list-group-item-danger');
                }
              });
            }
          }
        });
      });
    }
    $(document).ready(function(){
      $('#btn_find').click(function(){
        //console.log($('#ipt_keyword').val());
        var path = $('#target_dir').text();
        var keyword = $('#ipt_keyword').val();
        if(path != undefined && path != '') {
          if(keyword != undefined && keyword != ''){
            find(path, keyword);
          } else { //如果没有填keyword，给出提示
            alert('请填写关键字');
            return;
          }
        } else { //如果没有填path，给出提示
          alert('先选择需要扫描的目录：File->Open Folder');
          return;
        }
      });
      //click to show file content
      $('#file_list').click(function(e){
        var path = $('#target_dir').text() + '/' + $(e.target).text();
        var data = fs.readFileSync(path, "utf-8");
        //var node = line2br(data);
        // var node = '<pre><code data-language="python">' + data + '</code></pre>';
        $('#file_content').html(data);
        //Rainbow._highlight($('#file_content').html());
      });
    });
    </script>

    <!-- create menu -->
    <script type="text/javascript">
    const remote = require('electron').remote;
    const dialog = remote.dialog;
    //read file
    const fs = require('fs');
    const Menu = remote.Menu;
    //const MenuItem = remote.MenuItem;
    var template = [
      {
        label: 'File',
        submenu: [
          // {
          //   label: 'Open File',
          //   accelerator: 'CmdOrCtrl+O',
          //   click: function(){
          //     //console.log(dialog.showOpenDialog({ properties: [ 'openFile', 'openDirectory', 'multiSelections' ]}));
          //     var file_path = dialog.showOpenDialog({ properties: [ 'openFile', 'multiSelections' ]});
          //     console.log(file_path);
          //     if(file_path != undefined) {
          //       $('#target_dir').val(file_path);
          //     }
          //   }
          // },
          {
            label: 'Open Folder',
            accelerator: 'Shift+CmdOrCtrl+O',
            click: function(){
              //console.log(dialog.showOpenDialog({ properties: [ 'openFile', 'openDirectory', 'multiSelections' ]}));
              var dir_path = dialog.showOpenDialog({ properties: [ 'openDirectory' ]});
              if(dir_path != undefined) {
                $('#target_dir').text(dir_path);
                //list files
                if(fs != undefined) {
                  files = fs.readdirSync(dir_path[0]);
                  $('#file_list').empty();
                  files.forEach(function(file){
                    $('#file_list').append('<a href="javascript:void(0)" class="list-group-item" style="overflow: hidden;">' + file + '</a>');
                  })
                }
              }
            }
          }
        ]
      },
      {
        label: 'Edit',
        submenu: [
          {
            label: 'Undo',
            accelerator: 'CmdOrCtrl+Z',
            role: 'undo'
          },
          {
            label: 'Redo',
            accelerator: 'Shift+CmdOrCtrl+Z',
            role: 'redo'
          },
          {
            type: 'separator'
          },
          {
            label: 'Cut',
            accelerator: 'CmdOrCtrl+X',
            role: 'cut'
          },
          {
            label: 'Copy',
            accelerator: 'CmdOrCtrl+C',
            role: 'copy'
          },
          {
            label: 'Paste',
            accelerator: 'CmdOrCtrl+V',
            role: 'paste'
          },
          {
            label: 'Select All',
            accelerator: 'CmdOrCtrl+A',
            role: 'selectall'
          },
        ]
      },
      {
        label: 'View',
        submenu: [
          {
            label: 'Reload',
            accelerator: 'CmdOrCtrl+R',
            click: function(item, focusedWindow) {
              if (focusedWindow)
                focusedWindow.reload();
            }
          },
          {
            label: 'Toggle Full Screen',
            accelerator: (function() {
              if (process.platform == 'darwin')
                return 'Ctrl+Command+F';
              else
                return 'F11';
            })(),
            click: function(item, focusedWindow) {
              if (focusedWindow)
                focusedWindow.setFullScreen(!focusedWindow.isFullScreen());
            }
          },
          {
            label: 'Toggle Developer Tools',
            accelerator: (function() {
              if (process.platform == 'darwin')
                return 'Alt+Command+I';
              else
                return 'Ctrl+Shift+I';
            })(),
            click: function(item, focusedWindow) {
              if (focusedWindow)
                focusedWindow.toggleDevTools();
            }
          },
        ]
      },
      {
        label: 'Help',
        role: 'help',
        submenu: [
          {
            label: 'Learn More',
            click: function() { require('electron').shell.openExternal('http://electron.atom.io') }
          },
        ]
      },
    ];
    if (process.platform == 'darwin') {
      var name = require('electron').app.getName();
      template.unshift({
        label: name,
        submenu: [
          {
            label: 'About ' + name,
            role: 'about'
          },
          {
            type: 'separator'
          },
          {
            label: 'Services',
            role: 'services',
            submenu: []
          },
          {
            type: 'separator'
          },
          {
            label: 'Hide ' + name,
            accelerator: 'Command+H',
            role: 'hide'
          },
          {
            label: 'Hide Others',
            accelerator: 'Command+Alt+H',
            role: 'hideothers'
          },
          {
            label: 'Show All',
            role: 'unhide'
          },
          {
            type: 'separator'
          },
          {
            label: 'Quit',
            accelerator: 'Command+Q',
            click: function() { app.quit(); }
          },
        ]
      });
      // Window menu.
      template[3].submenu.push(
        {
          type: 'separator'
        },
        {
          label: 'Bring All to Front',
          role: 'front'
        }
      );
    }
    var menu = Menu.buildFromTemplate(template);
    Menu.setApplicationMenu(menu);
    </script>
  </head>
  <body>

    <div class="panel panel-default">
      <div class="panel-heading"><strong>Choose Directory</strong></div>
      <div class="panel-body" id="target_dir" style="color: #ccc;"></div>
    </div>

    <div class="form-group">
      <input id="ipt_keyword" type="text" class="form-control" placeholder="输入关键字" >
    </div>
    <a href="#" class="button button-primary button-rounded btn-block" id="btn_find">Find</a>

    <p></p>

    <div class="container-fluid">
      <div class="row">
        <div class="col-md-3 col-sm-3 col-xs-6" style="padding-left: 0;">
          <div class="list-group" id="file_list"></div>
        </div>
        <div class="col-md-9 col-sm-9 col-xs-18" style="padding-right: 0; padding-left: 0; ">
          <div class="panel panel-default" style="width: 100%; border:0;">
            <div class="panel-body" style="padding:0;">
              <textarea class="form-control" rows="50" id="file_content"></textarea>
            </div>
          </div>
        </div>
      </div>
    </div>

  </body>
</html>
```
> 上述程序中，我遇到了两个难点：一是想在菜单中添加 File -> Open Directory 选项，但是不知道怎么自定义主进程菜单；二是想要通过点击 Open Directory 呼出本地文件系统导航的对话框。另外，还用到了Node.js处理文件IO的相关API。页面UI框架使用了[BootStrap](http://www.bootcss.com/)，在依赖jQuery时需要特殊处理，详见代码Line_xxx。

***

** 运行程序，跑起来！**

要运行Electron程序非常简单，只需要cd到程序根目录，然后执行：
```
electron .
```
** 注意：** 不要漏掉后面的句点，句点和electron命令之间有一个空格。

有时候在运行时会报错，这可能是由于你的package.json文件格式不规范（例如：在数组的最后一个元素后面多加了一个逗号什么的），也有可能是main.js文件中错误地引用了脚本库，或是没有按照前面所说的在渲染进程中使用remote方式调用主进程。

> ** 调试小技巧：**
>
> 1. 在刚刚的main.js中，有一行代码：mainWindow.webContents.openDevTools(); 这是用来默认开启Chrome的调试控制台的，如果吓到你了，将其注销掉即可。
>
> 2. 如果默认关掉上述控制台了，而你有时又希望查看控制台日志，那么可以在应用程序主界面通过 Ctrl+Shift+I 快捷键将其呼出。
>
> 3. 界面上的中文乱码是否吓到你了？很简单，在 index.html 的头部加入meta为utf-8声明即可。
```html
<meta charset="UTF-8" />
```

***
** 打包和发布！ **

我们总不能让用户也通过命令行输入 electron 来执行应用程序，因此下面的任务是将代码打包（防止源代码泄露），并发布成可执行文件，这里我只做了Windows平台下的教程，如果想了解更多，请点击[这里](https://github.com/atom/electron/tree/master/docs)。

***
** 结语 **

使用Node.js开发桌面APP的好处是摆脱了重量级的IDE（Visual Studio | Eclipse .etc），并通过极其轻量级的方式完美实现了跨平台一致性。这一度让我产生了Node.js将带来桌面APP革命的错觉。

然而，在开发过程中，我逐渐认识到：框架开放的系统接口数量有限，打包后程序体积过大（上述这个最简单的程序都达到60M+），都将成为阻碍Node.js开发桌面APP的桎梏。尤其是后者，要在打包过程中加入整个Node运行时环境，这几乎是不可改善的。因此，对于Node.js开发桌面APP的前景还需要冷静对待。

与 Electron 类似的框架还有 [heX](http://hex.youdao.com/zh-cn/index.html) 和 [NW.js](http://nwjs.io/)，前者是网易前端团队开发的开源框架，不过已经4年没有更新了，可见不受重视。后者的背后支撑力来自于Intel，从GitHub提交记录来看，目前也和Electron一样比较活跃，貌似文档没有Electron那样详尽，不过听闻开放了更多的系统接口，有时间可以了解一下。
