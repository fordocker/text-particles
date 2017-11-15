代码文件
每周一点canvas动画系列文章目前已经更新了12篇，今天给大家发个福利。我们使用canvas来制作一个小的效果。这个小效果是我从codePen上看到的，我对其做了些修改增强，添加了一些新的功能。UI界面就如下图中看到的样子。我们要实现的效果就如我在图中操作的那样，在输入框中输入文字（不管中文，还是英文，还是各种表情也好）都可以在canvas画布中通过众多的粒子组成，在侧边栏中还有很多控件，它们可以控制粒子的各方面属性，以此来形成各种不同的绚丽效果。



### 1.目录结构


### 2.UI界面
UI界面的组成很简单，主要有侧边栏控制台和canvas画布两部分组成
```
<canvas id="canvas"></canvas>
<div id="control">...</div>
```
在侧边栏中有一系列的控制条，他们控制着粒子的各种属性，包括文字输入框：

<input type="text" id="message" value="hahaha" onchange="change()">
控制条

<input type="range" id="gra" value="0" min="-1" max="1" step="0.1" onchange="changeV()">
粒子选择
```
<p style="margin: 0 0 20px 10px;">
      <span id="ball">圆形</span>
      <span id="rect">方块</span>
</p>
```
在这我就不一一列举了！CSS样式文件主要是对UI界面的布局和样式处理，具体请查看代码文件。

### 3.侧边栏滑动
当点击菜单按钮时，侧边栏滑出，再次点击缩回。采用classList来切换滑出和缩回的class,在sidebar.js中
```
var btn = document.getElementById("btn");
var control = document.getElementById("control");

btn.addEventListener('click', function(e){
    control.classList.toggle("slide");
}, false)
```
这样我们的基础界面就搭建完成。下面就到了我们这个动画的核心思想

### 4.准备工作
首先，我们在我们的index.js文件中定义我们需要的一些变量
```
var canvas = document.getElementById('canvas');
    context = canvas.getContext('2d');
    W = canvas.width = window.innerWidth;
     H = canvas.height = window.innerHeight;
    gridY = 7, gridX = 7;
    type = "ball";

var message = document.getElementById('message'),
    gravity = document.getElementById('gra'),
    duration = document.getElementById('dur'),
    speed = document.getElementById('speed'),
    radius = document.getElementById('rad'),
    resolution = document.getElementById('res');

   graVal = parseFloat(gravity.value);
   durVal = parseFloat(duration.value);
   spdVal = parseFloat(speed.value);
   radVal = parseFloat(radius.value);
   resVal = parseFloat(resolution.value);      

colors = [
  '#f44336', '#e91e63', '#9c27b0', '#673ab7', '#3f51b5',
  '#2196f3', '#03a9f4', '#00bcd4', '#009688', '#4CAF50',
  '#8BC34A', '#CDDC39', '#FFEB3B', '#FFC107', '#FF9800',
  '#FF5722'
  ];

function change(){
      。。。
}

function changeV() {
     。。。
}

(function drawFrame(){
    window.requestAnimationFrame(drawFrame, canvas);
    context.clearRect(0, 0, W, H);

    。。。
}())
```
注意这里的的context， W， H等我们定义的是全局变量。
这里有两个变量可能你不知道他是干什么的gridX和gridY，之后我会详细介绍。

### 5.shape.js 文件
这个文件是我们整个动画效果的核心，只有理解了它，你才能了解这个效果的实现原理。因为不是很长，这里我把文件全部列出：
```
function Shape(x, y, texte){
        this.x = x;
        this.y = y;
        this.size = 200;
        this.text = texte;
        this.placement = [];
    }
Shape.prototype.getValue = function(){
         context.textAlign = "center";
         context.font =  this.size + "px arial";
         context.fillText(this.text, this.x, this.y);

         var idata = context.getImageData(0, 0, W, H);
         var buffer32 = new Uint32Array(idata.data.buffer);

         for(var j=0; j < H; j += gridY){
             for(var i=0 ; i < W; i += gridX){
                 if(buffer32[j * W + i]){
                     var particle = new Particle(i, j, type);
                     this.placement.push(particle);
                 }
             }
         }
         
         context.clearRect(0, 0, W, H);
}
```
接下来，我就详细的解一下该文件的代码！首先我们新建了一个构造函数Shape，该构造函数有3个参数:

x , y: 要绘制的文字的位置
texte: 要绘制的文字
我们设置了文字的大小为200px, 并且定义了一个属性placement，这个属性是一个数组。so wired!它是干什么的呢？别急，继续往下走。

