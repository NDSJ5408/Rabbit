# Day1

## setup

<script setup>

是一个语法糖，是可以不用return的，可以直接在template使用

## reactive  VS  ref

reactive（）：只能接受对象参数。之后又当作是这个相应式的属性，.调用

ref（）：可以放对象可以放普类型，但是必须式调用.value

以上都是响应式对象：当你修改它的值时，框架能**自动知道这个变化**，并**自动更新**依赖于这个值的视图（UI）或其他计算。

## computed

用一个对象接收后直接return

## watch

watch（对象，回调函数，立即执行/深度监听）

1. 可以是多个对象的监听watch([count,name],([newCount,newName],[oldCount,oldName]))
2. 通过watch监听的ref对象默认是浅层，不可以触发回调执行
3. **当你直接监听一个 `ref` 对象（而不是它的 `.value`），并且这个 `ref` 的值是一个对象时，如果你修改这个对象内部的属性（深层次修改），默认情况下**不会触发**watch的**回调函数执行
4. 如果只想深度监听一个属性，则把对象写成函数形式：（）=>对象.属性

## 父传子

1. 父组件在子组件上命名<sonCon message=" ">message就是用来传递消息的信封

2. 子组件通过props注册：

   const props=defineProps（{

   message:String })。需要定义出message的类型

## 子传父

1. 父亲用@来绑定事件：<sonCom @get-message="方法">

2. 儿子使用emit触发事件传递给父亲：

   const emit= defineEmits(['get-message'])

   const SD=()=>{emit('get-message','需要传输数据)}

   其中SD是一个需要被触发的方法，在子页面里面。触发SD方法再触发里面的get-message事件，通过这个这个事件把数据传输给父亲

## 模板引用

![image-20250902214903993](C:\Users\PC\AppData\Roaming\Typora\typora-user-images\image-20250902214903993.png)

## Provide和inject

#### **顶层向底层任意组件传递数据，实现跨组件通信**

**顶层通过provide函数提供数据：**

provide('key',顶层数据)

**底层用inject获取数据：**

const message=inject('key')；

**key就是传递的媒介**

#### **还可以传递方法**

底层想要修改数据就要修改方法。**谁提供的数据谁修改**

provide('key',方法)



## 