<template>
  <view class="body" v-show="firstRunShow">
    <view class="main">
      <p>多个直播视频流并行解码会消耗CPU、内存、网络流量等资源，请确保您的软硬件运行环境。</p>
      <p>本软件通过B站公共API实现功能，不支持发送弹幕、发送礼物等需要登录账号的功能。</p>
    </view>
    <view class="buttons">
      <checkbox-group @change="hideNextTimeChecked">
        <checkbox value="hideWarning">已知晓以上警告，下次不再提醒</checkbox>
      </checkbox-group>
      <button @click="startBtnClick">开始使用</button>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      firstRunShow: false,
      hideWarning: false
    }
  },
  onLoad() {
    // console.log(uni.getStorageSync('hideWarning'))
    if (uni.getStorageSync('hideWarning')) {
      uni.reLaunch({
        url: '/pages/index/index'
      })
    }else{
      this.firstRunShow = true
    }
  },
  methods: {
    hideNextTimeChecked(e) {
      console.log('checked', e)
      this.hideWarning = e.detail.value.indexOf('hideWarning') >= 0
    },
    startBtnClick() {
      console.log(this.hideWarning)
      if (this.hideWarning) {
        // uni.setStorage({
        //   key: 'hideWarning',
        //   data: true
        // })
        uni.setStorageSync('hideWarning', 1)
      }
      uni.navigateTo({
        url: '/pages/index/index'
      })
    }
  }
}
</script>

<style>
.body {
  height: 100vh;
}
/* .main {
  padding: 30px;
} */
.main * {
  margin: 16px 20px;
}
.buttons {
  position: absolute;
  width: 100%;
  bottom: 60px;
}
.buttons>* {
  margin: 16px 20px;
}
</style>