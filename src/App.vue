<template>
  <section>
    <article></article>
    <footer class="footer" :class="{disabled: !hasSelect}" ref="footer" @click.stop.prevent="submit">提&nbsp;交</footer>
  </section>
</template>

<script>
import Loading from '@/components/base/loading'
import Success from '@/components/base/success'
import Api from '@/core/service/service'

export default {
  name: 'app',
  data () {
    return {
      loadingTimer: null,
      otherReason: ''
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
    validate ($event) {
      let value = $event.target.value
      let maxLength = Number($event.target.getAttribute('data-maxlength'))
      this.otherReason = $event.target.value = value.substr(0, maxLength)
    },
    scrollBottom () {
      let h = document.documentElement.scrollHeight - document.documentElement.clientHeight
      if (h > 0) document.body.scrollTop = h
    },
    submit ($event) {
      // if ($event.target.classList.contains('disabled')) return
      let submitData = {
        reason: '1',
        other: '2'
      }
      this.showLoading(false, '提交中...')
      Api.get({
        data: submitData,
        method: 'get',
        always: () => { this.closeLoading(false) }
      }).then(res => {
        if (res.errorCode === 0) {
          Success.open('提交成功')
          setTimeout(() => {
            Success.close()
          }, 1000)
        } else {
          alert(res.errorMessage)
        }
      })
    }
  }
}
</script>

<style lang="less">
@import '~@/assets/style/all';

@header-height: 3.75rem;
@footer-height: 2.344rem;

.footer{
  position: relative;
  height: @footer-height;
  line-height: @footer-height;
  font-size: 0.7968rem;
  color: #333333;
  text-align: center;
  cursor: pointer;
  &:active{
    background: #F8F8F8;
  }
  &.disabled{
    color: #cccccc;
  }
  &:after{
    .setTopLine(#E4E6EB);
  }
}
</style>
