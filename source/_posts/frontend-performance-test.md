---
title: 性能专题：（2）如何分析页面性能
date: 2019-12-12 20:28:41
tags: 性能
---

# 使用Chrome开发工具

## 识别内存泄露

开发过程中，最常见的是内存泄露。通过谷歌开发工具，我们可以比较容易的识别出页面是否存在内存泄露。

可以用下面的代码做一次实验：

```html
<!DOCTYPE html>
<html lang="en">
	<head>
	    <title></title>
	    <meta charset="UTF-8">
	    <meta name="viewport" content="width=device-width, initial-scale=1">
	    <link href="css/style.css" rel="stylesheet">
	</head>
	
	<body>
	    <button id="grow">分配内存</button>
	</body>
	
	<script>
	    var x = [];
	
	    function grow() {
	        for (var i = 0; i < 10000; i++) {
	            document.body.appendChild(document.createElement('div'));
	        }
	        x.push(new Array(1000000).join('x'));
	    }
	
	    document.getElementById('grow').addEventListener('click', grow);
	</script>
</html>

```

1. **Performance monitor 实时观测**

`操作流程：command + shift + p, 输入show Performance Monitor`

<!--![](http://q0hfs054j.bkt.clouddn.com/img/20191104170243.jpg)-->
![](https://img11.360buyimg.com/jdphoto/s1950x1032_jfs/t1/65173/13/14780/43089/5dc5333aE182b2321/58a21d5eda07caba.png)

页面稳定后可实时观测图中红框框出来的JS Heap size（堆内存），Dom Nodes（节点数），js event listeners（监听器），如果这几个数值不断增长，可推测出现了内存泄露。

不停点击测试页面的按钮，结果如下图：CPU使用率不停上升到达100%， 堆内存一直上升，节点数也持续上升。代码可以看出，x数组在grow函数中被引用，grow事件又被监听，数组x不会被垃圾回收，相应分配的内存也不会被释放。

<!--![](http://q0hfs054j.bkt.clouddn.com/img/Snipaste_2019-11-05_17-32-20.png)-->
![](https://img11.360buyimg.com/jdphoto/s2028x458_jfs/t1/50221/2/15264/68286/5dc533e2E0155a1bd/ce64b6e9ef08ca51.png)



2.  **Performance录制**

`操作流程：点击垃圾回收-->开始录制-->点击垃圾回收-->结束录制`

<!--![](http://q0hfs054j.bkt.clouddn.com/img/20191104165616.jpg)-->
![](https://img11.360buyimg.com/jdphoto/s1954x1126_jfs/t1/58271/35/15402/76052/5dc53421E4e96fafe/6b0cd8157d139bd7.png)

可以观测图中红框框出来的JS Heap（堆内存），Nodes（节点数），listeners（监听器），如果这几个数值不断增长，可推测出现了内存泄露。

多次点击测试页面的按钮，观测结果见下图。在计数器窗格中可以观测到点击了3次，dom节点（绿色的线）数有三次上涨，每次上涨跃升可以看到堆内存上升，然后在进行垃圾回收时产生了回落，最终红框中的蓝色区域还是比最开始时高出一段，是由于x数组被引用没有释放堆内存。

<!--![](http://q0hfs054j.bkt.clouddn.com/img/Snipaste_2019-11-05_20-39-57.png)-->
![](https://img11.360buyimg.com/jdphoto/s2052x1290_jfs/t1/91662/8/1779/84298/5dc53453Ef4666a0e/960a3815284f0592.png)

3. **使用堆快照确定已分离的DOM树**

操作流程：

1. 	command + shift + p, 输入show Memory，选择Heap snap，开始录制 Snapshot 1
2. 录制结束后，进行可能导致内存泄露的操作
3. 再次录制一次堆快照 Snapshot 2
4. 再次进行可能导致内存泄露的操作
5. 第三次录制快照 Snapshot 3
6. 选择Snapshot 3 和 在Snapshot 1 到 Snapshot 2 之间生成的对象进行检查


<!--![](http://q0hfs054j.bkt.clouddn.com/img/20191104175918.jpg)-->
![](https://img11.360buyimg.com/jdphoto/s1712x1124_jfs/t1/94062/20/1796/101323/5dc5349dE0db6d901/a02261fe1cb0ac78.png)


class filter中输入detaced，可以看到已分离的 DOM 树。在 Object 窗格中，可以看到与正在引用该节点的代码相关的更多信息。 如果存在内存泄露，可以找到相应信息删除不用的节点。

快照最初存储在渲染器进程内存中，点击快照图标进行查看时，快照将根据要求被传输到 DevTools 中。
下图红色圈圈中的数字表示可到达 JavaScript 对象的总大小。

<!--![](http://q0hfs054j.bkt.clouddn.com/img/2019_06_05-15_30_29.png)-->
![](https://img11.360buyimg.com/jdphoto/s865x571_jfs/t1/83392/33/14767/35050/5dc534deEf78aa42a/1c49f65e3e5d2a75.png)

4. **时间轴查看堆内存的分配情况**

同上述打开memory，如下图选择：

<!--![](http://q0hfs054j.bkt.clouddn.com/img/Snipaste_2019-11-06_11-53-08.jpg)--><!---->
![](https://img11.360buyimg.com/jdphoto/s2278x644_jfs/t1/104015/14/1715/79522/5dc53501E87e78d34/40b528397fbc75a0.jpg)

录制之后，可看到下图结果：

<!--![](http://q0hfs054j.bkt.clouddn.com/img/Snipaste_2019-11-06_11-52-27.jpg)-->
![](https://img11.360buyimg.com/jdphoto/s2292x1128_jfs/t1/88147/20/1767/152890/5dc5353bE6157193e/1cd92588bfd90705.jpg)

蓝的竖线表示分配的内存，竖线越高代表分配的越多；
灰色的柱子表示回收的内存。

<!--![](http://q0hfs054j.bkt.clouddn.com/img/Snipaste_2019-11-06_14-10-49.jpg)-->
![](https://img11.360buyimg.com/jdphoto/s2280x1022_jfs/t1/98937/35/1737/120864/5dc5356eE9182776e/6fc34d81fb498db3.jpg)

把时间段缩小到想要观测的蓝色竖线范围，可以圈定当前为哪些对象分配了内存，点击可以看到相关对象的信息，如果判断出现了内存问题，可以找到对应的代码进行修改了。

5. **查看Chrome浏览器的任务管理面板**

Mac上点击Chrome左上角菜单栏 -> 窗口 -> 任务管理器，可以看到chrome打开的页面和扩展程序使用的内存情况。
<!--github图床 token： 1817acd2ec0bbb5f1c055abc23644d7aa9e3286f-->

<!--![](http://q0hfs054j.bkt.clouddn.com/img/20191104163010.jpg)-->
![](https://img11.360buyimg.com/jdphoto/s1226x1030_jfs/t1/52906/39/15184/85272/5dc535aaE91291450/87df6d5465c36cbd.png)

需要关注的是*内存占用空间*和*JavaScript使用的内存*， 如果页面已经加载完成并稳定下来，这两个数值还在不停增长，就可能出现了内存泄露。

## 观测FPS

我们可以使用chrome开发工具的performance记录来查看FPS的情况，下图中红框的是FPS，整一排是页面加载到结束过程中FPS的数值。 衡量页面是否卡顿可以看FPS横条的颜色，如果是绿色，证明页面帧率比较高不卡顿，如果是红色，则说明页面非常卡。 页面空白时间段由于加载及解析js等文件，导致主进程阻塞，这是可以观测到FPS是红色的。

<!--![](http://q0hfs054j.bkt.clouddn.com/img/20191104205848.jpg)-->
![](https://img11.360buyimg.com/jdphoto/s2006x396_jfs/t1/46517/31/15563/46463/5dc532c9Edb27925c/b220d8b212cee485.png)

滑动过程中，如果想实时监测FPS也可以做到。
`操作流程：command + shift + p, 输入show Rendering，勾选FPSmeter`，可以看到页面右上角出现了一个面板，滚动页面FPS的实时数值会显示在面板上。

<!--![](http://q0hfs054j.bkt.clouddn.com/img/20191104210623.jpg)-->
![](https://img11.360buyimg.com/jdphoto/s368x354_jfs/t1/49092/38/15150/32835/5dc532f5E75e4626c/64da63f868d00bc2.png)

## 检查代码覆盖率
`操作流程：command + shift + p, 输入show Coverage`

点击按钮刷新页面，可以看到引用文件的代码覆盖率，绿色部分为使用，红色非使用。如果覆盖率很低，可以在source中打开相应的文件，左侧标红的为未使用部分。

即便使用webpack的tree shaking也还是需要检查覆盖率，webpack跟踪 import/export 语句，优化也仅限于export的代码。如果代码覆盖改率还是很低，并且不是由于延迟加载造成的未使用，就要针对当前文件的红色部分进行细致化的处理了。

![](../../../../img/201912129130.jpeg)

# 使用性能检测工具

开发完页面后，需要检测完性能之后才能安心发布上线，如果性能不过关，需要修改代码重新检测了。推荐几个比较好用的工具:

## lighthouse

[lighthouse](https://github.com/GoogleChrome/lighthouse)是谷歌开源的自动化工具，可以把它作为一个 Chrome 扩展程序运行。 点击扩展程序的小图标即可进行页面性能分析，不得不说，简直太好用了。

<!--![](http://q0hfs054j.bkt.clouddn.com/img/20191104204512.jpg)-->
![](https://img11.360buyimg.com/jdphoto/s2134x1330_jfs/t1/69771/24/14787/44412/5dc535c5E1c7ae9f2/5d6545f40557d394.png)

上面是网页的跑分结果，我们可以继续查看分析报告，查看导致性能变差的原因有哪些。lighthouse会给出一个诊断结果，可以根据建议进行相关的修改。

<!--![](http://q0hfs054j.bkt.clouddn.com/img/20191104204943.jpg)-->
![](https://img11.360buyimg.com/jdphoto/s1940x932_jfs/t1/54138/20/15411/50651/5dc535cbE11750587/caf749b66d7fc22e.png)



### webpagetest

[webpagetest工具网址](www.webpagetest.org)

### GTmetrix

[GTmetrix工具网址](https://gtmetrix.com/)




