# Day7-购物车模块

![image-20250916102514128](C:\Users\PC\AppData\Roaming\Typora\typora-user-images\image-20250916102514128.png)

## 本地购物车加入

###   封装pinia

```
import { defineStore } from "pinia";
import { ref } from "vue";
export const useCartStore=defineStore('cart',()=>{
  const cartList=ref([])
  const addCart=(goods)=>{
     const item=cartList.value.find((item)=>goods.skuId===item.skuId)
     //判断真假，判断是否存在
     if(item){
      item.count++
     }else{
      cartList.value.push(goods)
     }
  }

  return{
    cartList,
    addCart
  }
})

```

### 组件里面调用

在购物车按钮里面绑定上

```
            <el-button type="primary" size="large" class="btn" @click="addCart">
```

函数逻辑

```
const addCart=()=>{
   if(skuObj.skuId){
    console.log('打印')
      carStore.addCart({
        id:goods.value.id,
        name:goods.value.name,
        picture:goods.value.mainPictures[0],
        price:goods.value.price,
        count:count.value,
        skuId:skuObj.skuId,
        attrsText:skuObj.specsText,
        selected:true
      })
   }else{
    ElMessage.warning('请选择规格')
   }
}
const skuChange=(sku)=>{
  console.log(sku)
   skuObj=sku
}
```

- carStore.addCart（）传的是**一个对象**，对象可以携带任意数量的键值对。



## 头部列表渲染

首先得到组件模板调用

```
 <div class="item" v-for="i in carStore.cartList" :key="i">
```

之后直接在模板里面调用即可



## 头部列表删除

现在pinia里面封装方法

```
const delCart=(skuId)=>{
    const idx=cartList.value.findIndex((item)=>skuId===item.skuId)
    cartList.value.splice(idx,1)
  }
```

找到当前的skuId，找到后splice删除

在组件里面调用



## 统计计算

把方法封装在pinia里面

```
  const allCount=computed(()=>cartList.value.reduce((a,c)=>a+c.count,0))
    const allPrice=computed(()=>cartList.value.reduce((a,c)=>a+c.count*c.price,0))
```

- `reduce` 就是 **“从左到右，把数组压成一个值”** 的工具。



## 列表购物车

### 在router里配置路由

### 在组件里设置跳转

```
<el-button size="large" type="primary" @click="$router.push('/cartlist')">去购物车结算</el-button>
```

### 选中功能

![image-20250916154201601](C:\Users\PC\AppData\Roaming\Typora\typora-user-images\image-20250916154201601.png)

组件中绑定方法

```
 <el-checkbox :modelValue="i.selected" @change="(selected)=>singleCheck(i,selected)"/>
```

在这里绑定的方法(selected)=>singleCheck(i,selected)把当前这个被选中的i传过去，好改变选中状态，在pinia里面

```
  const singleCheck=(skuId,selected)=>{
   const item=cartList.value.find((item)=>item.skuId===skuId)
   item.selected=selected
  }
```

找到要修改的item

`item` 是 `find` 在遍历 `cartList.value` 时**当前正在检查的那件商品**

判断当前*item*.skuId和传进来的*skuId*相等就返回当前item

### 全选

**pinia**

```
const isAll=computed(()=>cartList.value.every((item)=>item.selected))
  const allCheck=(selected)=>{
    cartList.value.array.forEach(item => item.selected=selected);
  }
```

**组件**

```
 <el-checkbox :modelValue="carStore.isAll" @change="allCheck"/>
```

**统计**

在pinia统一管理

```
const selectedCount=computed(()=>cartList.value.filter(item=>item.selected).reduce((a,c)=>a+c.count,0))
    const selectedPrice=computed(()=>cartList.value.filter(item=>item.selected).reduce((a,c)=>a+c.count*c.price,0))
```



## 接口购物车

### 加入

![image-20250916162406694](C:\Users\PC\AppData\Roaming\Typora\typora-user-images\image-20250916162406694.png)

找到加入购物车的函数，在pinia里

```
  const isLogin=computed(()=>userStore.userInfo.token)
  const addCart=async(goods)=>{
    const {skuId,count}=goods
    if(isLogin.value){
      await insertCartAPI({skuId,count})
      const res=await findNewCartListAPI()
      cartList.value=res.result
    }else{
     const item=cartList.value.find((item)=>goods.skuId===item.skuId)
     if(item){
      item.count++
     }else{
      cartList.value.push(goods)
     }
    }

  }
```

判断有没有登录。

  const isLogin=computed(()=>userStore.userInfo.token)看看有没有用户信息

然后封装两个接口insertCartAPI给后端传数据

findNewCartListAPI把数据拉到前端

### 删除

和加入一样

**优化**

```
 const updatnewList=async()=>{
 const res=await findNewCartListAPI()
      cartList.value=res.result
  }
```



## 退出清空

在userStore退出的时候，也清空购物车

购物车清空函数写在cart的store里面，一定在回调函数里调用useCartStore（）

```
export const useUserStore=defineStore('user',()=>{
const userInfo=ref({})
const cartStore=useCartStore()
const getUserinfo=async({account,password})=>{
const res=await loginAPI({account,password})
userInfo.value=res.result
}
const clearUserInfo=()=>{
  userInfo.value={}
  cartStore.clearCart()
}
return{
  userInfo,
  getUserinfo,
  clearUserInfo
}
},{
  persist:true,
}

)

```

意思是，const cartStore=useCartStore()必须在现在这个回调函数里面



## 合并购物车到服务器

![image-20250916164906380](C:\Users\PC\AppData\Roaming\Typora\typora-user-images\image-20250916164906380.png)

 重写一个merge接口把数据都传过去

```
export const mergeCartAPI=(data)=>{
 return request({
    url: '/member/cart/merge',
    method: 'POST',
    data
  })
}

```

在用户登录的时候合并

```
const getUserinfo=async({account,password})=>{
const res=await loginAPI({account,password})
userInfo.value=res.result
 await mergeCartAPI(cartStore.cartList.map(item=>{
    return{
      skuId:item.skuId,
      selected:item.selected,
      count:item.count
    }
  }))
  cartStore.updatnewList()
}
```

