# 介绍

# 可想造一个属于你自己的 jQuery 库?

> 作者：[MeCKodo](https://github.com/MeCKodo)
> 
> 来源：[forchange](https://github.com/MeCKodo/forchange)

* * *

*   [x] 0.[讲解基础框架格式](https://github.com/MeCKodo/forchange/tree/master/lesson-0)
*   [x] 1.[初步体验](https://github.com/MeCKodo/forchange/tree/master/lesson-1)
*   [x] 2.[新增 next,prev,parent,parents](https://github.com/MeCKodo/forchange/tree/master/lesson-2)
*   [x] 3.[完善 init 方法](https://github.com/MeCKodo/forchange/tree/master/lesson-3)
*   [x] 4.[新增必备方法 each](https://github.com/MeCKodo/forchange/tree/master/lesson-4)
*   [x] 5.[新增 find,last,eq,get,first,ajax](https://github.com/MeCKodo/forchange/tree/master/lesson-5)
*   [x] 6.[完善 hasClass 和 css 方法 新增 data 和 attr 方法](https://github.com/MeCKodo/forchange/tree/master/lesson-6)
*   [x] 7.[新增 html，remove，after，append，before 方法](https://github.com/MeCKodo/forchange/tree/master/lesson-7)
*   [x] 8.[引入 delegate 机制](https://github.com/MeCKodo/forchange/tree/master/lesson-8)
*   [x] 9.[如何实现 on 与 off](https://github.com/MeCKodo/forchange/tree/master/lesson-9)
*   [x] 10.[实现事件委托](https://github.com/MeCKodo/forchange/tree/master/lessonn-10)
*   [x] 11.[最后一节补充 width,height,extend](https://github.com/MeCKodo/forchange/tree/master/lessonn-11)

## 前言

> *   1.给一些很想自己实现一个 jQuery 或者是对实现 jQuery 非常好奇的人
> *   2.想提升自己 js 基础的小伙伴
> *   3.本教程系列不考虑兼容和性能问题,只考虑如何利用各种巧妙的方法去实现一个一模一样的 API
> *   4.从 dom 操作一直到事件机制`on(),off()`,全会逐一实现(事件机制是本人思考后的另类设计,纯原创)

本人一直很想自己造个 jQuery 的小库,第一是满足下自己,第二是去体验下 jQuery 内部的基情!

虽然 jQuery 很多源码看不懂,但是凭借着对 jQuery 的 API 实现的效果,我也基本实现了这样一个类库.

由于自己看别人源码的时候经常会想,作者要是能一步一步的告诉我他是怎么写怎么想的就好了 :).

接下来,我会在每一个 version 里写下我每一步的想法,让你了解到你如何也能自己造一个这样的轮子.

希望我的做法能给你带来许多的启发.(即使我在里面写的代码实在是不值得一提)

另外您的 star,是我的最大动力!

### TO DO LIST

*   [x] 1.css 操作
*   [x] 2.class 操作
*   [x] 3.attr 和 data 操作
*   [x] 4.简单的 dom 选择
*   [x] 5.dom 操作
*   [x] 6.ajax
*   [x] 7.each 循环
*   [x] 8.after && before 插入
*   [x] 9.事件委托
*   [x] 10.tap 实现(番外篇)
*   [x] 11.简单的一些手势（番外篇）