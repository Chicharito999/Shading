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
### 冯氏反射模型：
冯氏反射模型是基于单平行光源，不考虑光的连续折射变化。<br>
光的强度由R、G、B 3个分量组成，每个分量在[0,1]之间变化；物体表面的材质也由R、G、B 3分量表示，每个分量表示反射光的比例。两者的点乘才表示光照射物体后反射的光进入眼睛感受到的颜色。<br>
Phong Reflection Model是经典的光照模型，它计算光照包括三个部分：环境光+漫反射光+镜面光。<br>
所以，要展现出经过光的照射后物体表面的颜色，要先计算这3类光的RGB分量，然后再与物体表面反射光的RGB分量相乘的总和得到照射后的颜色。注意，对于不同类别的光，物体表面反射的RGB分量不一定相同。<br>
* 环境光：环境光是场景中光源给定或者全局给定的一个光照常量，它一般很小。环境光假设入射光均匀地从周围环境入射至物体表面并等量地向各个方向反射出去。<br>
* 漫反射光：表示特定点光源在物体表面某一点的反射光中那些向空间各个方向均匀反射出去的光。漫反射光的亮度与入射光线与表面法向量的余弦成正比。<br>
* 镜面反射光：镜面光成分模拟的是物体表面光滑时反射的高亮的光，镜面光反映的通常是光的颜色，而不是物体的颜色。镜面反射光的亮度与反射光线和人的眼睛与物体表面的连线成正比。<br>
![](https://github.com/Chicharito999/ImageCache/raw/master/image/图片27.png)<br>
![](https://github.com/Chicharito999/ImageCache/raw/master/image/图片26.png)<br>
得到3类光的RGB分量之后，与物体表面材质的RGB反射分量相乘即可获得光照射之后物体表面颜色。<br>
 
### CG编写Shader：
如果在openGL中使用CG(C for Graphic)语言，首先要下载并安装 NVIDIA的Cg Toolkit，然后在项目中的附加包含目录中添加头文件目录，在附加库目录中添加库文件目录，在附加依赖项中添加cg.lib cgGL.lib就可以在程序中使用了。<br>
着色器可以分为顶点着色器和片段着色器。顾名思义，顶点着色器是处理各个顶点的程序，而片段着色器处理片段，负责输出每个呈现三角形的最终像素颜色。<br>
顶点着色器的输出参数可以说是直接作为了片段着色器的形参传递过来。它的工作原理如下：片段着色器以输入的形式收到顶点着色器传递的所有这些片段，所以到达片段着色器的片段是顶点着色器的顶点属性输出的插值版本。<br>

### 古罗着色 冯氏着色 Blinn冯氏着色：
* 古罗着色：又叫做逐顶点着色，故名思意跟顶点有关，也就是在我们的顶点着色器中根据每个顶点上的入射向量L、法向量N、观察向量V等直接计算出每个顶点该有的颜色，然后传递给片段着色器进行插值着色，可想而知由于顶点是离散的，片段是连续的，所以引起着色效果的不光滑很容易理解。<br> 
* 冯氏着色：与古罗着色对应即是在片段着色器中，对法向量与坐标进行插值，然后再通过冯氏反射模型计算出每个像素点的颜色值，从而使离散的顶点计算出来的离散的颜色变得连续而光滑。我们直接把环境光、漫反射光、镜面反射光的计算拿到片段着色器中计算即可完成修改，那么法向量、观察向量、入射向量同理需要传递给片段着色器，而不再是直接传递一个颜色。<br> 
* Blinn冯氏着色：
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
 
