<template>
  <div class="input-wraper" >
    <div  class="input-left" >{{inputDatas.text}}</div>
    <div class="input-right"><input :type="inputDatas.type" :placeholder="inputDatas.placeholder" :name="inputDatas.name" :value="value" @input="updateValue($event.target)" ></div>
  </div>
</template>

<script>
  export default {
    props: {
      inputDatas: Object
    },
    data () {
      return{
        value:''
      }
    },
    methods: {
      updateValue: function(target){
        let value = target.value
        let name = target.name
        target.value = value
        this.value = value
        this.$emit('input',value,name)
      }
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
    color:#000;
    background: #fff;
    box-shadow:inset 15/25rem 0 0 0 #fff, inset 0 1px 0 0 #eee, 0 1px 0 0 #eee;
    &:first-child{
      box-shadow:inset 0 1px 0 0 #eee;
    }
    input{
      border:none;
    }
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