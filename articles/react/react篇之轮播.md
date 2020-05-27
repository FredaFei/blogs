轮播有很多功能，这篇文章主要分析怎么处理无缝轮播。以如下轮播草图为例：

![轮播草图](https://github.com/FredaFei/blogs/blob/master/articles/react/images/slides/1.jpg)

主要方案有以下几种代表：

#### jQ 实现无缝
1. 分别复制1、4（首尾）两个元素的dom；
2. 将复制的元素1插入4的后面，4插入1的前面；
3. 当滑动到队头复制的元素上时，把当前位置设置为真实第一张的位置；
4. 当滑动到队尾复制的元素上时，把当前位置设置为真实最后一张的位置；

DOM结构中共有六张轮播图，通过操作Dom实现无缝。

优点：无论用是否用库或者框架，均支持。

缺点：操作了DOM

#### Vue 实现无缝
1. 展示轮播的窗口（view）设置相对定位；
2. 仅对当前轮播元素可见
2. 利用Vue的`transition` API设置动画，并将类`slide-leave-active`设置当前元素为绝对定位

DOM结构中共有一张轮播图，通过动画实现。[效果预览](https://fredafei.github.io/amazing-ui/components/slides.html)

优点：Vue实现起来方便，不操作DOM

缺点：仅Vue实现起来方便

#### React 实现无缝
1. 展示轮播的窗口（view）设置相对定位；
2. 所有轮播元素绝对定位，叠加覆盖；
3. 根据当前下标和上一次下标确定方向（向前或向后），设置两套方向的CSS `animation`；
4. 根据方向，播放前一个元素的动画和当前元素的动画；

DOM结构中共有四张轮播图，通过动画实现。[效果预览](https://fredafei.github.io/AUI/#/carousel)

优点：不操作DOM，实现仅需CSS+JS。是我目前最推荐的无缝轮播方案。

缺点：暂无

推荐阅读

+ [react 无缝轮播 纯数据驱动](https://codepen.io/shadowwalkerzero/pen/rvGvjJ?editors=1010)