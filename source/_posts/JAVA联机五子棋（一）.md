---
title: JAVA设计联机五子棋
date: 2023-12-23 22:49:15
tags: JAVA 
---


[TOC]

项目已上传至github   <https://github.com/DemonWhiteY/Gobang_online>

百度网盘 链接：<https://pan.baidu.com/s/1c7dflsJ9Q-Scg-0DVgOdkg?pwd=YYDS>
提取码：YYDS
--来自百度网盘超级会员V3的分享

网盘下载可以看到Server文件夹和Client文件夹，控制台打开 java -jar GObang.jar 注意，要先打开服务端文件，再打开客户端文件，由于是联机游戏，最少需要打开两次客户端才能相互邀请正常游戏。

先来看看游戏最终效果
![屏幕截图 2023-12-25 205853.png](https://s2.loli.net/2023/12/25/zPb7nTuekA5JQXF.png)

联机大厅具有以下功能：

- 更改用户名字
- 用户刷新
- 用户邀请
- 用户更改服务器IP和端口

<br>

![p1.png](https://s2.loli.net/2023/12/24/gzoUHeBYk16ZMrE.png)

游戏主界面具有以下功能

- 玩家交替下棋，对方回合不得下棋。
- 玩家点击下棋，拖拽移动
- 玩家悔棋，最大悔棋次数3
- 认输
- 求和，最大求和次数3
- 显示对战玩家昵称
- 超时，没人30秒，超时警告，超时3次输掉比赛。

# JAVAGUI设计——swing与awt

在实现JAVA制作的五子棋之前，我们要先来补充一些基础知识，其中第一部分就是JAVA中swing与awt组件的知识

## 基本框架——JFream与JPanel

JFream属于JAVAGUI的最顶层组件，故名思意，就是JAVA最高的框架，所有的GUI组件都是写在JFream中的。

JPanel呢，则是相当于一个容器，画板的存在，因为一个JFream上如果存放太多东西容易导致布局的翁乱，因此经常在JFream下嵌套几个JFream来解决这样的问题。

### 创建一个简单的JFream窗口

```java
import javax.swing.*;
import java.awt.Color ;
public class JFrameDemo01{
    public static void main(String args[]){
        JFrame f = new JFrame("第一个Swing窗体") ;
        f.setSize(230,80) ;             // 设置组件的大小
        f.setBackground(Color.WHITE) ;  // 将背景设置成白色
        JLabel label=new JLabel("label");
        f.add(label);
        f.setLocation(300,200) ;        // 设置组件的显示位置
        f.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);      //设置窗口关闭及结束进程
        f.setVisible(true) ;            // 让组件可见
    }
};
```

上述这段代码即为创建一个简单窗口的方法，但仍然有几个需要注意的点

- JFream需要setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE); 很多同学觉得这一步没有太大用处把他删了程序也能运行，但是这有一个很重要的问题就是窗口关闭不代表程序关闭，窗口关闭只是JFream一个实例被关闭，而不是结束程序。
- 另外就是千万记得setVisible(true);

## setLayout

上面我们的程序中我们调用了一个JLabel组件，我们看到它被排在了窗口的最左边，很显然这是随机插入的，那么怎么add就能到我们需要的位置呢，以及如果插入多个Label怎么处理，这就需要我们来给fream/panel，设置模板。

### BoardLayout

## 让程序更多交互——JDialog

JDialog是除JFrame一外的另一个顶级模块，Dialog和Fream的不同之处在于Dialog是依托于JFreamc存在的，即JFrame关闭，JDialog也会关闭。

```java
import javax.swing.JButton;
import javax.swing.JDialog;
import javax.swing.JFrame;
import javax.swing.JLabel;
import javax.swing.JPanel;

public class MyDialogExample {

    public static void main(String[] args) {
        // 创建 JFrame
        JFrame mainFrame = new JFrame("Main Frame");
        mainFrame.setSize(400, 300);
        mainFrame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        // 创建一个按钮，点击时显示对话框
        JButton showDialogButton = new JButton("Show Dialog");
        showDialogButton.addActionListener(e -> showMyDialog(mainFrame));

        // 将按钮添加到 JFrame
        JPanel panel = new JPanel();
        panel.add(showDialogButton);
        mainFrame.add(panel);

        // 设置 JFrame 可见
        mainFrame.setVisible(true);
    }

    // 创建并显示自定义对话框
    private static void showMyDialog(JFrame parentFrame) {
        // 创建 JDialog
        JDialog myDialog = new JDialog(parentFrame, "My Dialog", true);
        myDialog.setSize(300, 200);

        // 在对话框中添加组件
        JPanel dialogPanel = new JPanel();
        dialogPanel.add(new JLabel("This is a custom dialog."));
        myDialog.add(dialogPanel);

        // 设置对话框可见
        myDialog.setVisible(true);
    }
}

```

关于Jdialog,需要强调以下几点。

#### dialog的相对位置

dailog是基于JFraem存在的所以我们在实际开发中希望JFrame的位置最好是相对JFrame的，最好的方式即放在JFrame中间，怎么解决呢？

```java
 JDialog waitDialog=new JDialog(frame);
         waitDialog.setBounds(600,600,200,150);
         waitDialog.setLocationRelativeTo(frame);
         waitDialog.setLocation(frame.getWidth()/2-100,frame.getWidth()/2-50);
```

只需要调用setLocationRelativeTo()函数，再设置Dialog的Location，即为相对位置啦。

## 自定义你的UI组件

![p1.png](https://s2.loli.net/2023/12/24/gzoUHeBYk16ZMrE.png)
可以看到我们实际要做的游戏中的大多数UI控件都不是JAVA swing直接为我们提供的那些丑不拉级的UI控件，而此时就需要我们封装我们自己的UI控件，这里主要提供两种方法供大家使用。

### JPanel+Graphics drawimage

```java
 public  Image background=((Image) new ImageIcon("src/pic/label.png").getImage());
```

然后再paintCompoent函数中绘制背景

```java
protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        g.drawImage(background, 0, 0, this.getWidth(), this.getHeight(), this);
    }
```

### JButton setIcon

```java
ImageIcon icon = new ImageIcon("src/pic/btn.png");//根据路径创建图标
            Image temp1 = icon.getImage().getScaledInstance(update.getWidth(), 
             update.getHeight(), icon.getImage().SCALE_SMOOTH);
            icon = new ImageIcon(temp1);
            update.setIcon(icon);
```

一般适用于JButton和JLabel的创建。

创建完以后我们就发现一个问题，就是BUTTON旁边的白边太突兀，太丑了，我们想把它去掉，怎么办，这个时候不要急，我们只需要下面两行代码就能解决。

```java
this.setBorderPainted(false);
this.setFocusPainted(false);
```

### 如何更新UI组件

有时候我们的面板上的UI组件会需要更新，这是就需要我们调用以下的三件套来实现更新UI的效果。

```java
panel.removeAll();
//重写panel中的组件；
panel.repaint();
panel.updateUI();
```

# JAVA网络编程

然后，应为我们要完成的是联机模式的五子棋嘛，这是就需要我们学习一些网络编程的知识了。

## TCP协议与UDP协议

### TCP（传输控制协议）

- 连接性： TCP是面向连接的协议，意味着在通信之前需要建立连接，然后进行数据传输，最后释放连接。

- 可靠性： TCP提供可靠的数据传输，确保数据的完整性和顺序性。如果有数据包丢失，TCP会进行重传，以保证所有数据都被正确地接收。

- 流控制和拥塞控制： TCP具有流控制和拥塞控制机制，可以根据网络的状态动态调整数据的传输速率，以防止网络拥塞。

- 适用场景： 适用于对数据完整性要求较高的应用，如文件传输、网页访问、电子邮件等。

### UDP（用户数据报协议）

- 连接性： UDP是无连接的协议，通信双方不需要建立连接即可直接发送数据。

- 可靠性： UDP不提供可靠的数据传输，不保证数据的完整性和顺序性。因此，它更轻量级，但也更容易丢失数据包。

- 流控制和拥塞控制： UDP没有流控制和拥塞控制机制，传输速率由应用程序自己控制。

- 适用场景： 适用于对实时性要求较高、可以容忍少量数据丢失的应用，如音频和视频流传输、在线游戏等。

### 优劣比较

#### TCP的优势

- 数据完整性和顺序性得到保障，适用于对可靠性要求高的应用。
- 适用于大文件传输，因为可以确保所有数据正确到达。

#### TCP的劣势

- 较高的开销，包括连接的建立和释放。
- 不适合实时性要求很高的应用，因为连接的建立和拆除都需要一定的时间。

#### UDP的优势

- 低延迟，适用于实时性要求高的应用。
- 简单、轻量级，适用于简单的数据传输场景。

#### UDP的劣势

- 不提供数据完整性和顺序性的保障，应用需要自行处理丢失数据的情况。
- 不适合大文件传输，因为无法保证所有数据都正确到达。

因此选择通讯协议时，需要·根据项目的实际情况来选择TCP和UDP协议，如果是对传输实时性要求高的项目，比如大型MMORPG游戏，就需要使用UDP协议来提升用户体验，如果是对于数据传输顺序要求较高，对实时性要求较低，比如五子棋，就可使用TCP通信。

## Scoket

服务器端

```java
    package socket.socket1.socket;

import java.io.BufferedReader;

import java.io.BufferedWriter;

import java.io.IOException;

import java.io.InputStreamReader;

import java.net.ServerSocket;

import java.net.Socket;

public class ServerSocketTest {

public static void main(String[] args) {

try {

// 初始化服务端socket并且绑定9999端口

            ServerSocket serverSocket  =new ServerSocket(9999);

            //等待客户端的连接

            Socket socket = serverSocket.accept();

            //获取输入流

            BufferedReader bufferedReader =new BufferedReader(new InputStreamReader(socket.getInputStream()));

            //读取一行数据

            String str = bufferedReader.readLine();

            //输出打印

            System.out.println(str);

        }catch (IOException e) {

e.printStackTrace();

        }

}

}

```

客户端

```java
package socket.socket1.socket;

import java.io.BufferedWriter;

import java.io.IOException;

import java.io.OutputStreamWriter;

import java.net.Socket;

public class ClientSocket {

public static void main(String[] args) {

try {

Socket socket =new Socket("localhost",9999);

            BufferedWriter bufferedWriter =new BufferedWriter(new OutputStreamWriter(socket.getOutputStream()));

            String str="你好，这是我的第一个socket";

            bufferedWriter.write(str);

            //刷新输入流

            bufferedWriter.flush();

            //关闭socket的输出流

            socket.shutdownOutput();

        }catch (IOException e) {

e.printStackTrace();

        }

}

}


```

这里要强调一个点就是客户端为什么要调用一个shuttdownOutput()函数,这个函数其实再这里可以替换成socket.close()函数，为什么要调用这两个函数呢，原因就在于socket再执行过程中会形成阻塞，阻塞的函数一个是accept()函数，另一个就是read()函数，以及一切读取的函数，这样代码运行到此处时，如果没有接受到信息，代码就会停在这里不动，直到接收到。

<br>
那么回过头来看我们的代码，如果我们没有给服务端传输客户端已经关闭的消息，服务端的read 就会接着阻塞在那里，等待客户端传输信息，但此时客户端已经关闭进程了，此时就会抛出一个Connection reset的异常

<br>

我们就可以使用客户端和服务器通讯了，成功以后我们会发现客户端只能传输一条语句，就推出了，这明显和我们的预期不符，所以我们接下来就是要实现客户端和服务器之间的实时通讯

## 利用while(true)接收多条消息

服务端

```java
package socket.socket1.socket;

import java.io.BufferedReader;

import java.io.BufferedWriter;

import java.io.IOException;

import java.io.InputStreamReader;

import java.net.ServerSocket;

import java.net.Socket;

public class ServerSocketTest {

public static void main(String[] args) {

try {

// 初始化服务端socket并且绑定9999端口

            ServerSocket serverSocket  =new ServerSocket(9999);

            //等待客户端的连接

            Socket socket = serverSocket.accept();

            //获取输入流,并且指定统一的编码格式

            BufferedReader bufferedReader =new BufferedReader(new InputStreamReader(socket.getInputStream(),"UTF-8"));
            //读取一行数据
            String str;
            //通过while循环不断读取信息，
            while ((str = bufferedReader.readLine())!=null){

                System.out.println(str);
            }

}catch (IOException e) {

e.printStackTrace();

        }

}

```

客户端

```java
package socket.socket1.socket;

import java.io.*;

import java.net.Socket;

public class ClientSocket {

public static void main(String[] args) {

try {

//初始化一个socket

            Socket socket =new Socket("127.0.0.1",9999);

            //通过socket获取字符流

            BufferedWriter bufferedWriter =new BufferedWriter(new OutputStreamWriter(socket.getOutputStream()));

            //通过标准输入流获取字符流

            BufferedReader bufferedReader =new BufferedReader(new InputStreamReader(System.in,"UTF-8"));

          while (true){

String str = bufferedReader.readLine();

              bufferedWriter.write(str);

              bufferedWriter.write("\n");

              bufferedWriter.flush();

          }

}catch (IOException e) {

e.printStackTrace();

        }

}

}


```

# JAVA多线程

### 双线程即时通讯客户端

我们想，我们的客户端想要无时无刻的和服务器通讯，就需要无时无刻的准备收到服务器的消息，因为我们也不知道服务器到底什么时候给我们发消息。

<br>

而在刚刚的代码中，我们又知道了socket的read会发生阻塞，所以如果我们一只检测服务端传来的消息，我们就被迫阻塞在那个地方，没有办法进行其他操作，怎么办，此时就需要我们使用多线程的知识，构建Receive 和Send两个线程来进行客户端和服务器端之间的实时通讯。

## 多用户与服务器的连接

当我们的服务器有多位用户时，是不是每一个用户就需要给分配一个线程？因此动态的加载新线程是很有必要的。