接下来我们在原型对象上定义了一个方法getValue.这几行代码：
```
 context.textAlign = "center";
 context.font =  this.size + "px arial";
 context.fillText(this.text, this.x, this.y);
```
很简单，设置文字对其方式，字体大小，并且通过fillText()在canvas上绘制文字。如果此时在控制栏中输入文字，在index.js中新建一个shape对象，并把文字传入，再调用getValue方法就可以看到你已经把输入的文字绘制到了canvas中，当然这时候忽略下面的代码啊！

回到正题，接下来我们调用了context.getImageData()，它是canvas绘制图片的API接口，通过它我们可以得到需要绘制的图片的数据内容。也许你会问，它是用来获取canvas上绘制的图片的数据内容，可是我们这并没有绘制图片啊？

其实，该方法的作用并不只是局限于获取图片的内容。只要canvas上有内容，不管是绘制的文字，还是图形它都能获取，甚至是空白的canvas它也能获取，只不过此时的数据都是0。

那么通过该API获取的内容是什么样的呢？首先，我们尝试获取一张空canvas的内容
```
var canvas = document.getElementById('canvas'),
    context = canvas.getContext('2d');
var imgData = context.getImageData(0, 0, canvas.width, canvas.height);
    console.log(imgData); 
```
结果如下：



我们看到，这里的imgData是一个对象，该对象的第一个属性就是data,是一个8位无符号整数的类型化数组Uint8ClampedArray。打开data看看都有什么,这里我随便打开其中的一个看看。



因为canvas为空，所以数据都为零。现在我们换一下，在canvas中画一个蓝色的矩形。

context.fillStyle = "#49f";
context.fillRect(0, 0, canvas.width,canvas.height);
var imgData = context.getImageData(0, 0, canvas.width, canvas.height);
console.log(imgData);


看看，我们的的数据是不是不为零了！OK! 原理我在这解释的都差不多了，我们回到正题，看下一行代码。

var buffer32 = new Uint32Array(idata.data.buffer);
idata.data.buffer ,在这里我们调用Uint8ClampedArray对象的buffer属性，获取此数组引用的 ArrayBuffer。然后将它传入Uint32Array对象(32位无符号整数值的类型化数组）。此时，我们看看上面绘制蓝色矩形的数据变成什么样了，首先数组长度变为[160000],刚好是上面的8位的四分之一



内容变为



相当于我们把一张图片的分辨率缩小了，以前有640000个数据， 现在只有160000个数据。当然，在本文中数据的内容不是我们所关心的。我们所关心的是在哪有数据。

所以，接下来,就是在有数据的地方，放上我们的粒子
```
for(var j=0; j < H; j += gridY){
             for(var i=0 ; i < W; i += gridX){
                 if(buffer32[j * W + i]){
                     var particle = new Particle(i, j, type);
                     this.placement.push(particle);
                 }
             }
         }

 context.clearRect(0, 0, W, H); //清除所画内容
```
我们遍历整个canvas, 通过buffer32[j * W + i]来判断这个位置的数据是否为空，如果不为空，那么，在这绘制一个粒子。粒子的位置为（i，j）我们作为参数传入。当然你也可以在数据为空的地方放上粒子，看看会出现什么样的效果。

这里用到了gridX和gridY，它们的作用是来判断每个多少个距离取一次数据。学过信号抽样的同学应该很好理解，如果你间隔大，抽样得到的数据就小，反之如果你设定的间隔小，那么抽到的数据就多。在我们的效果中，我们绘制的是文字，同样的道理，间隔小获取的数据就多，粒子就多，组成的文字就完整。间隔大获取的就少。那么粒子组成的文字就不那么完整，这两个变量的值，通过分辨率控件来绑定。思来想去还是上张图吧！



### 6.particles.js 文件
该文件就是我们的粒子文件，我就不做过多解释了,不懂得欢迎提问。

### 7.粒子切换
粒子切换的代码在slide.js中，很简单，就是绑定了两个事件。
```
/粒子切换
var ball = document.getElementById("ball");
var rect = document.getElementById("rect");

function chose(particleName){
    particleName.addEventListener('click', function(e){
        this.style.backgroundColor = "orange";
        (particleName == ball ? rect : ball).style.backgroundColor = "rgba(0, 0, 0, 0)";
        type = (type === "ball" ? "rect" : "ball");
        change();
        
    }, false)
}

chose(ball);
chose(rect);
```
Ok！这个效果的关键点，基本都已经讲完了，有兴趣自己看看吧！！！
