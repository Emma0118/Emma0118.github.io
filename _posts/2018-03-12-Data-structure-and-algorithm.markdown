---
layout: post
title: 前端工作中遇到的数据结构和算法
date: 2018-03-12 10:14:00
---

 **作为一个数学专业毕业的前端开发，有必要好好谈谈这个话题~~ **

## 一、 数据结构及查找算法的实现

1.递归大法

递归(recursion), 顾名思义，就是自己调用自己。一个经典的应用场景就是DOM树查找。

```bash
function getElementById(node, id){
    if(!node) return null;
    console.log(node.id);
    if(node.id === id) return node;
    for(var i = 0; i < node.childNodes.length; i++){
        var found = getElementById(node.childNodes[i], id);
        if(found) return found;
    }
    return null;
}
getElementById(document, "id-data-structure");

```
首先设定DOM树的根节点是document，通过for 循环获取每个子节点并调用函数自身，如果到叶子节点还没有找到，则返回 null，然后再次进入循环，i + 1，继续下一个子节点的查找。

这里首先是查找子节点，一直到叶子节点，如果没有才进入兄弟节点查找，因此，这里是一个深度优先的查找方式。

递归的有点是代码简单易懂，一目了然；然而，效率较低，因为每一次遍历都是从根节点开始一层层遍历到叶子节点，再原路返回查询结果；而且，getElementById函数同时承担了查找方式职责和遍历查找的实现职责，违反了代码设计中“单一职责”的原则。


2.非递归---另一种深度优先算法

非递归有很多形式，我仅使用一种最常用的来展示非递归在DOM树查找中的实现。

设计思路：修改nextElement的查找方式，如果有子节点，则下一个元素就是它的第一个子节点，否则，判断是否有相邻节点，如果有返回他的相邻元素。如果即没有子节点也没有相邻节点，就返回父节点的下一个相邻节点，然后重新进入循环队列。

```bash
<div id="id-data-structure">
    我是 body
</div>

```

```bash
function getElementById(node, id){
   while (node) {
       if(node.id === id) return node;
       node = nextNode(node);
   }
}

function nextNode(node) {
    if(node.children.length) {
        return node.children(0)
    }
    if(node.nextElementSibling) {
        return node.nextElementSibling;
    }
    while (node.parentNode) {
        if(node.parentNode.nextElementSibling) {
            return  node.parentNode.nextElementSibling;
        }
    }
    return null;
}


getElementById(document, "id-data-structure");

```

上面的实现按照我们的设计思路完成查找下一个元素，并将代码封装在nextNode中，这同样是一种深度优先的查找方式。

这种方式在Google浏览器查找元素中得到大量使用，不过Google首先使用m_map保存了所有元素id，然后通过map的方式实现查找，这个map 以id为做为key， 以element node 作为 velue，而map查找唯一元素的方法的时间复杂度是O(1)，这就是我们常说的使用id查询是最快的查找方式的原因。源码如下：

```bash
Map::iterator it = m_map.begin();
while(it != m_map.end()){
    LOG(INFO) << it->key << " " << it->value->element->tagName();
    ++it;
}
```

Google在通过class、tag，以及querySelector中通过单个Id和class查找中同样使用了非递归。看看“”别人家的”代码~~

（1）本体函数：

```bash
 NodeType* currentNode = collection.traverseToFirst();
 unsigned currentIndex = 0;
 while (currentNode) {
    m_cachedList.push_back(currentNode);
    currentNode = collection.traverseForwardToOffset(
        currentIndex + 1, *currentNode, currentIndex);
  }

```

首先获取符合条件的第一个节点，然后不断获取下一个节点，直到NULL,将他们存放在一个list里面，下次获取就不用重复查找了。

（2）nextNode函数实现下一个节点的查找

```bash
ElementType* element = Traversal<ElementType>::firstWithin(current);
while (element && !isMatch(*element))
    element = Traversal<ElementType>::next(*element, &current, isMatch);
return element;
```

先获取第一个节点，如果不符合，则找到下一个节点，下面就是我们刚才提到的通过非递归的方式实现下一个元素。

Traversal :: next 函数， 即上面使用javacript实现的nextNode方法。

```bash
 if (current.hasChildren())
    return current.firstChild();
  if (current == stayWithin)
    return 0;
  if (current.nextSibling())
    return current.nextSibling();
  return nextAncestorSibling(current, stayWithin);
```

