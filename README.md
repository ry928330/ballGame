# canvas系列二之《接球小游戏》
### 写在最前：
看了canvas的动画系列，已经抑制不住内心的冲动想写个小游戏了，还是那个套路——多写写，你才能了解它。加上这两天下班后我都没有机会去摸摸篮球，所以就写了个接球的小游戏（准确的说不能叫游戏，太简单了，叫动画吧...）。
    都是一些基础的实现，有时间你也可以试试，废话说到这里，我们开始吧。
### git地址：
https://github.com/ry928330/ballGame.git
### 成果展示：
http://note.youdao.com/noteshare?id=1cff79c426b187a5637a9e96d949cb09
### 实现思路：
这里我们采用疑问的句式给出实现的思路（步骤），因为我写这个demo的时候也是这样去想的：
    
```
- canvas上画一个小球，如何动起来？
- canvas上画一个横条儿，如何像在DOM上那样去拖动它？
- 如何实现游戏上面的小方块儿的绘制与删除？
- 碰撞问题：小球与canvas边界之间怎么去判断碰撞？小球与小横条之间怎么去判断碰撞？小球与上方的小方块儿之间怎么去判断碰撞？（其实原理差不多）
```

主要就是这4步的实现，然后把他们串接起来，你的简单接球小动画就实现了。
### 详细说明：
> ##### 如何让小球动起来？
其实这部分是比较简单的canvas动画，一个基本的动画步骤可以归纳为以下几个过程：

```
1.清除canvas：在每次画新的图形的时候，你都必须将之前的图形给清理掉，这样才有那种一帧一帧的重绘的感觉。
2.保存当前的state：在你没走一步时，你都需要将你当前的canvas状态给保存下来，状态包括：当前图形的位置（x,y轴的信息），当前图形的大小（宽高信息），当前图形的变化（也就是你对他做了拉伸，角度变化等等）等信息。
3.重新渲染你在当前位置所要绘制的图形，也就是把你现在想画的东西给它画在你的canvas上。
4.恢复canvas的状态，因为你之间对canvas的信息做了入栈保存，所以此时你必须restore它。
```
你以为这样就可以了吗？哈哈哈，并没有。加入我们把上面的四个步骤封装在一个名字叫draw的函数中，这时要让绘制的图形动起来，还需要借住下面三个函数之一：
    
```
setInterval(function, delay)，setTimeout(function, delay)，requestAnimationFrame(callback)
```
相信学习前端的小朋友对前两个都不陌生，我就不说了，我说说后面这个函数，之前我也没接触这个函数，而且该次demo用的就是这个函数：requestAnimationFrame函数会告诉浏览器你希望执行动画，并请求浏览器调用指定的函数（也就是你传入的回调函数）在下一次重绘之前更新动画。如果你想做逐帧动画的时候，你应该用这个方法，所以当你把draw函数作为会调传入requestAnimationFrame，你的draw函数就会不断的执行。
    
    贴下我的部分代码：
    //ball 对象用来存储一个球
	var ball = {
		x: 150,
		y: 200,
		vx: 5, //水平速度
		vy: 5, //垂直速度
		radius: 20,
		color: 'blue',
		draw: function() {
			ctx.beginPath();
			ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2, true);
			ctx.closePath();
			ctx.fillStyle = this.color;
			ctx.fill();
		}
	};
    function draw() {
		ctx.clearRect(0,0, canvas.width, canvas.height);
    	ball.draw();
    	bar.draw();
    	scores.draw();
    	targetRectangle.draw();
		ball.x += ball.vx;
		ball.y += ball.vy;
		animationJudge = window.requestAnimationFrame(draw);
		...
	}
draw()函数里面包含了小球的绘制，小横条儿的绘制，得分的绘制。
	在每次绘制之前都会清理下整个canvas，而且讲各自的状态保存在了各自命名的对象中，小球会通过水平和垂直速度不断改变它x和y的值来改变他的位置。也就形成了运动的小球。

> ##### 绘制小横条儿，怎么像操作DOM一样去拖动它？
在canvas上是没法儿那么自如的操作DOM元素，但是我们却能对canvas本身进行事件监听，拿到位置信息。所以跟DOM中拖拽的实现类似，在鼠标移动过程中不断的统计移动的距离，然后改变横条儿的位置，
    重新绘制它来达到拖动的效果，照例贴代码：
    
