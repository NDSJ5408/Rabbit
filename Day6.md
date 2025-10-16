# Day6 登录模块

template <v-if>/<v-else>区分登录状态

### 跳转

```
 <li><a href="javascript:;" @click="$router.push('/login')">请先登录</a></li>
```





### 表单绑定

![image-20250915113757931](C:\Users\PC\AppData\Roaming\Typora\typora-user-images\image-20250915113757931.png)

#### 对象写法

```
const form=ref({
  account:'',
  password:''
})
const rules={
  account:[
    {required:true,message:'用户名不可以为空',trigger:'blur'}
  ],
  password:[
    {required:true,message:'密码不可以为空',trigger:'blur'},
    {min:6,max:14,message:'密码为6-14',trigger:'blur'}
  ]
}
```

#### 绑定

```
<el-form :model="form" :rules="rules">
              <el-form-item prop="account" label="账户">
                <el-input v-model="form.account"/>
              </el-form-item>
</el-form>
```

prop只是让item去form里面找到这个prop字段



### 自定义校验

![image-20250915144009335](C:\Users\PC\AppData\Roaming\Typora\typora-user-images\image-20250915144009335.png)

自己定义一个校验名

```
agree:[{
    validator:(rule,value,callback)=>{
       if(value==true){
        callback()
       }else{
        callback(new Error('请勾选协议'))
       }
    }
  }]
```





### 登录-表单整体校验

**validate**通过调用form实例的方法

```
const doLogin=()=>{
  formRef.value.validate((valid)=>{
     console.log(valid)
  })
}
```

**：valid 的值由 Element Plus 根据你的 rules 和当前 form 数据自动算出，你只需在回调里用 `if(valid)` 判断即可。**

```
<el-form ref="formRef" :model="form" :rules="rules" label-position="right" label-width="60px">
```

ref="formRef" 就是给 这一份 <el-form> 实例 起个“名字”，好让你在 <script setup> 里拿到它，从而调用 Element Plus 提供的各种方法（如 validate、resetFields、clearValidate 等）。



### 登录业务逻辑

#### 封装接口

```
export const loginAPI=({account,password})=>{
  return request({
    url:'/login',
    method:'POST',
    data:{
      account,
      password
    }
  })
}
```

这是一个post接口，要传入数据进去

**{account,password}是解构的对象**

#### 传数据给接口

```
const doLogin=()=>{
  const {account,password}=form.value
  formRef.value.validate(async (valid)=>{
     if(valid){
      const res=await loginAPI({account,password})
      console.log(res)
     }
  })
}
```

#### 提示用户

```
 ElMessage({type:'success',message:'登录成功'})
```

使用ElMessage组件，在官网上查询使用方法

#### 跳转

```
import { useRouter } from 'vue-router';
```

使用useRouter函数

```
const doLogin=()=>{
  const {account,password}=form.value
  formRef.value.validate(async (valid)=>{
     if(valid){
      const res=await loginAPI({account,password})
      console.log(res)
      ElMessage({type:'success',message:'登录成功'})
      router.replace({path:'/'})
     }
  })

```

**错误**

```
httpInstance.interceptors.response.use(res => res.data, e =>{
  // 统一错误提示
  ElMessage({type:"warning",message:e.response.data.message})
  return Promise.reject(e)
}
```

在拦截器写：**成功时自动剥掉外壳返回 data，失败时自动弹提示并保证错误继续向下传递**，业务组件里只写“正常流程”即可。



### PINIA管理用户数据

**Pinia 让你“把接口和数据操作收拢到一处”，组件只负责“用数据”和“触发动作”，彻底告别到处写重复 API 调用。**

#### 调取API

```
import { defineStore } from "pinia";
import { loginAPI } from '@/apis/user';
import { ref } from "vue";
export const useUserStore=defineStore('user',()=>{
const userInfo=ref({})
const getUserinfo=async({account,password})=>{
const res=await loginAPI({account,password})
userInfo.value=res.result
}
return{
  userInfo,
  getUserinfo
}
})

```

把API接口调用起来，userInfo是state，装数据

getUserinfo是方法，操作数据

#### 组件使用API

```
const doLogin=()=>{
  const {account,password}=form.value
  formRef.value.validate(async (valid)=>{
     if(valid){
      await userStore.getUserinfo({account,password})
      ElMessage({type:'success',message:'登录成功'})
      router.replace({path:'/'})
     }
  })
}
```

组件只用引入这个store，然后调用这个store就可以了，不用再写很多API繁琐的步骤了



### PINIA持久化

#### 为什么需要持久化

![image-20250915162758540](C:\Users\PC\AppData\Roaming\Typora\typora-user-images\image-20250915162758540.png)

按照官方文档，下载插件在main.js配置

因为源代码多创建了一个pinia

![image-20250916101954935](C:\Users\PC\AppData\Roaming\Typora\typora-user-images\image-20250916101954935.png)

### 登录和非登录适配

![image-20250915170122536](C:\Users\PC\AppData\Roaming\Typora\typora-user-images\image-20250915170122536.png)

采用token

```
 <template v-if="useStore.userInfo.token">
```





### 请求拦截器带token

#### 为什么

![image-20250915170357685](C:\Users\PC\AppData\Roaming\Typora\typora-user-images\image-20250915170357685.png)

有token才可以给数据

```
 config.headers.Authorization= `Bearer ${token}`
```



### 退出功能实现

在pinia新增一个清除用户数据，跳回用户页面

```
const clearUserInfo=()=>{
  userInfo.value={}
}
```

之后在组件使用事件触发使用

```
const confirm=()=>{
  useStore.clearUserInfo()
  router.push('/login')
}
```





### Token失效后再请求的处理

![image-20250915173813189](C:\Users\PC\AppData\Roaming\Typora\typora-user-images\image-20250915173813189.png)

```
httpInstance.interceptors.response.use(res => res.data, e =>{
  // 统一错误提示
  ElMessage({type:"warning",message:e.response.data.message})
  const useStore=useUserStore()
const router=useRouter()
   if(e.response.status===401){
    useStore.clearUserInfo()
  router.push('/login')
  }
  return Promise.reject(e)


})
```

