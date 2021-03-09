<template>
  <view>
    <view class="list-item">定时关闭：{{autoCloseMinute > 0 ? (autoCloseMinute + '分钟') : '不启用'}}</view>
    <view v-for="op in options" :key="op" class="list-item" hover-class="list-item-hover" @click="optionClick(op)">
      {{op == 0 ? '不启用' : (op == -1 ? '自定义' : (op + '分钟'))}}
    </view>

  </view>
</template>

<script>
export default {
  data() {
    return {
      options: [0,5,10,15,30,-1],
      autoCloseMinute: 0,
    }
  },
  methods: {
    optionClick(op) {
      if (op == -1) {
        plus.nativeUI.prompt("定时关闭时间(分钟)", (res) => {
          if (res.index == 0) {
            const t = parseInt(res.value)
            if (!isNaN(t)) {
              getApp().globalData.autoCloseTime = t*60
              this.autoCloseMinute = t
            }
          }
        })
      }else{
        getApp().globalData.autoCloseTime = op*60
        this.autoCloseMinute = op
      }
    }
  },
  onLoad() {
    this.autoCloseMinute = Math.ceil(getApp().globalData.autoCloseTime/60)
    setInterval(() => {
      this.autoCloseMinute = Math.ceil(getApp().globalData.autoCloseTime/60)
    }, 1000);
  }
}
</script>

<style>
body {
  background-color: #31363b;
  color: white;
}
.list-item {
  padding: 8px;
  border-bottom: 1px solid #808080;
  font-size: 14px;
}
.list-item-hover {
  background-color: #1b1e20;
}
</style>