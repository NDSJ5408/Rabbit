# Day2

## pinia

其实就相当于一个管理器，管理公用的js文件，逻辑。是谁都可以用的公用组件

## pinia中的getters实现

可以选择直接使用computed函数进行返回

## storeRefs

可以让数据保持响应式解构，解构后可以直接使用，不用再被调用

const{数据}=storeRefs（对象参数）

对于方法直接解构

const {方法}=对象

## 项目配置git管理

![image-20250904212352531](C:\Users\PC\AppData\Roaming\Typora\typora-user-images\image-20250904212352531.png)

## ElementPlus引入

对于项目的组件分成两类

- 一类是通用的可以直接使用elementPlus的组件
- 一类是定制的组件

根据文档安装就好

## elementPlus定制主题

![image-20250904214027795](C:\Users\PC\AppData\Roaming\Typora\typora-user-images\image-20250904214027795.png)

根据文档，在vite中配置

## 路由设置

页面整体切换是一级路由，头部不变，只变内容是二级路由

![image-20250904214402066](C:\Users\PC\AppData\Roaming\Typora\typora-user-images\image-20250904214402066.png)

对于二级路由，设置path为空，是默认页面。一级也是

