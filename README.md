## 实战项目: CNODE社区仿站
使用Vue-cli2搭建,包含webpack配置.
# 组件目录
src
├─App.vue
├─main.js
├─router
|   └index.js
├─components
|     ├─Article.vue
|     ├─Header.vue
|     ├─Pagination.vue
|     ├─PostList.vue
|     ├─slideBar.vue
|     └UserInfo.vue
├─assets
|   ├─cnodejs_light.svg
|   ├─loading.gif   

## 项目模块组件：
Header模块
PostList模块
Article模块
Slider侧边栏模块
UserInfo用户个人中心模块
Pagination分页组件的开发
### 自我分析
实际上述模块应在pages文件夹下拥有各自文件夹和index.vue文件.
components文件夹应该放入抽离的公共组件,比如头部,Pagination.
不过也应该只有首页,文章页和个人页面有独立页面,其他应该是抽离组件.
## 主要用到的技术栈有：
vue.js计算属性
vue.js的内置指令和事件的绑定
vue.js的自定义事件和触发
vue-router路由的跳转和监听
父子组件之间的数据传递

## 创建项目
使用Vue-cli2搭建.
```js
vue init webpack cnode
```
## 安装axios
```js
//在main.js中引入.
import Axios from 'axios'

//挂载到原型上
Vue.prototype.$http = Axios
```
## Header组件
创建components文件夹,创建`src/components/Header.vue`
```js
//src/components/Header.vue
<template>
    <div class="header">
        <router-link  :to="{name:'root'}">
        //点击logo跳转首页
        <img src="../assets/cnodejs_light.svg">
        </router-link>
        <ul>
            <li><a href="#">首页</a></li>
            <li><a href="#">新手教程</a></li>
            <li><a href="#">API</a></li>
            <li><a href="#">关于</a></li>
            <li><a href="#">注册</a></li>
            <li><a href="#">登录</a></li>
        </ul>    
    </div>
</template>  

<script>
export default {
    name: 'Header'
}
</script>
<style scoped>
.header {
    background-color: #5a5555;
    height: 50px;
}
img {
    max-width: 120px;
    margin-left: 50px;
    margin-top: 10px;
}
ul {
    list-style: none;
    float: right;
    margin: 4px;
}
li {
    display: inline-block;
    padding: 10px 15px;
}
a{
    text-decoration: none;
    color: #ccc;
    font-size: 14px;
    text-shadow: none;
}
</style>
```
## Postlist组件(首页文章列表)
### 需求分析
+ 加载时未获取数据加载loading动画
+ 列表获取头像,回复/浏览量,标题
+ 使用过滤器,时间,帖子分类,是否置顶,是否精华


```js
<template>
<div class="PostList">
    <!-- 在数据未返回时加载loading -->
    <div class="loading" v-if="isLoading">
        <img src="../assets/loading.gif">
    </div>

    <!-- 代表帖子列表 -->
    <div class="posts" v-else>
        <ul>
            <li>
                <div class="toobar">
                <span>全部</span>
                <span>精华</span>
                <span>分享</span>
                <span>问答</span>
                <span>招聘</span>
                </div>
            </li>
            <li v-for="post in posts">
                <!-- 头像 -->
                <img :src="post.author.avatar_url" alt="">
                <!-- 回复/浏览 -->
                <span class="allcount">
                    <span class="reply_count">{{post.reply_count}}</span>
                    /{{post.visit_count}}
                </span>
                <!-- 帖子分类,置顶 -->
                <span :class="[{put_good: (post.good == true), put_top: (post.top == true),
                'topiclist-tab': (post.good != true && post.top != true)}]">  
                    <span>
                        {{post | tabFormater}}
                    </span>
                </span>
                <!-- 标题 -->
                <router-link :to="{
                    name: 'post_content',
                    params:{
                        id: post.id,
                        name: post.author.loginname
                    }}">
                <span>
                    {{post.title}}
                </span>
                </router-link>
                <!-- 最终回复时间 -->
                <span class="last_reply">
                    {{post.last_reply_at | formatDate}}
                </span>
            </li>
            <li>
              <pagination @handleList="renderList"></pagination>
            </li>
        </ul>
    </div>   
</div>  
</template>

<script>
import pagination from './Pagination' 
export default {
    name: "PostList",
    data(){
        return{
            isLoading: false,
            posts: [],
            postpage: 1
        }
    },
    components: {
      pagination
    },
    methods: {
        getData(){
            this.$http.get('https://cnodejs.org/api/v1/topics',{
                params: {
                page: this.postpage,
                limit: 20
                }
            })
              .then(res=>{
                this.isLoading = false//加载成功去除动画
                this.posts = res.data.data 
              })
              .catch(err=>{
                  console.log(err)
              })
        },
        //点击页码按钮触发传入对应页码,跳转对应页面
        renderList(value){
          this.postpage = value
          this.getData() 
        }
    },
    beforeMount() {
        this.isLoading = true//加载成功之前加载动画
        this.getData()//在页面加载之前获取数据
    }
}
</script>
```
>注意点:
绑定class的属性中进行判断,使用的是括号,因为不是对象,只是要布尔值.

