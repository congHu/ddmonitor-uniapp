<template>
  <view class="content" :style="'padding: '+safeArea.top+'px '+(safeArea.right-safeArea.width-safeArea.left)+'px '+(safeArea.bottom-safeArea.height-safeArea.top)+'px '+safeArea.left+'px; height: '+safeArea.height+'px'">
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
      safeArea: {
        top: 0,
        right: 0,
        bottom: 0,
        left: 0,
        width: 0,
        height: 0,
      },
      safeHeight: 0,
      landScapeObs: null
    };
  },
  onLoad() {
    
    
  },
  // mounted() {
  //   this.landScapeObs = uni.createMediaQueryObserver(this)
  //   this.landScapeObs.observe({
  //     orientation: 'landscape'
  //   }, (landscape) => {
  //     console.log("landscape", landscape)
  //     this.safeHeight = info.safeArea.height
  //     const info = uni.getSystemInfoSync()
  //     if (landscape) {
  //       const top = this.safeAreaInsets.top
  //       this.safeAreaInsets.top = this.safeAreaInsets.right
  //       this.safeAreaInsets.right = top
  //     }
  //   })
  // },
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
    const info = uni.getSystemInfoSync()
    console.log("info", info)
    this.safeArea = {...info.safeArea}
    console.log("safearea", this.safeArea)
    // this.safeHeight = info.safeArea.height
    this.$nextTick(() => {
      this.refreshAll()
    })
  },
  onResize() {
    const info = uni.getSystemInfoSync()
    console.log("resize", info)
    this.safeArea = {...info.safeArea}
    // this.safeHeight = info.safeArea.height

    // if (info.)
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
  destroyed() {
    this.landScapeObs.disconnect()
  }
};
</script>

<style>
body {
  background-color: #31363b;
  color: white;
  /* display: flex;
  flex-direction: column; */
  /* margin-top: env(safe-area-inset-top);
  margin-bottom: env(safe-area-inset-bottom);
  height: calc(100% - env(safe-area-inset-top) - env(safe-area-inset-bottom)); */
}
.content {
  display: flex;
  flex-direction: column;
  /* height: 100vh; */
  
}
DDLayout {
  flex: 1;
}
.topbar {
  display: flex;
  background-color: #31363b;
  border-bottom: 1px solid #808080;
  border-top: 1px solid #808080;
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
