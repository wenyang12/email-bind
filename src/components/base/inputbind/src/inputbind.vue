<template>
  <div class="input-wraper" :key="key" >
    <div  class="input-left" >{{inputData.text}}</div>
    <div class="input-right"><input :type="inputData.type"></div>
  </div>
</template>

<script>
  export default {
    props: {
      inputData: Object,
      key: Number
    }
  };
</script>

<style lang="less">
@inputHeight: 48/25rem;
  .input-wraper{
    display: flex;
    padding: 0 15/25rem ;
    height: @inputHeight;
    line-height: @inputHeight;
    font-size: 17/25rem;
  }
  .input-left{
    flex: 0 0 68/25rem;
  }
   .input-right{
    flex: auto;
    margin-left: 24/25rem;
    input{
      width: 100%;
    }
  }
</style>