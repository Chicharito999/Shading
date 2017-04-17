# Stickman
:sunny:Realize at least two different method such as Phong model, Gouraud model<br>
__________________________________________________________________________________________
Author:赵明福                                        Student ID：201400301087                            E-mail:1109646702@qq.com<br>
## Assignment
Here comes the new project.This project may be a little difficult.You have 2 choices: <br>
1:Based on last project:Let's your sitck man move<br>
2:Create your own scene,you can place anything in it<br>
Choose one and use at least two different method such as Phong model, Gouraud model to complete this project.<br>
![](https://github.com/Chicharito999/ImageCache/raw/master/image/图片24.png) 
![](https://github.com/Chicharito999/ImageCache/raw/master/image/图片25.png)
## Analysis
绘制木头人：
* glutWireSphere(R, 20, 20);//渲染一个半径为R 横线竖线为20的球体
* glScaled(l, w, h);//当前图形沿x,y,z轴分别放大为l，w，h倍   
* glutWireCube(1);//画边长为1的一个正方体
* glutWireCube(1)+glScaled(l, w, h)//绘制出一个长宽高为lwh的长方体
* glClipPlane(GL_CLIP_PLANE0, eqn);//设定切割模式为切割下半部分
* glutWireSphere(R, 20, 20);//渲染一个半径为R 横线竖线为20的球体
* glClipPlane(GL_CLIP_PLANE0, eqn)+glutWireSphere(R, 20, 20)//绘制一个球体的上半部分
 
思考人在走路或奔跑时的特征：
* 开始时，左右腿中一只腿先动，同时相反方向的胳膊开始向上摆动
* 随后，当先开始摆动的腿或胳膊往回摆时，另一对腿和胳膊开始向上摆动
* 胳膊在上摆动时，开始总是大小胳膊一起旋转，当旋转到一定角度时，大胳膊保持不动，小胳膊继续旋转一定的角度
* 胳膊在下摆动时与上面相反，小胳膊先向回旋转，随后大小胳膊一起复位
* 腿在摆动时，大腿旋转的角度远远大于小腿，尤其是在奔跑时小腿可以看作是一直垂直与地面
* 无论是胳膊还是腿向后旋转的角度并不大，可以考虑也可以忽略
* 当胳膊摆动到最高点时向前迈出一步
  
glPushMatrix()、glPopMatrix()：<br>
　　由于小胳膊既会自己摆动也会和大胳膊一起摆动，所以要用层次的思想来解决，opengl提供了glPushMatrix()、glPopMatrix()函数用于层次化绘制glPushMatrix()的作用是把矩阵压入栈中保存起来，留着以后再用，就好象把子弹压入枪膛一样，同时栈顶复制一份被压入的矩阵。所以调用glPushMatrix()之后的操作相当于在上一层矩阵的基础之上进行的操作，同理上一层的矩阵操作对下一层也有效。glPopMatrix()函数可以将栈顶的矩阵移除，下一层成为当前层，相当于把弹夹中最上面的子弹发射出去，让下面那颗子弹成为弹夹最顶上那颗。所以，大、小胳膊摆动的问题可以用下面的方式来解决：<br> 
```cpp
glRotatef(i, -1, 0, 0);//控制上胳膊与下胳膊旋转i度  
glPushMatrix();//矩阵压栈并在栈顶复制一份
glTranslatef(0.0, -1, 0.0);//移动到右下胳膊处
glRotatef(ii, -1, 0, 0);//控制下胳膊旋转ii度 
drawSkewed(1, 1.5, 1, WIRE);//绘制右下胳膊
glPopMatrix();//栈顶矩阵出栈  回到大胳膊处
drawSkewed(1, 1.5, 1, WIRE);//绘制右上胳膊
glPopMatrix();//栈顶矩阵出栈  回到右肩膀处
```

具体考虑旋转与位移：<br>
* 先让左大、小胳膊共同旋转i（0-45）度，然后左小胳膊独自向上旋转ii（0-15）度；与此同时右大、小腿旋转，旋转角度与胳膊同步也为i，但是右小腿同时往回旋转i度，使小腿保持垂直
* 然后小胳膊开始往回旋转ii（15-0）度，然后大小胳膊同时往回旋转i（45-0）度；与此同时右大、小腿往回旋转，角度与胳膊同步角度为i，右小腿为-i
* 当左小胳膊往回旋转到0度即ii=0时，右大小胳膊、左大小腿开始旋转，旋转情况与前面相同（ii1、i1）
* 当左小胳膊或者右小胳膊到达最高度15度时，向前移动一步（des）<br>
角度、位移控制代码：<br>
```cpp
	//i控制右胳膊和左腿  ii控制右小胳膊
	if (i < 0) {//大胳膊 小胳膊旋转到小于0度时  开始增加i
		a = 1;
	}
	if (i == 45) {//大胳膊 小胳膊旋转到45度时  保持i不变 开始调整ii
		a = 0;
		if (ii > 15) {//小胳膊旋转到大于15度时  开始减小ii
			b = -1;
			
		}
		if (ii < 0) {//小胳膊旋转到小于0度时  开始减小i  同时将b置为1
			b = 1;
			a = -1;
			c = c + 1;//i摆回到0度时c非0
			
		}
		ii = ii + b;
	}
	i = i + a;

	//当c非0时开始摆动左胳膊和左腿
	//i1控制左胳膊和左腿  ii1控制左小胳膊
	if (c != 0) {
		if (i1 < 0) {
			a1 = 1;
		}
		if (i1 == 45) {
			a1 = 0;
			if (ii1 > 15) {
				b1 = -1;

			}
			if (ii1 < 0) {
				b1 = 1;
				a1 = -1;
			}
			ii1 = ii1 + b1;
		}
		i1 = i1 + a1;
	}
	//小胳膊旋转到15度时 开始移动一步
	if (ii1 ==15||ii==15) {
		des++;
	}
```

## Display
![](https://github.com/Chicharito999/ImageCache/raw/master/image/EV~2017.04_clip2.gif)<br>
![](https://github.com/Chicharito999/ImageCache/raw/master/image/EV~2017.04_clip3.gif)<br>
![](https://github.com/Chicharito999/ImageCache/raw/master/image/EV~2017.04_clip4.gif) 
 