### 过滤器
#### 时间过滤器
因为response的时间是标准格式,需要转化.利用vue的过滤器.</br>
过滤器需要放在`main.js`中.
```js
Vue.filter('formatDate',function(str){
  if(!str) return ''
  let date = new Date(str)
  //当前时间减去发布时间得到时间间隔,即已发布了多久
  let time = new Date().getTime() - date.getTime()//现在的时间-传入的时间=相差的时间(单位-毫秒)
  if (time < 0){
    return ''
  }else if(time / 1000 < 30){
    return '刚刚'
  }else if (time / 1000 < 60){
    return parseInt((time / 1000)) + '秒前'
  }else if (time / 60000 < 60){
    return parseInt((time / 60000)) + '分钟前'
  }else if (time / 3600000 < 24){
    return parseInt((time / 3600000)) + '小时前'
  }else if (time / 86400000 < 31){
    return parseInt((time / 86400000)) + '天前'
  }else if (time / 2592000000 < 12){
    return parseInt((time / 2592000000)) + '月前'
  }else {
    return parseInt((time / 31536000000)) + '年前'
  }
})
```
#### 文章类型过滤器
```js
Vue.filter('tabFormater',function(post){
    if(post.good == true){
        return '精华'
    }else if(post.top == true){
        return '置顶'
    }else if(post.tab == 'ask'){
        return '问答'
    }else if(post.tab == 'share'){
        return '分享'
    }else{
        return '招聘'
    }
})
```

## Article页面
### 需求分析
+ 需要一个加载动画
+ 显示作者,发布时间,浏览量,来自
+ 文章内容使用markdown语法,需要解析.
+ 后面有回复,显示头像,用户名,都可以点击跳转
+ 最后的楼层数,点赞数

```js
<template>
    <div class="article">
        <!-- 正在加载显示loading -->
    <div class="loading" v-if="isLoading">
        <img src="../assets/loading.gif">
    </div>
        <div c v-else>
            <div class="topic_header"> 
                <div class="topic_title">{{post.title}}</div>
                <ul>
                    <li>*发布于: {{post.create_at | formatDate}}</li>
                    <li>*作者: {{post.author.loginname}}</li>
                    <li>* {{post.visit_count}}次浏览</li>
                    <li>*来自{{post | tabFormater}}</li>
                </ul>
                <div v-html="post.content" class="topic_content"></div>
            </div>
            <div id="reply">
                <div class="topbar">回复</div>
                <div v-for="(reply,index) in post.replies" class="replySec">
                  <div class="replyUp">
                    <router-link :to="{
                        name: 'user_info',
                        params: {
                            name: reply.author.loginname
                        }}">
                        <img :src="reply.author.avatar_url" alt="">
                    </router-link>
                    <router-link :to="{
                        name: 'user_info',
                        params:{
                            name:reply.author.loginname
                        }}">
                        <span>{{reply.author.loginname}}</span>
                    </router-link>
                    <span>{{index + 1}}楼</span>
                    <span v-if="reply.ups.length > 0" >☝{{reply.ups.length}}</span>
                    <span v-else></span>
                  </div>
                  <p v-html="reply.content"></p>
                </div>
            </div>
        </div>
    </div>
</template>
<script>
    export default{
        name: 'Article',
        data(){
            return {
                isLoading: false,
                post: []
            }
        },
        methods: {
            getArticleData(){
                this.$http.get(`https://cnodejs.org/api/v1/topic/${this.$route.params.id}`)
            }.then(res=>{
                if(res.data.success==true){
                    this.isLoading = false
                    this.post = res.data.data
                }
            }).catch(err=>console.log(err))
        },
    beforeMount(){
        this.isLoading = true
        this.getArticleData()
    },
    //检测到路由发生变化就重新获取数据
    watch:{
    '$route'(to,from){
        this.getArticleData()
    }
}

    }
