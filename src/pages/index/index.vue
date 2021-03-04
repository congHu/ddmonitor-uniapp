<template>
  <view class="content">
    <view class="topbar">
      <view class="btn" @click="refreshAll">全部刷新</view>
      <view class="btn" @click="muteAll">全部静音</view>
      <view class="btn" @click="danmuCloseAll">弹幕全关</view>
      <view style="flex:1"></view>
      <view class="btn right" @click="toggleLandscape">{{lockLandscape?'解锁横屏':'锁定横屏'}}</view>
      <view class="btn right" @click="openLayoutOption">更改布局</view>
      <view class="btn right" @click="openUpList">UP列表</view>
    </view>
    <DDLayouts :layout="layout" :liveids="liveids" ref="dd" />
  </view>
</template>

<script>
import DDLayouts from '../../components/DDLayouts.vue';
export default {
  components: { DDLayouts },
  data() {
    return {
      lockLandscape: false,
      layout: 3,
      liveids: [],
    };
  },
  onLoad() {
    // uni.createMediaQueryObserver(this).observe({
    //   orientation: 'landscape'
    // }, match => {
    //   console.log("landscape", match)
    // })
    
  },
  onShow() {
    uni.setKeepScreenOn({
      keepScreenOn: true
    })
    const layout = uni.getStorageSync('layout')
    if (layout) {
      this.layout =  layout
    }
    const liveids = uni.getStorageSync('liveids')
    console.log("liveids", liveids)
    if (liveids) {
      this.liveids = liveids.split(" ")
    }
    this.$nextTick(() => {
      this.refreshAll()
    })
  },
  onHide() {
    uni.setKeepScreenOn({
      keepScreenOn: false
    })
  },
  methods: {
    refreshAll() {
      console.log("index refresh")
      this.$refs.dd.refreshAll()
    },
    muteAll() {
      this.$refs.dd.muteAll()
    },
    danmuCloseAll() {
      this.$refs.dd.danmuCloseAll()
    },
    toggleLandscape() {
      this.lockLandscape = !this.lockLandscape
      if (this.lockLandscape) {
        plus.screen.lockOrientation('landscape')
      }else{
        plus.screen.unlockOrientation()
      }
    },
    openLayoutOption() {
      uni.navigateTo({
        url: '/pages/layoutOption/index'
      })
    },
    openUpList() {
      uni.navigateTo({
        url: '/pages/uplist/index'
      })
    }
  },
};
</script>

<style>
.content {
  display: flex;
  flex-direction: column;
  height: 100vh;
}
DDLayout {
  flex: 1;
}
.topbar {
  display: flex;
  background-color: #31363b;
  border-bottom: 1px solid #808080;
  color: white;
  font-size: 13px;
}
.topbar .btn {
  padding: 4px;
  border-right: 1px solid #808080;
  box-sizing: border-box;
}
.topbar .btn.right {
  border-right: none;
  border-left: 1px solid #808080;

}
</style>
