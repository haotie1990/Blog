# CSS3 Flex使用

#### 1.Flex布局是什么？

Flex是Flexible Box的缩写，意为“弹性布局”，用来为盒型模型提供最大的灵活性。

#### 2.基本概念

采用Flex布局的元素，称为Flex容器（flex container），简称“容器”。它的所有子元素自动成为容器成员，称为Flex项目（flex item），简称“项目”。

![flex](/asset/bg2015071004.png)

容器默认存在两根轴：水平的主轴（main axis）和垂直的交叉轴（cross axis）。主轴开始的位置（与边框的交叉点）叫做``main start``，结束的位置叫做``main end``；交叉轴的开始位置叫做``cross start``，结束的位置叫做``cross end``

项目默认沿主轴排列。单个项目占据的主轴空间叫做``main size``，占据的交叉轴空间叫做``cross size``

#### 3.容器的属性

#### 3.1 ``flex-direction``属性决定主轴的方向（即项目的排列方向）

* row（默认）：主轴为水平方向，起点在左端
* row-reverse：主轴为水平方向，起点在右端
* column：主轴为垂直方向，起点在上沿
* column-reverse：主轴为垂直方向，起点在下沿

#### 3.2 ``flex-wrap``属性定义如果一条轴线排不下，如何换行

* nowrap（默认）：不换行
* wrap：换行，第一行在上方
* wrap-reverse：换行，第一行在下方

#### 3.3 ``flex-flow``属性是``flex-direction``属性和``flex-wrap``属性的简写形式

* row nowrap：``flex-direction: row; flex-wrap: nowrap``
* row wrap：``flex-direction: row; flex-wrap: wrap``
* row wrap-reverse：``flex-direction: row; flex-wrap: wrap-reverse``
* column nowrap：``flex-direction: column; flex-wrap: nowrap``
* column wrap：``flex-direction: column; flex-wrap: wrap``
* column wrap-reverse：``flex-direction: column; flex-wrap: wrap-reverse``

#### 3.4 ``justify-content``属性定义了项目的主轴的对齐方式

* flex-start（默认）：左对齐
* flex-end：右对齐
* center：居中
* space-between：两端对齐，项目之间的间隔都相等
* space-around：每个项目两侧的间隔相等。所以，项目之间的间隔比项目与边框的间隔大一倍

#### 3.5 ``align-items``属性定义项目在交叉轴上如何对齐

* flex-start：与交叉轴的起点对齐
* flex-end：与交叉轴的终点对齐
* center：与交叉轴的中点对齐
* baseline：项目的第一行文字的基线对齐
* stretch（默认值）：如果项目未设置高度或设为auto，将占满整个容器的高度

#### 3.6 ``align-content``属性定义了多根轴线的对齐方式，如果项目只有一根轴线，该属性不起作用

* flex-start：与交叉轴的起点对齐
* flex-end：与交叉轴的终点对齐
* center：与交叉轴的中点对齐
* baseline：项目的第一行文字的基线对齐
* space-between：与交叉轴两端对齐，轴线之间的间隔平均分布
* space-around：每根轴线两侧的间隔都相等。所以，轴线之间的间隔比轴线与边框的间隔大一倍
* stretch（默认值）：轴线占满整个交叉轴

```
align-items和align-content的区别？
```

### 4.项目的属性

#### 4.1 ``order``属性定义项目的排列顺序，数值越小，排列越靠前，默认为0

#### 4.2 ``flex-grow``属性定义项目的放大比例，默认为0，即如果存在剩余空间，也不放大

![flex-grow](/asset/bg2015071014.png)

如果所有的项目``flex-grow``属性都为1，则它们将等分剩余空间（如果有的话）。如果一个项目的``flex-grow``属性为2，其他项目都为1，则前者占据的剩余空间将比其他的项目多一倍

#### 4.3 ``flex-shrink``属性定义了项目的缩小比例，默认为1，即如果空间不足，该项目将缩小

![flex-shrink](/asset/bg2015071015.jpg)

如果所有项目的``flex-shrink``属性都为1，当空间不足时，都将等比例缩小。如果一个项目的``flex-shrink``属性为0，其他项目都为1，则空间不足，前者不缩小。

负值对该属性无效

```
flex-grow和flex-shrink会相互抵消吗？
```

#### 4.4 ``flex-basis``属性定义了在分配多余空间之前，项目占据的主轴空间。浏览器根据这个属性，计算主轴是否多余空间。它的默认值为auto，即项目的本来大小。

#### 4.5 ``flex``属性是flex-grow，flex-shrink和flex-basis的缩写。默认值为``0 1 auto``。后两个属性可选。

建议优先使用这个属性，而不是单独写三个分离的属性，因为浏览器会推算相关值

#### 4.6 ``align-self``属性允许单个项目有与其他项目不一样的对齐方式，可覆盖``align-items``属性。默认值为auto，表示继承父元素的``align-items``属性，如果没有父元素，则等同于``stretch``

* auto
* flex-start
* flex-end
* center
* baseline
* stretch

```
flex-direction是否会影响主轴和交叉轴？
```
### 参考

[Flex 布局教程：语法篇](http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html?^%$)

[Flex 布局教程：实例篇](http://www.ruanyifeng.com/blog/2015/07/flex-examples.html)