```
canvas.addEventListener('mousedown', function(e) {
		if (!beginGame) {
			draw();
			beginGame = true;
		}
		var x = e.clientX;
		var y = e.clientY;
		//判断拖拽的位置
		if ((x >=bar.x && x <= bar.x + bar.width) && (y >= bar.y && y <= bar.y + bar.height) ) {
			bar.barDragJudge = true;
			bar.xDistance = x;
		}

	})
	canvas.addEventListener('mousemove', function(e) {
		if (bar.barDragJudge) {
			var x = e.clientX;
			var distance = x - bar.xDistance;
			if (bar.x + distance + 60 >= canvas.width) {
				if (distance >= 0) {
					distance = 0;
				}
			} else if (bar.x + distance <= 10) {
				if (distance <= 0) {
					distance = 0;
				}
			}
			bar.x += distance;
			bar.xDistance = x;


		}
	})
	window.addEventListener('mouseup', function(e) {
		bar.barDragJudge = false;
	})
```

监听mousedown事件，当鼠标按下并且鼠标位置是和横条所覆盖的位置重合（当然在我们开始游戏后，会绘制一次横条儿，即页面相应位置会出现横条儿）时，我将拖拽标志barDragJudge设置为true，表示可以进行拖拽了。然后在鼠标移动过程中通过计算鼠标移动的距离更新横条儿的位置，完成横条的拖动，并且判断当横条移动到canvas边界之后不能左拖和右拖。最后，结束点击，将barDragJudge设置为false。

> ##### 如何实现游戏上面的小方块儿的绘制与删除？
其实，光是绘制小方块儿并不能难，你就写一个二维数组，存储你要绘制的矩形方块儿的信息，贴下我的代码：

```
function initialTargetRectangleArr() {
		var targetRectangleArr = [];
		for (var i = 0; i < 4; i++) {
			targetRectangleArr[i] = [];
			for (var j = 0; j < 4; j++) {
				targetRectangleArr[i][j] = {};
				targetRectangleArr[i][j].x = 35 + j*(50 + 10);
				targetRectangleArr[i][j].y = 35 + i*(20 + 10);
				targetRectangleArr[i][j].width = 50;
				targetRectangleArr[i][j].height = 20;
			}
		}
		return targetRectangleArr;
	}
```

上面的函数返回了一个二维数组，数组里面的元素是对象，每个对象包含了你存储的小方块儿的位置以及大小。
接下来说下小方块儿的删除，这里我们先假设小球碰到了我们的小方块儿，碰到之后我们需要讲该方块儿擦除掉，
即借用clearRect函数，然后问题来了，你只是降页面的方块儿擦除掉了，但是方块儿还是在的，这时我采取的办法是降方块儿"移出"我的canvas，然后将他的宽高都设置为0。
	
> ##### 碰撞问题？
这个是本次DEMO的关键问题了，该如何去判断小球和各种东西之间的碰撞呢？我们这里拿小球和底部小横条儿的碰撞来说明下。
其实碰撞的核心在于位置，你要准确的拿到横条儿在canvas中的覆盖区域，然后小球在进入这个区域后就当作是发生了碰撞。照理，贴下代码(关键部分就一句话)：

```
if ((ball.x + ball.vx >= bar.x - ball.radius && ball.x + ball.vx - ball.radius <= bar.x + bar.width) && (ball.y + ball.vy + ball.radius >= bar.y)) {
		ball.vy = -ball.vy;
	}
```

我们稍微来解析下，(ball.x + ball.vx >= bar.x - ball.radius && ball.x + ball.vx - ball.radius <= bar.x + bar.width)
这部分主要判断两个事：&&号左边是判断小球落在横条儿左边时，小球的右边缘必须在横条所覆盖的区域；&&号右边是判断小球落在横条右边时，其左侧必须在横条覆盖的范围内。接着在满足上面这个条件下，(ball.y + ball.vy + ball.radius >= bar.y)判断小球的高度，球心加上当前小球的垂直移动速度以及半径，如果高度值大于等于横条的垂直高度则碰撞发生。怎么样，其实并没有想象中的那么难吧，至于小球和canvas边缘以及小球和上面的小方块儿碰撞原理和这里一样，就不再赘述。

```
关于计分以及碰到红色小块儿加速问题都是比较简单的，可以直接看我的代码，写到这里整个demo差不多就实现了吧。
```


### 写在最后：
刚想到写这个demo的时候还感觉有点困难，主要就是想到碰撞可能不好实现。直到完成后，诶，其实也挺简单的哈。各位大佬如果有时间玩儿玩儿这个小demo的话（现在只能从git上靠代码，自己在页面玩儿），如果存在什么bug之类的欢迎指出哈。（其实我有点担心碰撞边界的问题，设置不好会不会有什么bug出现）接下来，我想试试难度升级，如果有多个小球呢，不停的重绘
会不会导致页面卡顿，恩，当然，这是后话了。最后多谢各位大佬的支持，不多说了，我得打球去了~
