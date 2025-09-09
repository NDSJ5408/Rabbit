# Day3

## scoped

让样式只作用于当前组件，不会污染全局或其他组件。

## 函数封装

```
import axios from "axios";

const httpInstance=axios.create({
  baseURL:'http://pcapi-xiaotuxian-front-devtest.itheima.net ',
  timeout:5000
})

// axios请求拦截器
httpInstance.interceptors.request.use(config => {
  return config
}, e=>Promise.reject(e))

// axios响应式拦截器
httpInstance.interceptors.response.use(res => res.data, e =>{
  // 统一错误提示

  return Promise.reject(e)
})


export default httpInstance
```

Axios创建了一个实例httpInstance，默认基地址为'http://pcapi-xiaotuxian-front-devtest.itheima.net '

出发前（请求拦截器）：必须来我这报备一下，我可以给你加工一下包裹。

回来后（响应拦截器）：必须先把包裹拆好，处理好任何问题，再把最核心的东西交给主人。

最后 export default httpInstance 就是把这个定制好的快递员分享给项目里的其他成员使用。



- **`res`**：是**服务器给你的完整回复包裹**，你用 `res.data` 来拆开包裹拿里面最重要的数据。
- **`Promise`**：是一个**代表未来可能完成的操作的“承诺书”**。它让JavaScript可以在等待网络请求这种“慢操作”的同时，不去阻塞其他任务的执行。
  1. **pending**：承诺中（餐正在做/快递在路上）
  2. **fulfilled**：承诺已兑现（成功拿到餐/收到快递）
  3. **rejected**：承诺被拒绝（餐厅说没食材了/快递丢件了）

```
：import httpInstance from "@/utils/http";

export function getCategoryAPI(){
return httpInstance({
  url:'/home/category/head'
})
}
```

这是定义了一个函数：

- 对自己公司的专属快递员(`httpInstance`)说：“嘿，你去一趟总部的仓库(`baseURL`)，找到 `/home/category/head` 这个货架，把上面的东西给我取回来。”

## vue入口

Vue 的入口永远是 `main.js`（或 `main.ts`）里挂载的那个根组件——默认就是 `App.vue`。
你看到的“Layout 页面”其实只是 `App.vue` 里的 `<RouterView>` 把 `Layout` 组件渲染进去了而已。

## 把多个盒子平铺方法

只需要把父容器改成display：flex即可

## gap

![image-20250909085148993](C:\Users\PC\AppData\Roaming\Typora\typora-user-images\image-20250909085148993.png)

## 吸顶导航useScroll

useScroll(window)监听窗口滑动条的位置

```
:class="{ show: y > 78 }"
```

当滚动超过 78px 时，自动给元素加上 `show` 类，用来做**吸顶导航**的显隐/动画。

## 解构

**import  {httpInstance} from '@/utils/http'和import  httpInstance from '@/utils/http'什么区别**？

如果它是用 export default httpInstance 导出的，你就必须用 import httpInstance from '@/utils/http'。这里导出来的名字可以随便起，（比如叫httpInstance, myHttp, axiosInstance都可以）。

如果它是用 export const httpInstance = ... 导出的，你就必须用 import { httpInstance } from '@/utils/http'。你接收的时候，必须用**花括号 `{}`** 来精确指定你要哪个名字的盒子。**你不能随便给它改名**，因为它是按名字找的。

## props

- **定义**：**父组件向子组件传递数据的一种方式**。
- **重要特性**：**单向数据流** (One-Way Data Flow)
  - 数据只能从**父组件向下流动**到子组件。
  - 子组件**不能直接修改**父组件传过来的 props。如果试图修改，Vue 会在控制台发出警告。

### 声明props

**数组形式**

```
export default {
  props: ['title', 'content', 'isPublished']
}
```

**对象形式**

```
defineProps({
  title:{
    type:String
  },
  subTitle:{
    type:String，
    required: true，
    default: 1
  }
})
```

### 如何使用

**父组件传值子组件**

在父组件中，通过在子组件的标签上使用**属性绑定**（`v-bind:` 或 `:`）来传递数据；或者静态传值也可以

1.在父组件内引入子组件

```
import ChildComponent from './ChildComponent.vue'
```

2.准备要传递的数据

```
const parentUserName = ref('小明') // 使用ref定义响应式数据
const parentUserAge = ref(25)
```

3.在子组件上绑定

```
<ChildComponent :user-name="parentUserName" :user-age="parentUserAge"/>
```

4.子组件声明并且接受

```
<script setup>
// 1. 使用 defineProps 声明接收的属性
// 推荐使用对象形式，可以进行类型验证和设置默认值
const props = defineProps({
  userName: {
    type: String,
    required: true // 必传
  },
  userAge: {
    type: Number,
    default: 18 // 可选，默认值18
  }}）
  <script/>
```

5.子组件使用数据

**在模板中**：直接使用属性名（如 `{{ userName }}`）

**在 `<script setup>` 中**：通过 `props.属性名` 访问（如 `props.userName`）

```
<!-- ChildComponent.vue 的补充 -->
<script setup>
// 假设我们需要一个计算属性基于 props 计算
import { computed } from 'vue'

const isAdult = computed(() => {
  return props.userAge >= 18 // 基于 props 计算
})

const nextYearAge = computed(() => {
  return props.userAge + 1 // 基于 props 计算
})
</script>
```

**子组件传父组件**

1.子组件在script里面定义emits

```
const emit = defineEmits(['update-name'])
```

update-name是事件名

2.定义触发事件的方法

```
const requestChangeName = () => {
  const newName = '小花' // 子组件生成的新值
  emit('update-name', newName) // 触发事件，并传递新值
}
```

newName是要传入的值

3.父组件要监听事件

```
 <ChildComponent :user-name="parentUserName" 
    @update-name="handleUpdateName" <!-- 监听子组件事件 -->
  />
```

4.知道是事件后，在父组件定义方法改数据

```
const handleUpdateName = (newName) => {
  parentUserName.value = newName // 父组件自己修改自己的数据！
  console.log('用户名已更新为:', newName)
}
```

1. **父组件准备数据**：使用 `ref` 或 `reactive` 定义响应式数据。
2. **父组件传递数据**：在子组件标签上使用 `:属性名="数据"` 传递。
3. **子组件声明接收**：使用 `defineProps({...})` 声明接收哪些属性及其类型。
4. **子组件使用数据**：在模板中直接使用，在 script 中通过 `props.xxx` 使用。
5. **子组件想修改**：使用 `defineEmits` 定义事件，用 `emit('事件名', 值)` 触发。
6. **父组件响应修改**：监听子组件事件 `@事件名="处理函数"`，在处理函数中修改自己的数据。

**谁的数据谁来改**