</script>

```
## 创建路由
上面会进行跳转,需要设置路由.
```js
//router.js
import Vue from 'vue'
import Router from 'vue-router'
import PostList from '../components/PostList'
import UserInfo from '../components/UserInfo'

Vue.use(Router)

export default new Router{
    routes:[
        //首页路由
        {
            name: 'root',
            path: '/',
            components: {
                main: PostList
            }
        },
        //文章路由
        {
            name: 'post_content',
            path: 'topic/:id&author=:name',
            components: {
                main: Article
            }
        },
        //个人页面
        {
            name: 'user_info',
            path: '/userinfo/:name',
            components: {
                main: UserInfo
            }
        }
    ]
}
```
## userInfo个人页面
### 需求分析
+ 先添加一个loading动画,其实可以将loading动画封装成组件直接引入调用.
+ 分成三部分,上部分有头像,name,积分,注册时间
+ 中间是回复的主题
+ 下面是创建的主题


```js
<template>
    <div class="UserInfo">
    <!-- 在数据未返回时加载loading -->
    <div class="loading" v-if="isLoading">
        <img src="../assets/loading.gif">
    </div>
    <div class="userInformation" v-else>
        <section>
            <img :src="userinfo.avatar_url" >
            <span>{{userinfo.loginname}}</span>
            <p>{{userinfo.score}}积分</p>
            <p>注册时间: {{userinfo.create_at | formatDate}}</p>
        </section>
        <div class="replies">
            <p>回复的主题</p>
            <ul>
                <li v-for="item in userinfo.recent_replies">
                    <router-link :to="{
                        name: 'post_content',
                        params: {
                            id:item.id
                        }
                        }">
                        {{item.title}}
                    </router-link>
                </li>
            </ul>
        </div>
        <div class="topics">
            <p>创建的主题</p>
            <ul>
                <li v-for="item in userinfo.recent_topics">
                    <router-link :to="{
                        name: 'post_content',
                        params: {
                            id:item.id
                        }
                        }">
                        {{item.title}}
                    </router-link>
                </li>
            </ul>
        </div>
    </div>
    </div>
</template>

<script>
export default {
    name: 'UserInfo',
    data(){
        return {
            isLoading: false,
            userinfo: {}
        }
    },
    methods: {
        getData(){
            this.$http.get(`https://cnodejs.org/api/v1/user/${this.$route.params.name}`)
              .then(res=>{
                this.isLoading = false//加载成功去除动画
                this.userinfo = res.data.data
              })
              .catch(err=>{
                  console.log(err)
              })
        }
    },
    beforeMount(){
        this.isLoading = true;//加载成功之前显示加载动画
        this.getData();//在页面加载之前获取数据
      }
}
</script>
```
## Silderbar侧边栏
新建组件Silderbar.</br>
将组件引入到路由中,且合并到Article路由中.</br>
`App.vue`中记得添加`router-view`.
### 需求分析
+ 作者栏有头像,用户名,积分.
+ 分成三个部分,作者最近主题,作者最近回复.
+ 所有点击都可跳转,需加入路由.

```js
<template>
    <div class="autherinfo">
        <div class="authersummay">
            <div class="topbar">作者</div>
            <router-link :to="{
                name: 'user_info',
                params: {
                    name: userinfo.loginname
                }}">
             <img :src="userinfo.avatar_url" alt="">
            </router-link>
        </div>
        <div class="recent_topics">
            <div class="topbar">作者最近主题</div>
            <ul>
              <li v-for="list in topiclimitby5">
                <router-link :to="{
                  name: 'post_content',
                  params: {
                    id: list.id,
                    name: list.author.loginname
                  }
                }">
                  {{list.title}}
                </router-link>
              </li>
            </ul>
        </div>
        <div class="recent_replies">
            <div class="topbar">作者最近回复</div>
            <ul>
              <li v-for="list in replylimitby5">
                <router-link :to="{
                  name: 'post_content',
                  params: {
                    id: list.id,
                    name: list.author.loginname
                  }
                }">
                  {{list.title}}
                </router-link>
              </li>
            </ul>
        </div>
    </div>
