# Day8

## 配货地址数据渲染

![image-20250917110239091](C:\Users\PC\AppData\Roaming\Typora\typora-user-images\image-20250917110239091.png)

首先封装api接口。之后在组件里面写获取函数以及数组

```
const getCheckInfo=async()=>{
  const res=await getCheckInfoAPI()
  checkInfo.value=res.result
  const item=checkInfo.value.userAddress.find(item=>item.isDefault===0)
  defaultAdress.value=item
}
const checkInfo = ref({} ) // 订单对象
const curAddress = {}  // 地址对象
const defaultAdress=ref({})
 onMounted(()=>getCheckInfo())
```

## 打开弹窗切换地址

![image-20250917110444908](C:\Users\PC\AppData\Roaming\Typora\typora-user-images\image-20250917110444908.png)

使用v-model来打开弹窗

```
 <el-dialog v-model="showDialog" title="切换收货地址" width="30%" center>
```

点击切换地址的按钮后，可以把显示框弹出来

```
<el-button size="large" @click="showDialog = true">切换地址</el-button>
```

## 地址激活

![image-20250917111155278](C:\Users\PC\AppData\Roaming\Typora\typora-user-images\image-20250917111155278.png)

首先绑定@click="switchAddress(item)"在页面上，

```
div class="addressWrapper">
    <div class="text item" :class="{active:activeAddress.id===item.id}" @click="switchAddress(item)" v-for="item in checkInfo.userAddresses"  :key="item.id">
      <ul>
      <li><span>收<i />货<i />人：</span>{{ item.receiver }} </li>
      <li><span>联系方式：</span>{{ item.contact }}</li>
      <li><span>收货地址：</span>{{ item.fullLocation + item.address }}</li>
      </ul>
    </div>
  </div>
```

@click="switchAddress(item)"函数逻辑如下：

```
 const activeAddress=ref({})
 const switchAddress=(item)=>{
  activeAddress.value=item
 }
```

点击哪一个，就存下这个对象，存下这个对象:class="{active:activeAddress.id===item.id}" 用这个代码来控制是否显示样式

### 切换地址

在确定上绑定方法+关闭弹框

```
 const confirm=()=>{
  curAddress.value=activeAddress.value
 }
```



## 生成订单

在api里面封装好接口，在按钮处绑定如下方法

```
 const createOrder=async ()=>{
  await createOrderAPI({
    deliveryTimeType:1,
    payType:1,
    payChannel:1,
    buyerMessage:'',
    goods:checkInfo.value.goods.map(item=>{
      return {
        skuId:item.skuId,
        count:item.count
      }
    }),
    addressId:curAddress.value.id

  })
 }
```

map**“把原数组里的每一个元素，按照你给的规则‘映射’成新元素，最后得到一个新数组。**



之后补充跳转和清空购物车

```
const orderId=res.result.id
  router.push({
    path:'/pay',
    query:{
      id:orderId
    }
  })
  cartStore.updatnewList()
```

| 对比维度             | query（查询参数）            | params（路由参数）                      |
| -------------------- | ---------------------------- | --------------------------------------- |
| **URL 表现**         | `/#/pay?id=123` **问号后面** | `/#/pay/123` **路径的一部分**           |
| **路由配置**         | 不需要改路径                 | 需要先在路由表写 `path: '/pay/:id'`     |
| **是否必须事先声明** | 不需要，随时可加             | 必须先在路由里占位                      |
| **刷新页面**         | 参数 **不会丢**              | 若路由表里没配路径，刷新就 **404/丢参** |
| **可见性/书签**      | 完全可见，可收藏/分享        | 同样可见，可收藏                        |
| **传对象/数组**      | 可以（会转成字符串）         | 只能传字符串（需自行序列化）            |
| **编程方式**         | `name` 或 `path` 都能用      | **只能**用 `name` 跳转，不能用 `path`   |
| **典型场景**         | 分页、搜索过滤、临时透传     | 资源详情页 `/product/123`、固定层级     |



## 支付业务

![image-20250917143851571](C:\Users\PC\AppData\Roaming\Typora\typora-user-images\image-20250917143851571.png)

直接复制跳转地址



## 倒计时

安装day.js(在项目里面)

在composables里写一个通用逻辑

```
import { computed, ref } from "vue"
import dayjs from "dayjs"
export const useCountDown=()=>{
  const time=ref(0)
  const formatTime=computed(()=>dayjs.unix(time.value).format('mm分ss秒'))
  const start=(currentTime)=>{
    time.value=currentTime
    setInterval(()=>{
time.value--
    },1000)
  }
  return {
    formatTime,
    start
  }
}

```

之后在页面启动就好

```
const getPayInfo=async ()=>{
  const res=await getOrderAPI(route.query.id)
  payInfo.value=res.result
  start(res.result.countdown)
}
```

### 优化

```
import { computed, onUnmounted, ref } from "vue"
import dayjs from "dayjs"
export const useCountDown=()=>{
  let timer=null
  const time=ref(0)
  const formatTime=computed(()=>dayjs.unix(time.value).format('mm分ss秒'))
  const start=(currentTime)=>{
    time.value=currentTime
    setInterval(()=>{
time.value--
    },1000)
  }
  onUnmounted(()=>{
    timer&&clearInterval(timer)
  })
  return {
    formatTime,
    start
  }
}

```

关闭组件把倒计时销毁，等价于

```
if (timer) {
  clearInterval(timer);
}
```

