<template>
  <section>
    <article class="article">
      <inputbind :inputDatas="item" v-for="(item, index) in inputTextDatas" :key="index" @input="updateInput"></inputbind>
    </article>
    <template v-if="isManual">
      <article class="article article-imap">
        <h4>{{receiverDatas.title}}</h4>
        <inputbind :inputDatas="item" v-for="(item, index) in receiverDatas.datas" :key="index" @input="updateInput"></inputbind>
      </article>
      <article class="article article-imap">
        <h4>{{sendDatas.title}}</h4>
        <inputbind :inputDatas="item" v-for="(item, index) in sendDatas.datas" :key="index" @input="updateInput"></inputbind>
      </article>
    </template>
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
      isManual: false, // 是否为复杂绑定
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
      receiverDatas: {
        title: '收信',
        datas: [{
          type: 'text',
          text: '服务器地址',
          placeholder: 'imap.example.com',
          name: 'receiveHost'
        }, {
          type: 'text',
          text: '端口',
          placeholder: '',
          name: 'receivePort'
        }, {
          type: 'checkbox',
          text: 'SSL',
          placeholder: '',
          name: 'isReceiveSsl'
        }]
      },
      sendDatas: {
        title: '发信',
        datas: [{
          type: 'text',
          text: '服务器地址',
          placeholder: 'imap.example.com',
          name: 'sendHost'
        }, {
          type: 'text',
          text: '端口',
          placeholder: '',
          name: 'sendPort'
        }, {
          type: 'checkbox',
          text: 'SSL',
          placeholder: '',
          name: 'isSendSsl'
        }]
      },
      requests: {},
      errorTip: {
        account: '邮箱账号不能为空',
        password: '密码不能为空',
        receiveHost: '收件服务器地址不能为空',
        receivePort: '收件服务器端口不能为空',
        isReceiveSsl: '密码不能为空',
        sendHost: '发件服务器地址不能为空',
        sendPort: '发件服务器端口不能为空'
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
      if (!this.isManual) {
        this.bind()
      } else {
        this.manualBind()
      }
    },
    bind () { // 简单绑定
      this.validate((requestParams) => {
        this.showLoading(false, '验证中...')
        Api.bind({
          data: requestParams,
          always: () => { this.closeLoading(false) }
        }).then(res => {
          if (res.errorCode === 0) {
            let data = res.data
            if (data === 1) { // 绑定成功
              Success.toast({
                duration: 1000,
                text: '绑定成功'
              })
            } else if (data === 2) { // 继续验证：需要提供邮箱补充信息
              this.isManual = true
              this.requests.isReceiveSsl = false
              this.requests.isSendSsl = false
            } else {
              if (data === -1) {
                Success.toast({
                  duration: 1000,
                  text: '验证失败'
                })
              } else {
                Success.toast({
                  duration: 1000,
                  text: '已有一个邮箱绑定'
                })
              }
            }
          } else {
            Success.toast({
              duration: 1000,
              text: res.errorMessage
            })
          }
        })
      })
    },
    manualBind () { // 复杂绑定
      this.validate((requestParams) => {
        this.showLoading(false, '验证中...')
        Api.manualBind({
          data: requestParams,
          always: () => { this.closeLoading(false) }
        }).then(res => {
          if (res.errorCode === 0) {
            let data = res.data && res.data
            if (data === 1) { // 绑定成功
              Success.toast({
                duration: 1000,
                text: '绑定成功'
              })
            } else { // 绑定失败
              if (data === -1) {
                Success.toast({
                  duration: 1000,
                  text: '验证失败'
                })
              } else {
                Success.toast({
                  duration: 1000,
                  text: '已有一个邮箱绑定'
                })
              }
            }
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
      this.requests[name] = value
      if (!this.isManual) { // 简单绑定
        if (this.requests['account'] || this.requests['password']) {
          this.disabled = false
        } else {
          this.disabled = true
        }
      } else { // 复杂绑定
        this.disabled = true
        for (var key in this.requests) {
          if (this.requests[key]) {
            this.disabled = false
          }
        }
      }
    },
    validate (cb) {
      let requestPrams = this.requestPrams()
      let flag = true
      if (requestPrams['account'] && !/^([a-zA-Z0-9_.-])+@(([a-zA-Z0-9-])+\.)+([a-zA-Z0-9]{2,4})+$/.test(requestPrams['account'])) {
        Success.toast({
          duration: 2000,
          text: '邮箱帐号格式错误'
        })
        return
      }
      if (requestPrams['receivePort'] && !requestPrams['receivePort'].match(/\d+/)) {
        Success.toast({
          duration: 2000,
          text: '收信服务器端口必须为数字'
        })
        return
      }
      if (requestPrams['sendPort'] && !requestPrams['sendPort'].match(/\d+/)) {
        Success.toast({
          duration: 2000,
          text: '发信服务器端口必须为数字'
        })
        return
      }
      for (var key in requestPrams) {
        if (!requestPrams[key] && requestPrams[key] !== 0) {
          Success.toast({
            duration: 2000,
            text: this.errorTip[key]
          })
          flag = false
          break
        }
      }
      if (!flag) { return }
      cb && cb(requestPrams)
    },
    requestPrams () {
      let result = {}
      let requests = this.requests
      if (!this.isManual) { // 简单绑定
        result = {
          account: requests.account || '', // 邮箱账号
          password: requests.password || '' // 邮箱密码
        }
      } else { // 复杂绑定
        result = Object.assign({}, {
          account: '',
          password: '',
          emailType: 2, // 收件服务器类型 1:pop 2:imap
          receiveHost: '', // 收件服务器地址
          receivePort: '', // 收件服务器端口
          isReceiveSsl: 0, // 是否启用SSL收信 0:表示不使用 1：表示使用
          sendHost: '',  // SMTP发件服务器地址
          sendPort: '',  // SMTP发件服务器端口
          isSendSsl: 0  // 是否启用SSL发信 0:表示不使用 1：表示使用
        }, requests)
      }
      return result
    }
  }
}
</script>

<style lang="less">
@import '~@/assets/style/all';
.article{
  margin-top: 15/25rem;
  h4{
    padding: 0 15/25rem ;
    font-size:14/25rem;
    color:#999999;
    letter-spacing:0px;
    line-height:14/25rem;
    text-align:left;
    margin-bottom: 15/25rem;
  }
}
.article-imap{
  .input-left{
    flex: 0 0 88/25rem;
  }
}
.footer{
  margin-top: 15/25rem;
}
</style>
