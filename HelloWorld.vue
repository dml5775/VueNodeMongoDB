<template>
//template里只会显示第一个元素，需要在最外面加上一个div包括里面，不然只会显示一个输入账号input框
  <div>
    <input class="form-control" id="inputEmail3" placeholder="请输入账号" v-model="account">
    <input type="password" class="form-control" id="inputPassword3" placeholder="请输入密码" v-model="password">
    <button type="submit" class="btn btn-default" @click="login">登录</button>
    </div>
</template>
<script>
//用vue手脚架生成的项目修.vue文件后，保存编译报错，缺少2个空格
//分析：
//这样是因为初始化项目的时候Use ESLint to lint your code? (Y/n) 这一步选了y，设置了eslint，
//如果不想有规范的js代码，可以重新初始化关掉eslint。Use ESLint to lint your code? (Y/n) 这一步选no
//如果想有良好的规范，其实错误已经很清晰，大多数就是缩进不规范，分号不需要等原因，很容易解决的。写多了就成习惯了。
    export default {
      data () {
        return {
          account: '',
          password: ''
        }
      },
      methods: {
        login () {
          // 获取已有账号密码
          this.$http.get('http://localhost:8088/api/login/getAccount', {account: this.account, password: this.password})
            .then((response) => {
              // 响应成功回调
              console.log(response)
              let params = {
                account: this.account,
                password: this.password
              }
              // 创建一个账号密码
              return this.$http.post('http://localhost:8088/api/login/createAccount', params)
            })
            .then((response) => {
              console.log(response)
            })
            .catch((reject) => {
              console.log(reject)
            })
        }
      }
}
</script>
