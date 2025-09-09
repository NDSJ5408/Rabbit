# Day4

## 逻辑函数拆分业务

实际的业务逻辑封装到了不同的页面

**use打头内部封装逻辑，return数据和方法给组件**

## 路由缓存问题

是为了优化，在页面跳转的时候，不变的模块不用请求

**方法一**

给router-view添加key

![image-20250909205226820](C:\Users\PC\AppData\Roaming\Typora\typora-user-images\image-20250909205226820.png)

**方法二**

使用beforeRouteUpdate导航钩子

在每次路由更新之前执行，在回调中执行需要数据更新的业务逻辑

## 轮播图

![image-20250909205645765](C:\Users\PC\AppData\Roaming\Typora\typora-user-images\image-20250909205645765.png)

技术难点，**存疑**

## 路由配置

在router的js页面：配置：

![image-20250909210040910](C:\Users\PC\AppData\Roaming\Typora\typora-user-images\image-20250909210040910.png)

在组件里的RouterLink

![image-20250909210750581](C:\Users\PC\AppData\Roaming\Typora\typora-user-images\image-20250909210750581.png)

## 封装组件props传入

```
<li v-for="goods in cate.goods" :key="goods.id">
            <GoodsItem :goods="goods" />
          </li>
```

只需要传入goods就可以无限替换

## 懒加载

为了让网页，图片被优化

只有进入到视口区才会有图片的发送

```
<img v-img-lazy="">
```

在directives里面定义插件

```
//懒加载
import { useIntersectionObserver } from '@vueuse/core'


export const lazyPlugin={
  install(app){
   app.directive('img-lazy',{
  mounted(el,binding){
    //el指令指定的元素
    //binding：指令等于号后面绑定的表达式的值 图片url
     console.log(el,binding.value)
     const {stop}=useIntersectionObserver(
      el,([{isIntersecting}])=>{
        console.log(isIntersecting)
        if(isIntersecting){
          el.src=binding.value
          stop()
        }
      }
     )
  }
})
  }
}

```

这个插件就送img-lazy

在组件里面图片地方直接使用v-img-lazy

const {stop}解构得到stop方法，拿到图片过后，不再重复加载。防止内存浪费
