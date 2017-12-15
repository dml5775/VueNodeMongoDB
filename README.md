# VueNodeMongoDB
Vue+Node+MongoDB从开发到正式部署
一：开发部署
1. 前后端分离，各自开发。这里前后端分离是这样的：前端用Vue开发静态页面，路由通过Vue-Router进行，后端用Node仅用于编写API给前端调用获取数据。 
2. 前端开发时通过Vue-Cli中提供的proxyTable进行代理，由此可跨域调用Node编写的API。 
3. 前后端各自开发完成，测试无误后，前端通过webpack打包压缩，后端拉取前端打包压缩好的文件即部署完成。
二：以登录功能为例跑一遍这个过程
这里会以一个不完善的登录功能来演示这个过程。 
功能很简单，填写完账号密码点击登录按钮，就把账号密码插入数据库，并且查出现在已经有哪些数据。

用到的东西：

Vue-Cli
Vue-Resource
Node + Express
MongoDB
下面正式开始：”

安装Vue-Cli 
npm i -g vue-cli

去到你想要放置项目的目录，进行初始化 
vue init webpack XXX（你想要项目的名字）

按提示一直下一步，完成后按提示 
cd XXX 
npm install （这里建议使用淘宝镜像cnpm，不然墙外的东西……） 
npm run dev 
如无意外你就会看到下图，证明成功了 

4、这时我们就可以去开发我们的登录页面了。开发过程就不说了，样式怎样也无关紧要，这里只需要两个input框用于填写账号跟密码，还有一个登录按钮负责请求后端接口而已。记得安装一下Vue-Resource。核心代码：
<template>
<div>
  <input class="form-control" id="inputEmail3" placeholder="请输入账号" v-model="account">
<input type="password" class="form-control" id="inputPassword3" placeholder="请输入密码" v-model="password">
<button type="submit" class="btn btn-default" @click="login">登录</button>
  </div>
</template>

<script>
    export default {
    data() {
        return {
            account : '',
            password : ''
        }
    },
    methods:{
      login() {
        // 获取已有账号密码
        this.$http.get('/api/login/getAccount')
          .then((response) => {
            // 响应成功回调
            console.log(response)
            let params = { 
              account : this.account,
              password : this.password
            };
            // 创建一个账号密码
            return this.$http.post('/api/login/createAccount',params);
          })
          .then((response) => {
            console.log(response)
          })
          .catch((reject) => {
            console.log(reject)
          });
        }
      }
    }
</script>
这时回到浏览器，如无意外应该会出现两个输入框和一个登录按钮，当然现在去点击登录按钮请求接口，是不行的。所以我们现在需要去搭建Node。 
- 搭建Node 
这里默认大家已经安装好Node，不然上面那些都没办法安装的…… 
在项目的根目录新建一个叫server的目录，用于放置Node的东西。进入server目录，再新建三个js文件： 
- index.js （入口文件） 
- db.js （设置数据库相关） 
- api.js （编写接口） 
好，现在我们来搞Node。这里使用的Node框架是Express。安装Express： 
npm install express –save
index.js：

// 引入编写好的api
const api = require('./api'); 
// 引入文件模块
const fs = require('fs');
// 引入处理路径的模块
const path = require('path');
// 引入处理post数据的模块
const bodyParser = require('body-parser')
// 引入Express
const express = require('express');
const app = express();

app.use(bodyParser.json());
app.use(bodyParser.urlencoded({extended: false}));
app.use(api);
// 访问静态资源文件 这里是访问所有dist目录下的静态资源文件
app.use(express.static(path.resolve(__dirname, '../dist')))
// 因为是单页应用 所有请求都走/dist/index.html
app.get('*', function(req, res) {
    const html = fs.readFileSync(path.resolve(__dirname, '../dist/index.html'), 'utf-8')
    res.send(html)
})
// 监听8088端口
app.listen(8088);
console.log('success listen…………');
db.js： 
这个js文件主要用于配置mongoDB相关东西，mongoDB的相关概念大家自行搜索学习。我们这里使用mongoose 中间件来连接mongoDB。mongoDB的安装大家百度即可。有一点需要注意的是，mongoDB安装完之后，我们需要自己手动去新建一个文件夹用于放置mongoDB的数据。我的mongoDB安装在E盘，所以也在E盘根目录创建一个叫mongo-data的文件夹（名字随便取），进入该文件夹，再新建一个名叫db的文件夹。当我们去启动mongoDB服务时，我们需要指定该路径。如何开启mongoDB服务？进入mongoDB所在目录，再进入它的bin目录，敲这条命令： 
mongod.exe –dbpath e:\mongo-data\db 
如无意外mongoDB就会顺利启动。而关于怎样使用mongoose 中间件去操作mongoDB，建议大家移步到mongoose 的官网，照着quick start敲一遍就可以了。

