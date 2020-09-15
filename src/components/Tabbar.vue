<template>
  <van-tabbar class="ac-tabbar" :fixed="false" v-model="active">
    <van-tabbar-item icon="friends-o">共用</van-tabbar-item>
    <van-tabbar-item icon="flower-o">老婆</van-tabbar-item>
    <van-tabbar-item icon="contact">老公</van-tabbar-item>
  </van-tabbar>
</template>

<script>
const tabValurMap = ['shared', 'wife', 'husband']
export default {
  name: 'Tabbar',
  props: {
    value: {
      type: [Number, String],
      default: 0
    }
  },
  model: {
    prop: 'value', // 绑定的值，通过父组件传递
    event: 'change' // 自定义时间名
  },
  computed: {
    active: {
      get(val) {
        return tabValurMap.findIndex(item => item === this.value)
      },
      set(val) {
        this.$emit('change', tabValurMap[val])
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.ac-tabbar {
  & /deep/ { 
    .van-icon {
      font-size: 25px; 
    }
  }
  
}
</style>