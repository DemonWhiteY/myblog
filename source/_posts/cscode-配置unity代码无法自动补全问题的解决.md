---
title: vscode 配置unity代码无法自动补全问题的解决
date: 2023-04-27 10:07:59
tags: [Unity2D,环境配置,游戏,C#]
categories: 
- [游戏引擎,unity,2D]
top_img: https://xinzhuobu.com/wp-content/uploads/2022/06/20220611004.jpg
---

不会还有人用vscode配置unity环境时最后出现代码无法自动补全的问题吧，本人就是这样………直接上手unity，开写代码时候才发现没有代码补全……于是就悲剧了，调整了两天才终于找到解决方法……  

因此写这篇文章来告诉大家怎么解决这种问题，帮大家节省一点时间。  

首先先吐槽一下某度和CSDN，屎山里觅食不是吹的，我搜了两天，愣是没找到对的解决方法，结果一用谷歌，头几个就是可行的方法。  

## 下面首先介绍几种CSDN里的解决办法

打开 Unity，在 Unity 编辑器中点击 Edit -> Preferences -> External Tools

第一行 External Script Editor，选择 Visual Studio Code，如果没有的话点 Browse…，找到 VSCode 的程序即可（不知道在哪里的话找到 VSCode 的任意一个快捷方式，右键，属性，打开文件位置）。

### 在 VSCode 中安装必要的插件  

打开 VSCode，在界面左侧的五个图标中找到 Extensions（最下面那个，找不到的话按快捷键Ctrl+Shift+X），如果你用 C# 写脚本的话搜索并安装这两个扩展：

* C#
* Debbuger for Unity

### 为什么没有代码补全  

很有可能是因为 1.1.4 版本的 Visual Studio Code Editor 不会自动生成 .csproj 文件，所以按照以下步骤生成即可：

在 Unity 中点击 Window -> Package Manager
找到 Visual Studio Code Editor，点击左侧小箭头，点击 See all versions
将其升级为最新版本（目前最新版本是 1.2.1）
重启 Unity
点击 Edit -> Preferences -> External Tools
将 Generate .csproj files for 下方的所有文件都勾上
点击 Regenerate project files
新建一个脚本，会发现代码补全出现了

# 当然以上方法在我的情况下没什么用罢了

## 实际的问题在于 是否安装了合适版本.NET Framework  

按照错误提示，点进.NET的下载网站，下载一个要求的版本的SDK安装就ok了（这里注意，必须是要安装要求的版本，不是最新版本哦）。  

不过肯定有同学对于配置环境这种事情一脸懵逼（比如我），因此我还是给大家准备了傻瓜式的解决办法。  

首先我们需要知道一点，下载unity的时候，正常情况下unity会帮我们配置VS环境的，但现在没有代码补全，问题就出在我们以前可能下载过VS环境，并且在下载这个环境时没有添加unity配置（比如只添加了C++配置）这个时候unity下载时会检测到我们已经有VS环境了，就不会给我们自动配置了，这就需要我们自己手动操作。  

不过手动操作对于小白来说确实难了点，所以我给出了自动和半自动的解决方法。
* 自动
卸载unity和VS重新下载unity自动生成一切需要的配置。
* 半自动
打开VS，点击创建新项目，在下载需要拓展的页面把和unity和C#有关的全部勾选，重新启动unity(亲测有效，vs和vscode都可以补全代码了)