复杂选择器查找的实现稍微复杂一些，但关键的查找依然使用了上面nextNode方法~~。

3、哈希结构及相关算法

现在有如个问题：后端接口返回一组图片，图片的id，现在需要根据图片id从这组图片中找到指定id的图片展示出来。这个问题就可以抽象成：在由{key : picId, value : picUrl}组成的数组中根据指定的picId找到对应图片的src。

（1）遍历查找

```bash
function sequentialSearch(sTable, targetPic) {
    for (var i = sTable.length - 1; i >= 0 && sTable[i].id !== targetPic.id; --i);
    return sTable[i];
}
```

上面使用了倒叙查找，某些情况下会比正序查找快些，但最坏情况下时间复杂度仍然为O(n)；


（2）使用javascript实现Hash查找

Hashtable是最常用的数据结构之一，然而，Javascript中并没有这种数据结构。通过使用Javascript的动态添加属性可以实现基于Hashtable的hash查询。

一般接口给的数据以json数组为主，为了将数据变成我想要的那种{picId : 123, picId : src, ... ...}形式，需要对接口数据处理一下。

假设数据结构如下：

```bash
const picArray = [
    {
        picId : '123',
        src : 'hhh'
    },
    {
        picId : '456',
        src : 'mmm'
    }
];
```

我们想要的是当输入‘123’，得到 ‘hhh’。

```bash
function Hashtable() {
    this._hashValue = {};
}

Hashtable.prototype.add = function (inputArray) { //处理接口数据
    for(let i = 0; i < inputArray.length; i++) {
      this._hashValue[inputArray[i]['picId']] = inputArray[i]['src'];
    }
    return this._hashValue;
}


Hashtable.prototype.get = function (key) { //根据id获得src
    if(typeof key === 'string' && this._hashValue[key]) {
        return this._hashValue[key];
    }
}

const createHash = new Hashtable();

createHash.add(picArray);
console.log(createHash._hashValue);
console.log(createHash.get('123')); // hhh
```

这里简单介绍一下哈希查询。

哈希表就是一种以 键-值(key-indexed) 存储数据的结构，我们只要输入待查找的值即key，即可查找到其对应的值。

哈希的思路很简单，如果所有的键都是整数，那么就可以使用一个简单的无序数组来实现：将键作为索引，值即为其对应的值，这样就可以快速访问任意键的值。这是对于简单的键的情况，我们将其扩展到可以处理更加复杂的类型的键。

使用哈希查找有两个步骤:

* 使用哈希函数将被查找的键转换为数组的索引。在理想的情况下，不同的键会被转换为不同的索引值，但是在有些情况下我们需要处理多个键被哈希到同一个索引值的情况。所以哈希查找的第二个步骤就是处理冲突

* 处理哈希碰撞冲突。

由于Javascriot Object数据类型就是基于 Hashtable实现的，所以，当使用使用对象的键查找值时，浏览器引擎已经在底层通过哈希函数将每个键转化为数据的索引，并使用内建的哈希碰撞处理函数处理了碰撞冲突。

在上面的例子中_this.hashValue对象每一个key拥有一个独一无二的index，在javascript底层，我们其实是通过这个index获得想要的src的。

如上所示，哈希查找算法需要一定的时间和空间，在计算机内存足够大时，哈希查找的时间复杂度趋近于O(1)，是一种极有效率的查找算法！如果没有时间限制，那么我们可以使用无序数组并进行顺序查找，这样只需要很少的内存。哈希表使用了适度的时间和空间来在这两个极端之间找到了平衡。只需要调整哈希函数算法即可在时间和空间上做出取舍。如前所述，Google浏览器一般使用哈希查找实现查找唯一元素，ES6中Map数据结构就是一种哈希表结构。

## 二、排序算法及javascript实现

刚才说了一下和查询相关的算法，但是编程中算法应用最广泛的领域当属排序。具体到前端，我们经常遇到切换排序方式，实现按需展示的需求，比如，根据“最新”、“最热”、“评价最高”等来展示相关资讯；在网上商场中通过切换标签按“价格”、“评分”、“销量”、“时间”等展示相关商品等。

常见的设计是前端将排序条件作为请求参数传递给后端，后端将排序结果作为请求响应返回前端。但在实际生产中，受限于服务器成本等因素，当单次数据查询成为整体性能瓶颈时，也会考虑通过将排序在前端完成的方式来优化性能。

