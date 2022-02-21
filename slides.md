---
# try also 'default' to start simple
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://source.unsplash.com/collection/94734566/1920x1080
# apply any windi css classes to the current slide
class: 'text-center'
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: false
# some information about the slides, markdown enabled
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# persist drawings in exports and build
drawings:
  persist: false
---

# V8 中的js对象存储和访问

Presentation slides for developers

<div class="pt-12">
  <span @click="$slidev.nav.next" class="px-2 py-1 rounded cursor-pointer" hover="bg-white bg-opacity-10">
    Press Space for next page <carbon:arrow-right class="inline"/>
  </span>
</div>

<div class="abs-br m-6 flex gap-2">
  <button @click="$slidev.nav.openInEditor()" title="Open in Editor" class="text-xl icon-btn opacity-50 !border-none !hover:text-white">
    <carbon:edit />
  </button>
  <a href="https://github.com/slidevjs/slidev" target="_blank" alt="GitHub"
    class="text-xl icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div>

<!--
question
-->

---

# V8 中的js对象存储和访问


- 📝 **问题1** - v8 里的对象是用 散列表/哈希表/字典 存的吗？
- 🎨 **问题2** - 循环对象的属性是按什么顺序输出的？

#### 哈希表
<img src="https://pic4.zhimg.com/80/v2-a54bfa69d5cadec9de1a5340603d6447_hd.jpg" width="400">
<br>
<br>

 

<!--
You can have `style` tag in markdown to override the style for the current page.
Learn more: https://sli.dev/guide/syntax#embedded-styles
-->

<style>
h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
</style>
<!-- code -->
---

# Code


```js
function Foo() {
    this[100] = 'test-100'
    this[1] = 'test-1'
    this["B"] = 'bar-B'
    this[50] = 'test-50'
    this[9] =  'test-9'
    this[8] = 'test-8'
    this[3] = 'test-3'
    this[5] = 'test-5'
    this["A"] = 'bar-A'
    this["C"] = 'bar-C'
}
var bar = new Foo()


for(key in bar){
    console.log(`key:${key}  value:${bar[key]}`)
}
```
<!-- another code -->
---
---
# Code


```js
var a = { 
  1: "a", 
  2: "b", 
  "first": 1, 
  3: "c", 
  "second": 2 
}

var b = {
  "second": 2, 
  1: "a", 
  3: "c", 
  2: "b", 
  "first": 1 
}

console.log(a) 
// { 1: "a", 2: "b", 3: "c", first: 1, second: 2 }

console.log(b)
// { 1: "a", 2: "b", 3: "c", second: 2, first: 1 }
```

<!-- 总结 -->
---

# 规律

1. 设置的数字属性被最先打印出来了，并且是按照数字大小的顺序打印的；
2. 设置的字符串属性依然是按照之前的设置顺序打印的，比如我们是按照 B、A、C 的顺序设置的，打印出来依然是这个顺序。


[一行 Object.keys() 引发的血案](https://juejin.cn/post/7041049741458669576)
<img
  v-click
  class="absolute -bottom-9 -left-7 w-80 opacity-50"
  src="https://sli.dev/assets/arrow-bottom-left.svg"
/>
<p v-after class="absolute bottom-23 left-45  ">
  之所以出现这样的结果，是因为在 ECMAScript 规范就是这样定义的。
</p>

<!-- ecma 规范 -->

---

# ECMAScript 规范

之所以出现这样的结果，是因为 ECMAScript 规范就是这样定义的。

可索引属性应该按照索引值大小升序排列，常规属性根据创建时的顺序升序排列。

在这里我们把对象中的
数字属性称为可索引属性，在 V8 中被称为 elements，  
字符串属性就被称为常规属性，在 V8 中被称为 properties。
<!-- 存储 -->
---
preload: false
---

# 存储
在 V8 内部，为了有效地提升存储和访问这两种属性的性能，分别使用了两个线性数据结构来分别保存排序属性和常规属性


<img width="500" src="https://static001.geekbang.org/resource/image/af/75/af2654db3d3a2e0b9a9eaa25e862cc75.jpg">

---

# 常规属性之对象内属性


<img src="https://static001.geekbang.org/resource/image/f1/3e/f12b4c6f6e631ce51d5b4f288dbfb13e.jpg">


不过对象内属性的数量是固定的，默认是 10 个，
<!-- 
Named properties are stored in a similar way in a separate array. However, unlike elements, we cannot simply use the key to deduce their position within the properties array 
数组、哈希表特点
-->
---

# 常规属性之快属性和慢属性
<img src="https://pic4.zhimg.com/80/v2-f0d59113dfbbf020863500a7ff3d02af_hd.jpg">
<!-- <img width="500" src="https://static001.geekbang.org/resource/image/e8/17/e8ce990dce53295a414ce79e38149917.jpg"> -->


---

# Demo
```js

function Foo(element_num, property_num) {
    //添加可索引属性
    for (let i = 0; i < element_num; i++) {
        this[i] = `element${i}`
    }
    //添加常规属性
    for (let i = 0; i < property_num; i++) {
        let ppt = `property${i}`
        this[ppt] = ppt
    }
}
var bar = new Foo(10,10)
```

---

# 解答

- 📝 **问题1** - v8 里的对象是用 散列表/哈希表/字典 存的吗？

不完全是

- 🎨 **问题2** - 循环对象的属性是按什么顺序输出的？

ECMAscrit 规范  
可索引属性  
常规属性

---


# 总结
1. Array-indexed properties are stored in a separate elements store.
2. Named properties are stored in the properties store.
3. Elements and properties can either be arrays or dictionaries.
4. Each JavaScript object has a HiddenClass associated that keeps information about the object shape.
<img width="500" src="https://static001.geekbang.org/resource/image/4e/6d/4eab311ab4ab94693325a0ca24618b6d.jpg">
[Fast properties in V8](https://v8.dev/blog/fast-properties)
