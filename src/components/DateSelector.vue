<template>
  <div class="selector-container">
    <span @click="show = true">
      {{ value }}
    </span>
    <div class="selector-icon">
      <van-icon name="calender-o" color="rgb(145, 142, 142)" size="17px" />
    </div>
    <van-calendar
      v-model="show"
      color="#1989fa"
      :default-date="defaultDate"
      :min-date="minDate"
      :max-date="maxDate"
      :get-container="getContainer"
      @confirm="onConfirm"
    />
  </div>
</template>

<script>
import moment from 'moment'

export default {
  name: 'DateSelector',
  props: {
    value: {
      type: [String, Object],
      default: () => {
        return null
      }
    }
  },
  model: {
    prop: 'value', // 绑定的值，通过父组件传递
    event: 'change' // 自定义时间名
  },
  computed: {
    defaultDate() {
      return new Date(this.value)
    }
  },
  data() {
    return {
      show: false,
      minDate: new Date(2019, 0, 1),
      maxDate: new Date(2030, 11, 30),
    }
  },
  methods: {
    onConfirm(val) {
      this.$emit('change', moment(val).format('YYYY/MM/DD'))
      this.show = false
    },
    // 返回一个特定的 DOM 节点，作为挂载的父节点
    getContainer() {
      return document.querySelector('#app');
    }
  }
}
</script>

<style lang="scss" scoped>
.selector-container {
  border-radius: 4px;
  border: 1px #cecbcb solid;
  display: inline-block;
  font-size: 14px;
  padding: 5px 10px;

  .selector-icon {
    display: inline-block;
    transform: translate(0px, 3px);
  }
}
</style>