##简介
Flappy Bird是一款非常优秀且容易上手的游戏，可以作为一个很好的HTML5游戏的教程。在这片教程中，我们使用Phaser框架写一个只有65行js代码的简化版Flappy Bird。

点击此处可以先体验一下我们即将要制作的游戏。

提示1：你得会JavaScript  
提示2：想学习更多关于Phaser框架的知识可以看这篇文章[getting started](http://blog.lessmilk.com/make-html5-games-with-phaser-1/)

##展示
![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid100000labid1202timestamp1437457926884.png?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

如果没有耐心的同学，只是想玩玩
 1. wget http://labfile.oss.aliyuncs.com/courses/373/flappybird.zip
 2. unzip fapplybird.zip
 3. cd fapplybird
 4. firefox index.html


##制作流程


要下载一个为大家准备的空的模板  
1. wget http://labfile.oss.aliyuncs.com/courses/373/basic_template.zip
2. 列表项unzip basic_template.zip


文件结构
 - phaser.min.js  简化了的Phaser框架v1.1.5
 - index.html     开始玩游戏的地方
 - main.js        我们写代码的地方
 - assets/        来保存附件的目录

在main.js里面可以看到程序的基本机构，类似于main函数

```
// 初始化phaser框架，创造一个400X490的div用来装游戏
var game = new Phaser.Game(400, 490, Phaser.AUTO, 'game_div');
 
// Creates a new 'main' state that will contain the game
var main_state = {
 
    preload: function() { 
        // 加载附件
    },
 
    create: function() { 
        // 创造游戏    
    },
 
    update: function() {
        // 更新：频率设置为每秒60次
    },
};
 
// Add and start the 'main' state to start the game
game.state.add('main', main_state);  
game.state.start('main');
```
接下来我们一次完成preload(),create()和update()方法，并增加一些新的方法。

##小鸟的制作
我们先来看如何添加一个用空格键来控制的小鸟。

首先我们来更新preload(),create()和update()方法。

```
preload: function() {  
    // 改变背景颜色
    this.game.stage.backgroundColor = '#71c5cf';
 
    // 加载“鸟”的图片
    this.game.load.image('bird', 'assets/bird.png'); 
},
 
create: function() {  
    // 显示“鸟”
    this.bird = this.game.add.sprite(100, 245, 'bird');
 
    // 加载重力效果，y轴
    this.bird.body.gravity.y = 1000;  
 
    // 按空格键跳跃
    var space_key = this.game.input.keyboard.addKey(Phaser.Keyboard.SPACEBAR);
    space_key.onDown.add(this.jump, this);     
},
 
update: function() {  
    //当鸟飞的太高或者太低，重置游戏
    if (this.bird.inWorld == false)
        this.restart_game();
},
```

##管子的制作
在preload()中添加管子的载入

```
this.game.load.image('pipe', 'assets/pipe.png');
```
然后再在create()中添加画一组管子的方法。因为我们在游戏中要用到许多管子，所以我们先做一个包含20段管子的组。

```
this.pipes = game.add.group();  
this.pipes.createMultiple(20, 'pipe');
```
现在我们需要一个新的方法来把管子添加到游戏中，默认情况下，所有的管子都没有被激活也没有显示。我们选一个管子激活他并显示他，保证他在不在显示的情况下移除他的激活状态，这样我们就有用不尽的管子了。

```
add_one_pipe: function(x, y) {  
    // Get the first dead pipe of our group
    var pipe = this.pipes.getFirstDead();
 
    // Set the new position of the pipe
    pipe.reset(x, y);
 
    // Add velocity to the pipe to make it move left
    pipe.body.velocity.x = -200; 
 
    // Kill the pipe when it's no longer visible 
    pipe.outOfBoundsKill = true;
},
```
之前的方法只显示了一段管子，但是我们在一条垂直的线上要显示6段，并保证中间有一个能让小鸟通过的缺口。下面的方法实现了此功能。

```
add_row_of_pipes: function() {  
    var hole = Math.floor(Math.random()*5)+1;
 
    for (var i = 0; i < 8; i++)
        if (i != hole && i != hole +1) 
            this.add_one_pipe(400, i*60+10);   
},
```
我们需要每1.5秒调用一次add_row_of_pipes()方法来实现管子的添加。为了达到这个目的，我们在create()方法中添加一个计时器。

```
this.timer = this.game.time.events.loop(1500, this.add_row_of_pipes, this);

```
最后在restart_game()方法的最前面添加下面这行代码来实现游戏重新开始时停止计时器。

```
this.game.time.events.remove(this.timer);

```
现在可以测试一下了，已经有点儿游戏的样子了。


##实现得分和碰撞 
最后一步我们来实现得分和碰撞，这很简单。
在create()中添加下面的代码来实现分数的显示。

```
this.score = 0;  
var style = { font: "30px Arial", fill: "#ffffff" };  
this.label_score = this.game.add.text(20, 20, "0", style);
```
下面的代码放入add_row_of_pipes()用来实现分数的增加。

```
this.score += 1;  
this.label_score.content = this.score;
```

下面的代码放入update()方法来实现每次碰到管子时调用restart_game()。

```
this.game.physics.overlap(this.bird, this.pipes, this.restart_game, null, this);
```
大功告成！

```
firefox index.html
```


这里下载完整的文件http://labfile.oss.aliyuncs.com/courses/373/flappybird.zip

##问题分析
 - 跨界访问  
   如果是在自己的电脑上制作的同学就会发现，加载不出“鸟”和“管子”的图片，如果有人出现了这样的问题，请在下面提问，我就告诉你为啥，没人问就算了

##作业
   做一个属于自己的小鸟吧~





