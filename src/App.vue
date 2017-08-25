<template>
  <section>
    <template v-if="isBindPage && !isBind">
      <header v-if="isManual"  class="email-bind-header">
        <ul @click="tapImapOrPop">
          <li :class="{active:isImap}">IMAP</li>
          <li :class="{active:isPop}">POP</li>
        </ul>
      </header>
      <article class="article">
        <inputbind :inputDatas="item" v-for="item in inputTextDatas" :val="item.val && item.val" :key="item.name" @input="updateInput"></inputbind>
      </article>
      <template v-if="isManual">
        <article class="article article-istitle">
          <h4>{{receiverDatas.title}}</h4>
          <inputbind :inputDatas="item" v-for="item in receiverDatas.datas" :val="item.val && item.val" :key="receiverDatas.id" @input="updateInput"></inputbind>
        </article>
        <article class="article article-istitle">
          <h4>{{sendDatas.title}}</h4>
          <inputbind :inputDatas="item" v-for="item in sendDatas.datas" :val="item.val && item.val" :key="item.name" @input="updateInput"></inputbind>
        </article>
      </template>
      <footer class="footer">
        <inputbutton :inputDatas="inputbuttonBind" :isDisabled="disabled" @click.submit.stop.prevent.native="submit"></inputbutton>
      </footer>
    </template>
    <template v-if="isBindStart && !isBind">
      <carousel></carousel>
      <footer class="footer-carousel">
        <inputbutton :inputDatas="inputbuttonCarousel" @click.submit.stop.prevent.native="jumpBindPage"></inputbutton>
      </footer>
    </template>
    <template v-if="isBind">
      <article class="article article-istitle">
          <h4>{{emailDatas.title}}</h4>
          <inputbind :inputDatas="item" v-for="item in emailDatas.datas" :key="item.name" @input="updateInput"></inputbind>
      </article>
      <footer class="footer">
        <inputbutton :inputDatas="inputbuttonSend" :isDisabled="disabled" @click.submit.stop.prevent.native="send"></inputbutton>
      </footer>
    </template>
  </section>
</template>
<script>
import Loading from '@/components/base/loading'
import Success from '@/components/base/success'
import Api from '@/core/service/service'
import Inputbind from '@/components/base/inputbind'
import Inputbutton from '@/components/base/inputbutton'
import Carousel from '@/components/base/carousel'
import Jsapi from '@/core/base/jsapi'
import Util from '@/core/base/util'

