<template>
  <div class="SelectButtons">
    <div
      v-for="(button, index) in buttons"
      :key="index" class="ac-select__item"
      :class="{'actived': active === button.val}"
      @click="active = button.val"
    >
      {{ button.name }}
    </div>
  </div>
</template>

<script>
export default {
  name: 'SelectButtons',
  model: {
    prop: 'value', // 绑定的值，通过父组件传递
    event: 'change' // 自定义时间名
  },
  props: {
    value: {
      type: [Number, String],
      default: ''
    },
    buttons: {
      type: Array,
      default: () => {
        return [
          {name: '請傳buttons近來', val: '請傳buttons近來'}
        ]
      }
    }
  },
  computed: {
    active: {
      get() {
        return this.value
      },
      set(val) {
        this.$emit('change', val)
      }
    }
  }
}
</script>

<style lang="scss">
.SelectButtons {
  padding: 0;
  display: flex;
  .ac-select__item {
    padding: 4px 10px;
    border: 1px #cecbcb solid;
    border-right: 0px;

    &:nth-child(1) {
      border-radius: 5px 0 0 5px;
    }

    &:nth-last-child(1) {
      border-radius: 0 5px 5px 0;
      border-right: 1px #cecbcb solid;
    }

    &.actived {
      background: #1989fa;
      color: white;
    }
  }
}
</style>