1.排序算法

排序算法是计算机科学中的一种最基础的算法，相关描述可以参见 [算法介绍](https://zh.wikipedia.org/wiki/%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95%E3%80%82)。但为了下面叙述方便，我这里简单介绍一下算法中重要的几个方面.

为了下面叙述方便，我这里简单介绍一下算法中重要的几个方面。


一般我们会根据排序的方式来区分算法，比如我们常见的冒泡排序、插入排序、快速排序、归并排序等。一般使用算法复杂度（时间复杂度和空间复杂度）来评价一个算法，并根据实际需要对几种可选算法方案进行取舍。

时间复杂度越低意味着实现一个算法所需要的时间越少，这个算法就越有效率、越有价值，空间复杂度越低意味着实现一个算法占用的计算机内存越少，性能越好。

有时对算法进行取舍时还要考虑一个因素：稳定性。算法一般根据分为稳定性算法和不稳定性算法，没有中间值。稳定算法是指具有相同键值的元素在排序前后的位置是不变的，而不稳定算法正相反。举个栗子：对学生排序时我们规定：先按年龄排序，年龄相同的按学号排序；如果我们采用稳定算法排序时每次排序后年龄相同学号不同的学生的位置是不变的，采用不稳定算法排序时则不能保证相同的结果，因此每次排序后还要再对年龄相同的学生按学号排序。

关于各种算法其时间复杂度和空间复杂度的计算是另外一个大话题，我会在以后的文章中补充。

一（zi）图(ji)胜(qu)千(ling)言(wu)：

![](../assets/images/algorithm.png)

2.排序在前端的应用及Javascript实现

既然说到前端排序，自然首先会想到JavaScript的原生接口 Array.prototype.sort。由于较低的时间复杂度，目前大部分浏览器都通过使用归并排序和快速排序实现sort方法。

从上图可以看出，在平均时间复杂度上两种算法相同，在空间复杂度上，由于归并排序是out-place，快速排序是in-place，所以归并排序的空间复杂度高，但如果不经过改造，归并排序一般是一种稳定排序，在考虑稳定性的情况下占有优势。

我们首先试着使用Javascript通过快速排序实现sort！

3.快速排序实现sort方法

算法思路：

* 先从数列中去除一个数作为“基准”（理论上可以随便选取一个数）

* 实现数组分区：将比这个”基准数“大的数放到“基准”的右边，小于或等于“基准数”的数放到“基准”的左边；

* 再对左右区间重复第二步，直到各区间只有一个数

算法实现：

```bash
/**
 * Created by majie on 17-7-9.
 */

//经典 快速排序 的 实现

const quickSort = arr => {
    if(arr.length <= 1) {
        return arr;
    }
    const pivotIndex = Math.floor(arr.length / 2); //重要！！   算法思路中第一步 基准为位置 理论上可以任选
    const pivot = arr.splice(pivotIndex, 1)[0];
    const left = [];
    const right = [];
    arr.forEach((item, idx) => {
        if(arr[idx] < pivot) {
            left.push(arr[idx])
        }else {
            right.push(arr[idx])
        }
    })
    return quickSort(left).concat([pivot], quickSort(right)); //连接 左数组、基准数和右数组
}

//  const getSort = quickSort([20,44,23,55,77,19]);
// console.log(getSort);

function testTime() {
    console.time('经典排序时间');
    quickSort([20,44,23,55,77,19]);
    console.timeEnd('经典排序时间');
}
testTime(); // 0.173ms;

```


 上面是一种经典的快速排序方法（只不过是Javascript版本的~~）算法的实现，简单易懂。但是我们发现，上面的实现使用了left和right两个数组存放左右两边递归的数据，因此必须分配一块内存！所以，这其实使用out-place实现的快速排序。算法的空间复杂度为O(n)，与理想的O(logn)相差很大。

 在对性能要求高的环境下，必须进一步优化算法，实现in-place排序。

 ```bash
 /**
  * Created by majie on 17-7-10.
  */
 function quickSort(arr) {
     function swap(arr, a, b) {
         const temp = arr[a];
         arr[a] = arr[b];
         arr[b] = temp;
     }
     function partition(arr, low, high) {

         let pivot = arr[low]; //选取固定 基准 为 arr[low]

         let i = low, j = high;
         while (i !== j) {
             while (arr[j] >= pivot && i < j) {
                 j--;
             }
             while (arr[i] <= pivot && i < j) {
                 i++
             }
             if(i < j) {
                 swap(arr, i, j);
             }
         }
         swap(arr, low, i); //将基准数归位
         return i; //返回基准点
     }
     function Qsort(arr, low, high) {
         let pivot;
         if(low < high) {
             pivot = partition(arr, low, high); //获得 基准的 位置
             Qsort(arr, low, pivot - 1); // 对基准位置左边的元素进行排序
             Qsort(arr, pivot + 1, high); // 对基准位置右边的元素进行排序
         }
     }
     Qsort(arr, 0, arr.length - 1);
     return arr;
 }

 function testTime() {
     console.time('worst-case-quickSort排序时间');
     quickSort([20,44,23,55,77,19]);
     console.timeEnd('worst-case-quickSort排序时间');
 }


 const getSort = quickSort([20,44,23,55,77,19]);
 console.log(getSort); [19, 20, 23, 44, 55, 77]

 testTime(); //0.0193ms

 ```

 上面的实现，充分利用swap对数组元素进行原位交换找到最正确的pivot，在保证时间复杂度的前提下使空间复杂度降低到O(logn)。

 然而，可能大家早就已经注意到，在图一中快速排序在最坏情况下时间复杂度退化成O(n^2)。这是什么意思呢？原来，在快速排序中，算法核心是找到一个基准(pivot)——将经过比较交换的数组按基准分解为两个区域然后通过递归继续分解、比较和交换——这也是我们上面实现算法时一直在做的。我们考虑一种情况：我们对一个未知的但已经是正序的数组进行快速排序，如果我们像刚才in-place的做法一样选择第一个或最后一个元素，那么每次都会有一个数区是空的！递归的层数将达到n，最后导致算法的时间复杂度退化为O(n^2)。因此，考虑到这种情况，我们必须在选择pivot时，进行处理。

 我们可以尝试通过随机数选择一个pivot，但依然存在选择的随机数是最左或最右的可能。我们可以再引入一个既不可能为最左元素也不可能为最右元素的元素作为pivot的备选。比如，中间位置的元素！经过处理后我们的中间元素为大小介于最左元素和最右元素的中间值。


 我们在上面的代码的基础上增加这个逻辑。

 ```bash
 /**
  * Created by majie on 17-7-10.
  */
 function quickSort(arr) {
     function swap(arr, a, b) {
         const temp = arr[a];
         arr[a] = arr[b];
         arr[b] = temp;
     }
     function partition(arr, low, high) {
         let middle = low + Math.ceil((high- low) / 2);
         if(arr[low] > arr[high]) {
             swap(arr, low, high)
         }
         if(arr[middle] > arr[high]) {
             swap(arr, high, middle)
         }
         if(arr[middle] > arr[low]) {
             swap(arr, middle, low)
         }
         let pivot = arr[low]; //这时 arr[low] 为左中右三个关键字的中间值

         let i = low, j = high;
         while (i <= j) { //从序列的两端交替和中间元素比较
             while (arr[j] > pivot) {
                 j--;
             }
             while (arr[i] < pivot) {
                 i++
             }
             if(i <= j) {
                 swap(arr, i, j);
                 i++;
                 j--;
             }
         }
         // swap(arr, low, i);
         return i;
     }
     function Qsort(arr, low, high) {
         let pivot;
         if(arr.length > 1) {
             pivot = partition(arr, low, high);
             if(low < pivot - 1) {
                 Qsort(arr, low, pivot - 1);
             }
             if(pivot < high) {
                 Qsort(arr, pivot, high);
             }

         }
     }
     Qsort(arr, 0, arr.length - 1);
     return arr;
 }

 function testTime() {
     console.time('worst-case-quickSort排序时间');
     quickSort([20,44,23,55,77,19]);
     console.timeEnd('worst-case-quickSort排序时间');
 }


 const getSort = quickSort([20,44,23,55,77,19]);
 console.log(getSort);  //[19,20,23,44,55,77]

 testTime(); //0.022ms

 ```

 在我们一步步完善代码后，由于增加了确定“基准”元素的逻辑，运行时间虽有所增加，但在大数量的情况下，考虑到可以有效避免最差数据组合的算法退化，这种时间增加几乎可以忽略不计。

 讲到前端排序，我们自然而然地想到Javascript的sort方法。然而sort方法的实现在各个浏览器中却不尽相同。Firefox/IE采用了稳定的算法实现，而Google的V8引擎则采用了我们刚才研究的快速排序，正如我们刚才的分析，Google采用了同样的方法来规避快速排序最差数据情况下算法退化的情况，只不过Google在更大数据量的情况下做了进一步处理。比如当数据量小于1000时，采取中间元素作为基准点，超过1000后，对1000以外的的元素每隔一个固定的值比如（200）个取值的索引，然后取这些索引的中位数作为基准点的位置。多个数取中位数很简单，这里不做解释，如果感兴趣可以扒来Google的代码研究~~。然而快速排序又是一种不稳定的排序，记得早在Google实现了sort算法不多久，就有人发现的这个问题，Google的解释是，快速排序是in-place排序，内存占用少，引擎性能好，二期快速排序在实际计算机执行环境中比同等时间复杂度的其他排序算法更快（不命中最差组合的情况下）。

 4.归并排序实现sort方法

 对于采取稳定算法实现sort的浏览器，大多数采用了归并排序实现，比如Firefox。归并排序和快速排序的共同点是都采用了“分治”和“递归”的思想——将数组分成两部分然后递归处理。

 归并排序，顾名思义，就是将已经排序好的子序列合并成一个序列，这个过程也成为“二路归并”。

 以图释义：

 ![](../assets/images/algorithm2.png)

 算法实现很简单，这里也简单实现一下。

 ```bash
 /**
  * Created by majie on 2017/7/12.
  */
 function merge(leftArr, rightArr){
     const result = [];
     while (leftArr.length > 0 && rightArr.length > 0){
         //取出最小值，放到结果集中
         if (leftArr[0] < rightArr[0])
             result.push(leftArr.shift());
         else
             result.push(rightArr.shift());
     }
     return result.concat(leftArr).concat(rightArr); //合并，即“二路归并”
 }

 function mergeSort(array){
     if (array.length === 1) return array; //设定 递归结束的标志

     //将数组分成两部分
     const middle = Math.floor(array.length / 2);
     const left = array.slice(0, middle);
     const right = array.slice(middle);

     //递归调用并合并
     return merge(mergeSort(left), mergeSort(right));
 }

 const arr = mergeSort([32,12,56,78,76,45,36]);
 console.log(arr);   // [12, 32, 36, 45, 56, 76, 78]
 function testTime() {
     console.time('mergeSort排序时间');
     mergeSort([32,12,56,78,76,45,36]);
     console.timeEnd('mergeSort排序时间');
 }
 testTime(); //0.0249ms
 ```

由上面的算法也可以看出，归并排序必须递归将数组分成两部分存放，因此其空间复杂度为O(n)。

到这里，我们基本通过快速排序和归并排序实现了Javascript的Array.prototype.sort方法。事实上，即便在我们处理各种复杂算法的时候，这两种排序也因为其优势成为最经常采用的排序算法。值得一提的是，另一种排序，插入排序，在样本数量较少时具有更好的效率，也值得应时而用。比如Google的V8引擎就在数据量小于10时采用了插入排序，充分体现了Google对于性能优化的极致追求。

事实上，每种算法都有自己的优势和缺点，我们要做的是充分分析需求采取最合适的算法，这在复杂多变的前端尤其重要。比如，在对于稳定性要求高的排序中，采用不稳定算法实现的排序结果往往不尽相同，而对于内存、性能要求高的环境中，我们则更倾向于选择空间复杂度低的算法~~





  # 参考文档

  * [](https://zh.wikipedia.org/wiki/排序算法)
  * [](https://zh.wikipedia.org/wiki/Google_Chrome#.E7.99.BC.E5.B8.83)
  * [](https://github.com/v8/v8/blob/master/src/js/array.js)
  * [](https://bugs.chromium.org/p/v8/issues/detail?id=90)
  * [](https://bugzilla.mozilla.org/show_bug.cgi?id=224128)
  * [](https://bugzilla.mozilla.org/show_bug.cgi?id=715181)
  * [](https://mail.mozilla.org/pipermail/es-discuss/2013-June/031276.html)
  * [](https://github.com/Microsoft/ChakraCore/blob/master/lib/Common/DataStructures/quicksort.h)
  * [](http://gs.statcounter.com/#browser-ww-monthly-200809-200809-bar)
  * [](http://ofb.net/~sethml/is-sort-stable.html)