</template>

<script>
export default {
    name: 'slidebar',
    data(){
        return{
            isLoading:false,
            userinfo: {}
        }
    },
    methods: {
        getData(){
            this.$http.get(`https://cnodejs.org/api/v1/user/${this.$route.params.name}`)
              .then(res=>{
                this.isLoading = false//加载成功去除动画
                this.userinfo = res.data.data
              })
              .catch(err=>{
                  console.log(err)
              })
        }
    },
    computed: {
        //筛选前五个主题
      topiclimitby5(){
        if(this.userinfo.recent_topics){
          return this.userinfo.recent_topics.slice(0,5)
        }
      },
       //筛选前五个主题
      replylimitby5(){
        if(this.userinfo.recent_replies){
          return this.userinfo.recent_replies.slice(0,5)
        }
      }
    },
    beforeMount(){
      this.isLoading = true;//加载成功之前显示加载动画
      this.getData();//在页面加载之前获取数据
    }

}
</script>
```

## Pagination分页组件
### 需求分析
+ 底部页面有首页,上一页,下一页,数字按钮
+ 点击按钮颜色反转,数字超过4后自动向前
+ 点击第5个时,出现省略符号.
### 实现
在postList中引入,并注册在components中.</br>
判断上下页后,记得return.
```js
 <tempalte>
    <div class="pagination">
        <button @click="changeBtn">首页</button>
        <button @click="changeBtn">上一页</button>
        <button v-if="judge" class="pagebtn">...</button>
        <button @click="changeBtn(btn)"
                v-for="btn in pagebtns"
                :class="[{currentPage: btn==currentPage},'pagebtn']"
        ></button>
        <button @click="changeBtn">下一页</button>
    </div>
 </tempalte>
 <script>
 export default{
     name: 'Pagination',
     data(){
         return {
             //初始页码
             pagebtns: [1,2,3,4,5,'...'],
             //当前页码
             currentPage:1,
             judge: true
         }
     },
     methods: {
         //切换页码
         changeBtn(page){
             //点击上一页,下一页,首页
             if(typeof page != 'number'){
                 switch(page.target.innerText){
                     case '上一页':
                        $('button.currentPage').prev().click()
                        break;
                     case '下一页':
                        $('button.currentPage').next().click()
                        break;
                     case '首页':
                        this.pagebtns =[1,2,3,4,5,'...']
                        this.changeBtn(1)
                        break;
                     default:
                        break;
                 }
                 //最后将这个判断return出去
                 return
             }
             //判断前面的省略号是否出现
             this.currentPage = page
             if(page>4){
                 this.judge = true
             }else{
                 this.judge = false
             }

             //点击数字会前进后退
            if(page = this.pagebtns[4]){
                //移除第一个
                this.pagebtns.shift()
                //添加最后一个,因为移除之后剩4个所以是pagebtns[3]
                this.pagebtns.splice(4,0,this.pagebtns[3]+1)
                //如果选中是第一个元素,且不等于1
            }else if(page == this.pagebtn[0] && page != 1){
                //第一个位置加一个
                this.pagebtns.unshift(this.pagebtns[0]-1)
                //移除最后一个
                this.pagebtns.splice(5,1)
            }
            //触发点击按钮即切换页面的事件
            this.$emit('handleList',this.currentPage)
     }

 }
 </script>
 ```
 触发的事件传递给父组件postList.
 ```js
 //postlIst.vue
 <pagination @handleList="renderList"></pagination>
 ```
 ## 路由跳转
 路由在app.vue中显示
 ```js
 //app.vue
 <template>
 <div id="app">
    <Header></Header>
    <div class="main">
        <router-view name="slidebar"></router-view>
        <router-view name="main"></router-view>
    </div>
 </div>
 </template>
 <script>
    import Header form "./components/Header"
    import PostList form "./components/PostList"
    export default{
        name: "App",
        compponents: {
            Header,PostList
        }
    }
 </script>
