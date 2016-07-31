#C3.js: a D3-based chart lib
> C3是一款用于html5上绘制图表的库，基于d3这种高端的库而设计，但是大大减少了使用复杂度。他的依赖文件也只有d3，引入c3时先引入d3即可

> 相对于chart.js,美观上和响应式上并不逊色。但是当我使用chartjs时，第一件事让我干的竟然是写个```<canvas></canvas>```,而且还要加上宽度高度！？excuse me？你不是responsive么－_－||. 而且js里还要让我写getElementById blah blah!!何其不爽。。（当然也可以直接$）

> 吐槽完毕，以上只是非常表面的对比，具体还请自行研究～那来看看c3是怎么简便易用的吧.(这里只介绍直线图，c3还提供很多图形，看这里 http://c3js.org/examples.html )。本例是自己的一点心得，欢迎分享，还有很多不足望指正。作者github:zzs1020

## Setup
```
bower install c3 --save
```
```html
<!-- Load c3.css -->
<link href="/path/to/c3.css" rel="stylesheet" type="text/css">
<!--此处忽略html head body标签,请自行脑补-->
    <div id="chart"></div>
<!--此处忽略html head body标签,请自行脑补-->
<!-- Load d3.js and c3.js -->
<script src="/path/to/d3.v3.min.js" charset="utf-8"></script>
<script src="/path/to/c3.min.js"></script>
```

## Basic Usage
在你的js里写上c3.generate({})就可以用了，里面的参数bindto到id，data就是你要用的数据(这里把数据写死在这里是为了展示，你当然可以从数据库取了数据再传入这个数组)，columns里面有几个数组就有几条线(块，柱...)
```js
var chart = c3.generate({
    bindto: '#chart',
    data: {
      columns: [
        ['data1', 30, 200, 100, 400, 150, 250],
        ['data2', 50, 20, 10, 40, 15, 25]
      ]
    }
});
```
c3也支持AMD API, 如果你用requirejs，也可以相应地load ---> 自己看官网doc
![chart 1](https://github.com/zzs1020/sharedInfo/blob/master/chart1.png)

## Customize chart
你可以自定义很多其他部分：1. Additional Axis 2. Show Axis Label 3. Change Chart Type 4. Format values 。。。

而你要做的也仅仅是在那个data里加上axes，types等literal。举个栗子
```js
var chart = c3.generate({
    bindto: '#chart',
    data: {
      columns: [
        ['data1', 30, 200, 100, 400, 150, 250],
        ['data2', 50, 20, 10, 40, 15, 25]
      ],
      axes: {
        data2: 'y2'
      },
      types: {
        data2: 'bar'
      }
    },
    axis: {
      y: {
        label: {
          text: 'Y Label',
          position: 'outer-middle'
        },
        tick: {
          format: d3.format("$,") // ADD
        }
      },
      y2: {
        show: true,
        label: {
          text: 'Y2 Label',
          position: 'outer-middle'
        }
      }
    }
});
```
![chart 2](https://github.com/zzs1020/sharedInfo/blob/master/chart2.png)
关于自定义样式也可以在css文件里加样式就可以啦，线条变粗神马的好羞射
```css
#chart .c3-line-data2 {
  stroke-width: 5px;
}
```

## Use API
如果一个库不能动态加载数据，那跟咸鱼有什么区别？
```js
chart.load({
  columns: [
    ['data1', 300, 100, 250, 150, 300, 150, 500],
    ['data2', 100, 200, 150, 50, 100, 250]
  ]
});
```
所以当你设置个按钮给第一个例子，点击load时，图表的数据就会更新成新的并自动绘制，自带fade效果，如下
![chart 3](https://github.com/zzs1020/sharedInfo/blob/master/chart3.png)
自然也有unload方法，
```js
chart.unload({
    ids: ['data2', 'data3']
});
```
当然也可以同时load一些数据并unload另一些数据
注意：Please use unload param in load() API if load and unload need to run simultaneously.
```js
chart.load({
        columns: [
            ['data1', 130, 120, 150, 140, 160, 150],
            ['data4', 30, 20, 50, 40, 60, 50],
        ],
        unload: ['data2', 'data3'],
    });
```
你也可以show／hide某条数据(线块柱...)
```js
chart.hide(['data2', 'data3']);
chart.show(['data2', 'data3']);
```
大体就介绍完了，有兴趣的小伙伴可以去尝试一下 http://c3js.org
