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
      landScapeObs: null,
      saveids: [],
      liveStatus: {}
    };
  },
  onLoad() {
    setInterval(() => {
      this.saveids.forEach(liveid => {
        let that = this
        uni.request({
          url: 'https://api.live.bilibili.com/xlive/web-room/v1/index/getInfoByRoom?room_id=' + liveid,
          success(res) {
            // console.log(res)
            const newStatus = res.data.data.room_info.live_status
            if (newStatus == 1 && that.liveStatus[liveid] == 0) {
              uni.showToast({
                title: res.data.data.anchor_info.base_info.uname + ' 开播了'
              })
            }
            
            that.$set(that.liveStatus, liveid, newStatus)

          }
        })
      })
    }, 10000);
    
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
    let layoutChanged = false
    const layout = uni.getStorageSync('layout')
    if (layout) {
      if (this.layout != layout) {
        layoutChanged = true
        this.layout = layout
      }
    }

    const info = uni.getSystemInfoSync()
    console.log("info", info)
    this.safeArea = {...info.safeArea}
    // this.safeHeight = info.safeArea.height
    console.log("safearea", this.safeArea)


    const liveids = uni.getStorageSync('liveids')
    console.log("liveids", liveids)
    
    if (liveids) {
      const liveidList = liveids.split(" ")
      if (liveidList.length == this.liveids.length) {
        let liveidsChanged = false
        for (let i in liveidList) {
          if (liveidList[i] != this.liveids[i]) {
            liveidsChanged = true
            this.liveids = liveidList
            // this.refreshAll()

            this.$nextTick(() => {
              this.refreshAll()
            })
            break
          }
        }
        if (layoutChanged && !liveidsChanged) {
          this.$nextTick(() => {
            this.refreshAll()
          })
        }
      }else{
        this.liveids = liveidList
        // this.refreshAll()

        this.$nextTick(() => {
          this.refreshAll()
        })
      }
    }

    const saveids = uni.getStorageSync('liveids')
    if (saveids) {
      this.saveids = saveids.split(" ")
    }
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
