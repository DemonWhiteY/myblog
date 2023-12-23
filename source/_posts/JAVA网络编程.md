---
title: JAVA设计联机五子棋
date: 2023-12-23 22:49:15
tags: JAVA 
---
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

# JAVA多线程

## 线程和进程

## 线程的重写

# JAVA网络编程

## TCP协议与UDP协议

## Scoket

## 多用户与服务器的连接

# 实现JAVA五子棋

# MVC模型

# 网络系统

# 消息管理器