export default {
  name: 'app',
  components: {
    Inputbind,
    Inputbutton,
    Carousel
  },
  data () {
    return {
      openBindStart: true, // 是否开启绑定页的开场动画
      isBind: false, // 判断是否绑定
      isBindPage: false, // 绑定是否显示绑定页
      isBindStart: false, // 绑定开场图片页
      isJsapiReady: false, // 判断jsapi是否被初始化
      isImap: true, // 判断是否是IMAP绑定
      loadingTimer: null,
      otherReason: '',
      disabled: true,
      isManual: false, // 是否为复杂绑定
      sendEmail: '', // 发件人
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
      inputbuttonBind: {
        text: '绑定',
        type: 'submit'
      },
      inputbuttonSend: {
        text: '发送',
        type: 'submit'
      },
      inputbuttonCarousel: {
        text: '绑定我的邮箱',
        type: 'button'
      },
      receiverDatas: {
        title: '收信',
        datas: [{
          type: 'text',
          text: '服务器地址',
          placeholder: 'imap.example.com',
          val: '',
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
          val: '',
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
        receivePort: '收件端口不能为空',
        sendHost: '发件服务器地址不能为空',
        sendPort: '发件端口不能为空'
      }
    }
  },
  computed: {
    emailDatas () {
      return {
        title: '收件人会收到付款订单邮件',
        datas: [{
          type: 'noinput',
          text: ' 发件人',
          placeholder: this.sendEmail,
          name: 'sendEmail'
        }, {
          type: 'text',
          text: '收件人',
          placeholder: '请输入收件人邮箱',
          name: 'toList'
        }]
      }
    },
    isPop () {
      return !this.isImap
    }
  },
  created () {
    Jsapi().then(() => {
      this.isJsapiReady = true
      this.getBindMsg(() => {
        this.setTitle()
      })
    }).catch(() => {
      this.getBindMsg(() => {
        this.setTitle()
      })
    })
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
    sendSuccess () { // 发送成功回调方法
      if (!Util.device.pc && this.isJsapiReady) {
        // 这里执行jsapi接口调用操作，关闭当前页
        FSOpen.webview.close({
          extras: {
            data: 1
          }
        })
      } else { // pc
        history.back()
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
              this.getBindMsg(() => {
                this.setTitle()
              })
            } else if (data === 2) { // 继续验证：需要提供邮箱补充信息
              this.jumpManualBindPage()
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
              duration: 4000,
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
              this.getBindMsg(() => {
                this.setTitle()
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
              duration: 4000,
              text: res.errorMessage
            })
          }
        })
      })
    },
    // 接收子组件通过事件传递过来的实时input值
    updateInput (value, name) {
      this.requests[name] = value
      if (!this.isManual && !this.isBind) { // 简单绑定
        if (this.requests['account'] || this.requests['password']) {
          this.disabled = false
        } else {
          this.disabled = true
        }
      } else { // 复杂绑定或发送页
        this.disabled = true
        if (this.isManual) {
          for (var key in this.requests) {
            if (this.requests[key]) {
              this.disabled = false
            }
          }
        }
        if (this.isBind) {
          if (this.requests['toList']) {
            this.disabled = false
          }
        }
      }
    },
    validate (cb) { // 验证简单绑定和复杂绑定
      let requestPrams = this.requestPrams()
      if (requestPrams['account'] && !/^([a-zA-Z0-9_.-])+@(([a-zA-Z0-9-])+\.)+([a-zA-Z0-9]{2,4})+$/.test(requestPrams['account'])) {
        Success.toast({
          text: '邮箱帐号格式错误'
        })
        return
      }
      if (requestPrams['receivePort'] && !requestPrams['receivePort'].match(/\d+/)) {
        Success.toast({
          text: '收信服务器端口必须为数字'
        })
        return
      }
      if (requestPrams['sendPort'] && !requestPrams['sendPort'].match(/\d+/)) {
        Success.toast({
          text: '发信服务器端口必须为数字'
        })
        return
      }
      for (var key in requestPrams) {
        if (!requestPrams[key] && requestPrams[key] !== 0) {
          Success.toast({
            text: this.errorTip[key]
          })
          return
        }
      }
      cb && cb(requestPrams)
    },
    validateSend (cb) { // 发送验证
      let tolist = this.requests['toList']
      if (tolist && !/^([a-zA-Z0-9_.-])+@(([a-zA-Z0-9-])+\.)+([a-zA-Z0-9]{2,4})+$/.test(tolist)) {
        Success.toast({
          text: '收件人格式错误'
        })
        return
      }
      cb && cb()
    },
    requestPrams () { // 构建简单绑定和复杂绑定的请求对象，用于发送请求
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
          emailType: this.isImap ? 2 : 1, // 收件服务器类型 1:pop 2:imap
          receiveHost: '', // 收件服务器地址
          receivePort: '', // 收件服务器端口
          isReceiveSsl: 0, // 是否启用SSL收信 0:表示不使用 1：表示使用
          sendHost: '',  // SMTP发件服务器地址
          sendPort: '',  // SMTP发件服务器端口
          isSendSsl: 0  // 是否启用SSL发信 0:表示不使用 1：表示使用
        }, requests)
      }
      return result
    },
    jumpBindPage () {
      this.isBindPage = true
      this.isBindStart = false
      document.body.style.backgroundColor = '#f2f2f2'
    },
    send () {
      // 调用发送接口
      let self = this
      this.validateSend(() => {
        // 验证通过
        this.showLoading(false, '发送中...')
        this.requests['toList'] = [this.requests['toList']]
        Api.sendEACollectEmailForWeb({ // 用的FCP接口 要加上urlPrefix，因为默认掉的时https接口
          data: this.requests,
          urlPrefix: '/FHH/EM1APAY/',
          always: () => { this.closeLoading(false) }
        }).then(res => {
          let data = res.Value && res.Value || {}
          if (data.errorCode === 0) { // 发送成功
            // 通过jsapi关闭当前页
            Success.toast({
              duration: 1000,
              text: '发送成功',
              afterClose () {
                self.sendSuccess()
              }
            })
          } else {
            Success.toast({
              text: data.errorMessage
            })
          }
        })
      })
    },
    getBindMsg (cb) {
      Api.get({ // 用来判断邮箱是否绑定
        method: 'get'
      }).then(res => {
        if (res.errorCode === 0) { // 成功
          let data = res.data && res.data || {}
          this.sendEmail = data.account || ''
          if (data.status === 3) { // 未绑定
            this.setUnbindStatus()
          } else { // 已绑定
            this.setInitBinded()
          }
        } else if (res.errorCode === -10008) { // 未绑定
          this.setUnbindStatus()
        } else {
          this.setUnbindStatus()
          Success.toast({
            duration: 1000,
            text: res.errorMessage
          })
        }
        cb && cb()
      })
    },
    setUnbindStatus () {
      this.isBind = false
      this.openBindStart && (this.isBindStart = true) || this.jumpBindPage()
    },
    setInitBinded () { // 初始化已绑定的设置
      this.isBind = true
      this.isBindStart = false
      document.body.style.backgroundColor = '#f2f2f2'
      this.requests = {} // 初始化请求对象
      this.requests['orderNo'] = Util.getQuery('orderNo') || ''
      this.requests['urlParameter'] = Util.getQuery('urlParameter') || ''
    },
    tapImapOrPop (evt) { // 切换imap 和 pop
      let target = evt.target
      if (target.innerHTML === 'IMAP') {
        this.isImap = true
      } else {
        this.isImap = false
      }
      // 切换时，更改了收信服务器的数据
      this.updateReceiverDatas()
    },
    jumpManualBindPage () { // 绑定后跳转复杂绑定
      this.isManual = true
      this.updateReceiverDatas()
      this.updateSendDatas()
      this.updateInputTextDatas()
    },
    updateReceiverDatas () { // 更新复杂绑定收信服务器数据
      let host = this.isImap ? 'imap' : 'pop'
      let receiveHostValue = host + '.' + this.requests['account'].split('@')[1]
      this.receiverDatas.datas.splice(0, 1, {
        type: 'text',
        text: '服务器地址',
        placeholder: 'imap.example.com',
        val: receiveHostValue,
        name: 'receiveHost'
      })
      this.receiverDatas.id = host // 触发更新对象
      // 更新请求参数
      this.requests['receiveHost'] = receiveHostValue
      this.requests['isReceiveSsl'] = 0
      this.requests['receivePort'] = ''
    },
    updateSendDatas () { // 更新复杂绑定发信服务器数据
      let sendHostValue = 'smtp.' + this.requests['account'].split('@')[1]
      this.sendDatas.datas.splice(0, 1, {
        type: 'text',
        text: '服务器地址',
        placeholder: 'imap.example.com',
        val: sendHostValue,
        name: 'sendHost'
      })
      this.requests['sendHost'] = sendHostValue
    },
    updateInputTextDatas () {
      this.inputTextDatas.splice(0, 2, {
        type: 'text',
        text: '邮箱账号',
        placeholder: '请输入企业邮箱账号',
        val: this.requests['account'],
        name: 'account'
      }, {
        type: 'password',
        text: '密  码',
        placeholder: '请输入密码',
        val: this.requests['password'],
        name: 'password'
      })
    },
    setTitle () {
      if (this.isBind) { // 已绑定，跳转到发送页面
        document.title = '发邮件'
        FSOpen.webview.navbar.setTitle({ // 已绑定设定的标题
          title: '发邮件'
        })
        document.body.style.backgroundColor = '#f2f2f2'
      } else {
        document.title = '绑定邮箱'
        FSOpen.webview.navbar.setTitle({ // 未绑定设定的标题
          title: '绑定邮箱'
        })
      }
    }
  }
}
</script>

<style lang="less">
@import '~@/assets/style/all';
@activeColor: #fcb058;
.email-bind-header{
  ul{
    display: flex;
   
    width:100%;
    background: #fff;
    li{
      border-bottom:2px solid transparent;
      width:50%;
      height: 40/25rem;
      line-height:40/25rem;
      text-align:center;
      &.active{
        border-color: @activeColor;
        color:  @activeColor;
      }
    }
  }
}
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
.article-istitle{
  .input-left{
    flex: 0 0 88/25rem;
  }
}
.footer{
  margin-top: 15/25rem;
}
.footer-carousel{
   margin-top: 0;
   position:fixed;
   bottom: 0;
   left: 0;
   padding: 15/25rem;
   width:100%;
   background: #fff;
   z-index: 20;
   .button-wrap{
     button{
      color: #fff;
      background:  @activeColor;
      border-radius: 5/25rem;
     }
   }
}
</style>