// Schema、Model、Entity或者Documents的关系请牢记，Schema生成Model，Model创造Entity，Model和Entity都可对数据库操作造成影响，但Model比Entity更具操作性。
const mongoose = require('mongoose');
// 连接数据库 如果不自己创建 默认test数据库会自动生成
mongoose.connect('mongodb://localhost/test');

// 为这次连接绑定事件
const db = mongoose.connection;
db.once('error',() => console.log('Mongo connection error'));
db.once('open',() => console.log('Mongo connection successed'));
/************** 定义模式loginSchema **************/
const loginSchema = mongoose.Schema({
    account : String,
    password : String
});

/************** 定义模型Model **************/
const Models = {
    Login : mongoose.model('Login',loginSchema)
}

module.exports = Models;

api.js：

// 可能是我的node版本问题，不用严格模式使用ES6语法会报错
"use strict";
const models = require('./db');
const express = require('express');
const router = express.Router();

/************** 创建(create) 读取(get) 更新(update) 删除(delete) **************/

// 创建账号接口
router.post('/api/login/createAccount',(req,res) => {
    // 这里的req.body能够使用就在index.js中引入了const bodyParser = require('body-parser')
    let newAccount = new models.Login({
        account : req.body.account,
        password : req.body.password
    });
    // 保存数据newAccount数据进mongoDB
    newAccount.save((err,data) => {
        if (err) {
            res.send(err);
        } else {
            res.send('createAccount successed');
        }
    });
});
// 获取已有账号接口
router.get('/api/login/getAccount',(req,res) => {
    // 通过模型去查找数据库
    models.Login.find((err,data) => {
        if (err) {
            res.send(err);
        } else {
            res.send(data);
        }
    });
});

module.exports = router;

至此我们的后端代码就编写好了，进入server目录，敲上 node index命令，node就会跑起来，这时在浏览器输入http://localhost:8088/api/login/getAccount就能访问到这个接口了 
5. 回到前端，尝试请求接口 
现在我们点击登录按钮去请求接口，当然还是不行的，因为使用npm run dev 进行开发时，其实webpack会启动一个8080的web服务用于我们进行开发，而我们后端是在8088端口的，所以我们肯定请求不到后端的接口。怎么办？跨域问题相信前端都很熟悉，难道我们又要去搞一遍解决这些问题？我们只是想开发的时候能调到后端的接口而已…… 
贴心的脚手架其实已经为我们解决了这个问题。进入 
这里写图片描述 
我们会发现有有一个proxyTable，这就是用来开启一个代理服务从而解决我们这个问题的：Vue-Cli官方描述。 
于是，我们写成这样：

 proxyTable: {
        '/api': {
        target: 'http://localhost:8088/api/',
        changeOrigin: true,
        pathRewrite: {
          '^/api': ''
        }
      }
    }
这时，我们在前端接口地址前加上/api，就会指向http://localhost:8088/api/，于是我们就能访问到后端的接口了！让我们来点击一下登录按钮，会发现接口请求成功了！再去数据库看看！也插入了一条新数据！成功！ 
6. 前后端开发完成，最后一步，前端打包，后端部署。 
前端打包就很简单了，一个命令： 
npm run build 这就生成了一个dist目录，里面就是打包出来的东西。 
现在回过头来看server里面的入口文件index.js

// 访问静态资源文件 这里是访问所有dist目录下的静态资源文件
app.use(express.static(path.resolve(__dirname, '../dist')))
// 因为是单页应用 所有请求都走/dist/index.html
app.get('*', function(req, res) {
    const html = fs.readFileSync(path.resolve(__dirname, '../dist/index.html'), 'utf-8')
    res.send(html)
})
// 监听8088端口
app.listen(8088);
这里的关键是express.static，利用 Express 托管静态文件。于是我们才能访问到前端打包出来的静态页面index.html。 
最后，我们在浏览器输入http://localhost:8088/，就会跳到index.html。

到此为止，我们就完成了整个前后端各自开发到正式部署的流程。
