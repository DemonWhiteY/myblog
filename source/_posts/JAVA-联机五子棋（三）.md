---
title: JAVA 联机五子棋（三）
date: 2023-12-24 12:34:22
tags: JAVA
---

# MVC模式

现在我们先不来考虑联机的部分，单纯只考虑五子棋如何实现的，这里用MVC设计模式来实现

## 什么是MVC

### 模型（Model）

模型代表应用程序的数据和业务逻辑。它负责处理数据的存储、检索、更新和验证，同时也包括定义应用程序的业务规则。模型与用户界面和用户交互无关，使得数据的处理可以独立于视图和控制器。

### 视图（View）

视图是用户界面的表示，负责显示数据并将用户的输入传递给控制器。视图接收模型的数据并以某种形式（例如图形界面、命令行界面）呈现给用户。视图通常不包含业务逻辑，而只负责呈现数据。

### 控制器（Controller）

控制器是模型和视图之间的协调者。它接收用户的输入，根据输入更新模型的数据，并相应地更新视图。控制器负责将用户输入转换为对模型的操作，并通知视图更新以反映模型的变化。控制器使模型和视图能够相互独立工作，提高了系统的灵活性。

### MVC模式的工作流程通常如下

- 用户与视图进行交互。
- 视图将用户的输入传递给控制器。
- 控制器根据用户输入对模型进行相应的操作。
- 模型更新后，通知控制器。
- 控制器通知视图更新，以反映模型的变化。
- 视图更新后等待用户下一次交互。

## View

我们来看我们的View部分，View部分主要由两部分组成，一部分是棋盘的主题部分BorderPanel，另一部分是周围的按钮等即toolPanel。

<br>
设置背景图和棋子图片。

```java
 Image background=((Image) new ImageIcon("src/pic/backgroun.png").getImage());

    Image chess1=((Image) new ImageIcon("src/pic/chess1.png").getImage());
    Image chess2=((Image) new ImageIcon("src/pic/chess2.png").getImage());
```

为面板提供交互操作。

```java
public ChessPanel(){
            this.setPreferredSize(new Dimension(800,800));
            this.addMouseMotionListener(new MouseMotionAdapter() {
                @Override
                public void mouseDragged(MouseEvent e) {
                    super.mouseDragged(e);
                    tempPoint=e.getPoint();

                    repaint();
                }
            });
            
            this.setBackground(Color.decode("#000816"));
            this.addMouseListener(new MouseListener() {
                @Override
                public void mouseClicked(MouseEvent e) {
                    System.out.println(e.getPoint().x+""+e.getPoint().y);
                    try {
                        Vars.controler.addChess(e.getPoint());
                    } catch (IOException ex) {
                        throw new RuntimeException(ex);
                    }
                }
                @Override
                public void mousePressed(MouseEvent e) {

                }

                @Override
                public void mouseReleased(MouseEvent e) {
                    try {
                        Vars.controler.addChess(e.getPoint());
                    } catch (IOException ex) {
                        throw new RuntimeException(ex);
                    }
                    tempPoint.x=10000;
                    tempPoint.y=10000;
                }
                @Override
                public void mouseEntered(MouseEvent e) {

                }

                @Override
                public void mouseExited(MouseEvent e) {

                }
            });

    }
```

绘图模块，画线和画棋子

```java
protected  void drawtempchess(Graphics g)
    {
        int cellWidth = Vars.chessPanel.getWidth() / 20;
        int cellHeight = Vars.chessPanel.getHeight() / 20;
        int x,y;
        x= tempPoint.x/cellWidth;
        y= tempPoint.y/cellHeight;
        System.out.println(x+" "+y);
        if(Vars.controler.getColor()){
            g.drawImage(chess1,cellWidth/2+cellWidth*x,cellWidth/2+cellWidth*y,cellWidth,cellHeight,this);
        }
        else{
            g.drawImage(chess2,cellWidth/2+cellWidth*x,cellWidth/2+cellWidth*y,cellWidth,cellHeight,this);
        }

    }

    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        g.drawImage(background,0,0,this.getWidth(),this.getHeight(),this);
        System.out.println(background.getHeight(this));


        int cellWidth = getWidth() / 20;
        int cellHeight = getHeight() / 20;
        ((Graphics2D)g).setStroke(new BasicStroke(3));
       for(int i=0;i<19;i++)
       {
           g.setColor(Color.decode("#0fa99a"));
           g.drawLine(cellWidth,cellWidth*(i+1),cellWidth*(19),cellWidth*(i+1));
           g.drawLine(cellWidth*(i+1),cellWidth,cellWidth*(i+1),cellWidth*(19));
       }

        if(Vars.controler.turnyou)drawtempchess(g);
        LinkedList<chess> data=Vars.model.getModel();
       for(chess chess:data)
       {
           if(chess.color){
               g.drawImage(chess1,cellWidth/2+cellWidth*chess.x,cellWidth/2+cellWidth*chess.y,cellWidth,cellHeight,this);
           }
           else{
               g.drawImage(chess2,cellWidth/2+cellWidth*chess.x,cellWidth/2+cellWidth*chess.y,cellWidth,cellHeight,this);
           }

       }

    }
```

