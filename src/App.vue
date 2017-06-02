<template>
  <section>
    <article class="article">
      <inputbind :inputDatas="item" v-for="(item, index) in inputTextDatas" :key="index" @input="updateInput"></inputbind>
    </article>
    <footer class="footer">
      <inputbutton :inputDatas="inputbuttonDatas" :isDisabled="disabled" @click.submit.stop.prevent.native="submit"></inputbutton>
    </footer>
  </section>
</template>

<script>
import Loading from '@/components/base/loading'
import Success from '@/components/base/success'
import Api from '@/core/service/service'
import Inputbind from '@/components/base/inputbind'
import Inputbutton from '@/components/base/inputbutton'

export default {
  name: 'app',
  components: {
    Inputbind,
    Inputbutton
  },
  data () {
    return {
      loadingTimer: null,
      otherReason: '',
      disabled: true,
      inputTextDatas: [{
        type: 'text',
        text: '邮箱账号',
        placeholder: '请输入企业邮箱账号',
        name: 'account'
      }, {
        type: 'password',
        text: '密  码',
        placeholder: '请输入密码',
        name: 'password'
      }],
      inputbuttonDatas: {
        text: '绑定',
        type: 'submit'
      },
      values: {
        account: '',
        password: ''
      }
    }
  },
  methods: {
    showLoading (useJsapi = true, showText = '加载中...') {
      if (this.loadingTimer) return
      this.loadingTimer = setTimeout(() => {
        if (this.isJsapiReady && useJsapi) {
          FSOpen.widget.showPreloader({
            text: showText
          })
        } else {
          Loading.open(showText)
        }
      }, window.$showLoadingDelay)
    },
    closeLoading (useJsapi = true) {
      if (!this.loadingTimer) return
      clearTimeout(this.loadingTimer)
      this.loadingTimer = null
      if (this.isJsapiReady && useJsapi) {
        FSOpen.widget.hidePreloader()
      } else {
        Loading.close()
      }
    },
    scrollBottom () {
      let h = document.documentElement.scrollHeight - document.documentElement.clientHeight
      if (h > 0) document.body.scrollTop = h
    },
    submit ($event) {
      this.validate(() => {
        this.showLoading(false, '提交中...')
        Api.bind({
          data: this.values,
          always: () => { this.closeLoading(false) }
        }).then(res => {
          if (res.errorCode === 0) {
            Success.toast({
              duration: 1000,
              text: '提交成功'
            })
          } else {
            Success.toast({
              duration: 1000,
              text: res.errorMessage
            })
          }
        })
      })
    },
    // 接收子组件通过事件传递过来的实时input值
    updateInput (value, name) {
      this.values[name] = value
      if (this.values['account'] || this.values['password']) {
        this.disabled = false
      } else {
        this.disabled = true
      }
    },
    validate (cb) {
      let account = this.values['account']
      if (!account) {
        Success.toast({
          duration: 1000,
          text: '邮箱账号不能为空'
        })
        return
      }
      if (!this.values['password']) {
        Success.toast({
          duration: 1000,
          text: '密码不能为空'
        })
        return
      }
      if (account && !/^([a-zA-Z0-9_.-])+@(([a-zA-Z0-9-])+\.)+([a-zA-Z0-9]{2,4})+$/.test(account)) {
        Success.toast({
          duration: 1000,
          text: '邮箱帐号格式错误'
        })
        return
      }
      cb && cb()
    }
  }
}
</script>

<style lang="less">
@import '~@/assets/style/all';
.article{
  margin-top: 15/25rem;
}
.footer{
  margin-top: 15/25rem;
}
</style>