### Model模块

接下来我们看model模块该怎么写，首先我们想我们的视图需要反应模型，那么模型储存这什么呢，就是我们的棋盘数据，将整个棋盘以一个19X19的矩阵存入Model。但是光这样是不行的，应为我们会发现如果这么存的话，我们只知道棋盘上当前的情况，但却不知道这些棋子的顺序，如果这样的话，我们就无法实现我们的悔棋功能了，所以我们需要一个队列容器来记录每一步的棋子都下在了哪里。

<br>

然后我们再次可以提前考虑一下我们之后的联机模块，我们的联机是要与不同的对手对抗的，记录对手的信息以是必要的，还有双方之间的网络消息队列，也可以放在Model模块里

```java
import java.util.LinkedList;

public class Model {
    private LinkedList<chess> model=new LinkedList<>();

    public LinkedList<user> userList=null;

    public  LinkedList<Message>messageModel=new LinkedList<Message>();

    public String username;

    public int enemyID=-1;

    public String enemyname;

    public int ID;

    private int[][]board=new int[22][22];


    public void addChess(int x,int y,boolean color)
    {

            chess newChess = new chess(x, y, color);
            model.add(newChess);
            if (color)
                board[x][y] = 1;
            else
                board[x][y] = 2;

    }

    public  void removeChess()
    {
        board[model.getLast().x][model.getLast().y]=0;
        model.removeLast();

    }

    public LinkedList<chess> getModel()
    {
        return  model;
    }

    public int[][] getboard()
    {
        return board;
    }

    public int getboard(int x,int y)
    {
        return board[x][y];
    }

    public void clear(){
        while(!model.isEmpty())
        {
            model.removeLast();
        }
        for(int i=0;i<22;i++)
            for(int j=0;j<22;j++)
            board[i][j]=0;
    }





}
class chess{
    boolean color;
    int x;
    int y;
    chess(int x,int y,boolean color)

    {
        this.x=x;
        this.y=y;
        this.color=color;
    }
}
```

### Controller模块

Controller 中包括了游戏中所有的操作函数接口。首先我们来分析一下项目中的一些操作需求

#### “下棋”操作

下棋应该是项目中最为基本的内容，下棋需要将棋子存入Model,还需要向对手发送消息，告知对手自己下在了哪一步。

```java
    public void addChess(Point point) throws IOException {
        if(turnyou) {

            int cellWidth = Vars.chessPanel.getWidth() / 20;
            int cellHeight = Vars.chessPanel.getHeight() / 20;
            int x, y;
            x = point.x / cellWidth;
            y = point.y / cellHeight;
            System.out.println(x + " " + y);
            if (Vars.model.getboard(x, y) == 0) {

                Vars.model.addChess(x, y, chessColor);
                if (chessColor) {
                    chessColor = false;
                } else {
                    chessColor = true;
                }
                Vars.chessPanel.repaint();

               Message msg=new Message("chess",x,y,chessColor);
               Vars.model.messageModel.addLast(msg);
                turnyou = false;
                Vars.timePanel.restart();
                Vars.roundPanel.update();
                judgement();
            }
        }

    }
```

#### “对手下棋”操作

同样，我们还需要执行对手下棋的操作，这两个步骤可不可以混合在一起？其实可以，但这样会是代码的可读性变差，而且回合的切换也会没有那么流畅。

```java
public void addNetChess(int x,int y) {

        System.out.println(x + " " + y);
        if (Vars.model.getboard(x, y) == 0) {

            Vars.model.addChess(x, y, chessColor);
            if (chessColor) {
                chessColor = false;
            } else {
                chessColor = true;
            }
            Vars.chessPanel.repaint();
            turnyou=true;
            Vars.timePanel.restart();
            Vars.roundPanel.update();
            judgement();

        }
    }
```

#### “判断胜负”操作

判断胜利： 在每次有玩家落子后，都需要检查是否有一方取得了胜利。检查的方法通常包括检查每个落子位置的水平、垂直、左上到右下、右上到左下四个方向上是否有连续的五个相同的棋子。

- 水平检查： 从左到右检查每一行是否有连续的五个相同的棋子。
- 垂直检查： 从上到下检查每一列是否有连续的五个相同的棋子。
- 左上到右下检查： 从左上角到右下角检查斜线上是否有连续的五个相同的棋子。
- 右上到左下检查： 从右上角到左下角检查斜线上是否有连续的五个相同的棋子。

```java
 public  void judgement() {
            // 示例输入
            int[][] chessboard = Vars.model.getboard();  // 初始化一个21x21的棋盘，初始值都为0

            int winner = checkWinner(chessboard);
            if (winner == 1) {
                System.out.println("白棋获胜！");
               Vars.dialogStyle.Gameover(Start.getStart(),"白棋获胜");
               
            } else if (winner == 2) {
                System.out.println("黑棋获胜！");
                Vars.dialogStyle.Gameover(Start.getStart(),"黑棋获胜");

            } else {
                System.out.println("游戏尚未结束，或者平局！");
            }
        }

        public int checkWinner(int[][] chessboard) {
            // 检查每一行
            for (int i = 0; i < 22; i++) {
                if (checkRow(chessboard, i) != 0) {
                    return checkRow(chessboard, i);
                }
            }

            // 检查每一列
            for (int j = 0; j < 22; j++) {
                if (checkColumn(chessboard, j) != 0) {
                    return checkColumn(chessboard, j);
                }
            }

            // 检查两个对角线
            if (checkDiagonal(chessboard) != 0) {
                return checkDiagonal(chessboard);
            }

            // 游戏尚未结束
            return 0;
        }

        public int checkRow(int[][] chessboard, int row) {
            for (int i = 0; i < 16; i++) {  // 因为是五子棋，所以只需检查前17个位置
                if (chessboard[row][i] == chessboard[row][i + 1] &&
                        chessboard[row][i + 1] == chessboard[row][i + 2] &&
                        chessboard[row][i + 2] == chessboard[row][i + 3] &&
                        chessboard[row][i + 3] == chessboard[row][i + 4] &&
                        chessboard[row][i] != 0) {
                    return chessboard[row][i];
                }
            }
            return 0;
        }

        public int checkColumn(int[][] chessboard, int col) {
            for (int i = 0; i < 16; i++) {  // 因为是五子棋，所以只需检查前17个位置
                if (chessboard[i][col] == chessboard[i + 1][col] &&
                        chessboard[i + 1][col] == chessboard[i + 2][col] &&
                        chessboard[i + 2][col] == chessboard[i + 3][col] &&
                        chessboard[i + 3][col] == chessboard[i + 4][col] &&
                        chessboard[i][col] != 0) {
                    return chessboard[i][col];
                }
            }
            return 0;
        }

        public int checkDiagonal(int[][] chessboard) {
            // 检查左上到右下对角线
            for (int i = 0; i < 16; i++) {
                for (int j = 0; j < 16; j++) {
                    if (chessboard[i][j] == chessboard[i + 1][j + 1] &&
                            chessboard[i + 1][j + 1] == chessboard[i + 2][j + 2] &&
                            chessboard[i + 2][j + 2] == chessboard[i + 3][j + 3] &&
                            chessboard[i + 3][j + 3] == chessboard[i + 4][j + 4] &&
                            chessboard[i][j] != 0) {
                        return chessboard[i][j];
                    }
                }
            }

            // 检查左下到右上对角线
            for (int i = 0; i < 16; i++) {
                for (int j = 4; j < 19; j++) {
                    if (chessboard[i][j] == chessboard[i + 1][j - 1] &&
                            chessboard[i + 1][j - 1] == chessboard[i + 2][j - 2] &&
                            chessboard[i + 2][j - 2] == chessboard[i + 3][j - 3] &&
                            chessboard[i + 3][j - 3] == chessboard[i + 4][j - 4] &&
                            chessboard[i][j] != 0) {
                        return chessboard[i][j];
                    }
                }
            }

            return 0;
        }


        public boolean getColor(){
        return chessColor;
        }

    public void reportColor(){
        if (chessColor) {
            chessColor = false;
        } else {
            chessColor = true;
        }
    }
```

#### 悔棋操作

悔棋和对手悔棋的操作主要的就是看是否是自己的回合，如果是自己的回合，需要需要悔棋两步，如果不是自己的回合，只需要悔棋一步，并更改下棋方。

```java
public void back(){

        if(this.Backnum>0) {
            if(!turnyou) {
                Vars.model.removeChess();
                Vars.chessPanel.repaint();
                Vars.controler.reportColor();
                turnyou=true;
            } else if (turnyou) {
                Vars.model.removeChess();
                Vars.model.removeChess();
                Vars.chessPanel.repaint();

            }
            Backnum--;
        }
    }
```

#### 对手悔棋操作

```java
public void NETback()
    {
        if(!turnyou) {
            Vars.model.removeChess();
            Vars.model.removeChess();
            Vars.chessPanel.repaint();

        } else if (turnyou) {


            Vars.model.removeChess();
            Vars.chessPanel.repaint();
            Vars.controler.reportColor();
            turnyou=false;
        }
    }
```
