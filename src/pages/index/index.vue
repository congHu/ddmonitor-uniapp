<template>
  <view class="content" :style="'padding: '+safeArea.top+'px '+(safeArea.right-safeArea.width-safeArea.left)+'px '+(safeArea.bottom-safeArea.height-safeArea.top)+'px '+safeArea.left+'px; height: '+safeArea.height+'px'">
    <view class="topbar">
      <view class="btn">全局</view>
      <view class="btn iconfont" @click="refreshAll">&#xe618;</view>
      <view class="btn iconfont" @click="muteAll">&#xe747;</view>
      <view class="btn iconfont" @click="danmuCloseAll">&#xe696;</view>
      <view style="flex:1"></view>
      <view class="btn right iconfont" @click="openMessage">&#xe69d;</view>
      <view class="btn right iconfont" @click="openTimerOption">&#xe645;{{autoCloseMinute>0?autoCloseMinute:''}}</view>
      <view :class="'btn right iconfont' + (lockLandscape ? ' active' : '')" @click="toggleLandscape">&#xe664;</view>
      <view class="btn right iconfont" @click="openLayoutOption">&#xebe5;</view>
      <view class="btn right iconfont highlight" @click="openUpList">&#xe64f;编辑</view>
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
      liveStatus: {},
      autoCloseMinute: 0,
      autoCloseStarted: false
    };
  },
  onLoad() {
    if (!uni.getStorageSync('showMessage')) {
      this.openMessage()
    }

    const saveids = uni.getStorageSync('liveids')
    if (saveids) {
      this.saveids = saveids.split(" ")
    }else{
      this.saveids = [47377,8792912,21652717,213]
      uni.setStorageSync('saveids', this.saveids.join(" "))
    }
    
    // setInterval(() => {
    //   // console.log("saveids", this.saveids)
    //   this.saveids.forEach(liveid => {
    //     let that = this
    //     // console.log('https://api.live.bilibili.com/xlive/web-room/v1/index/getInfoByRoom?room_id=' + liveid)
    //     uni.request({
    //       url: 'https://api.live.bilibili.com/xlive/web-room/v1/index/getInfoByRoom?room_id=' + liveid,
    //       success(res) {
    //         // console.log(res)
    //         const newStatus = res.data.data.room_info.live_status
    //         if (newStatus == 1 && that.liveStatus.hasOwnProperty(liveid) && that.liveStatus[liveid] == 0) {
    //           uni.showToast({
    //             title: res.data.data.anchor_info.base_info.uname + ' 开播了',
    //             icon: 'none'
    //           })
    //         }
            
    //         that.$set(that.liveStatus, liveid, newStatus)

    //       }
    //     })
    //   })
    // }, 10000);

    setInterval(() => {
      let time = getApp().globalData.autoCloseTime
      // console.log("time",time)
      this.autoCloseMinute = Math.ceil(time/60)
      if (time > 0) {
        this.autoCloseStarted = true
        getApp().globalData.autoCloseTime -= 1
      }else{
        if (this.autoCloseStarted) {
          this.autoCloseStarted = false
          uni.setKeepScreenOn({
            keepScreenOn: false
          })
          this.pauseAll()
        }
      }
      
    }, 1000);
    
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
    console.log("index onshow")
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
    }else{
      uni.setStorageSync('layout', this.layout)
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
        }else{
          console.log("keep play")
          this.keepPlayAll()
        }
      }else{
        this.liveids = liveidList
        // this.refreshAll()

        this.$nextTick(() => {
          this.refreshAll()
        })
      }
    }

    const saveids = uni.getStorageSync('saveids')
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
    keepPlayAll() {
      this.$refs.dd.keepPlayAll()
    },
    pauseAll() {
      this.$refs.dd.pauseAll()
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
    },
    openTimerOption() {
      uni.navigateTo({
        url: '/pages/timerOption/index'
      })
    },
    openMessage() {
      uni.showModal({
        title: 'DD监控器v1.0.8 by CongHu',
        content: '· 点击右上角"编辑"按钮添加直播UP主。\n· 观看多个直播时请注意宽带网速、流量消耗、电池电量、机身发热、系统卡顿等软硬件环境问题。',
        cancelText: '关闭',
        confirmText: 'B站视频',
        success(res) {
          console.log(res)
          uni.setStorageSync('showMessage', 1)
          if (res.confirm) {
            // plus.runtime.openURL('bilibili://live/5050')
          }

        }
      })
    }
  },
  destroyed() {
    this.landScapeObs.disconnect()
  }
};
</script>

<style>
@font-face {
  font-family: 'iconfont';
  src: url('/static/iconfont/iconfont.eot');
  src: url('/static/iconfont/iconfont.eot?#iefix') format('embedded-opentype'),
      url('/static/iconfont/iconfont.woff2') format('woff2'),
      url('/static/iconfont/iconfont.woff') format('woff'),
      url('/static/iconfont/iconfont.ttf') format('truetype'),
      url('/static/iconfont/iconfont.svg#iconfont') format('svg');
}
.iconfont {
  font-family: "iconfont" !important;
  font-style: normal;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  line-height: 20px;
}
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
.active {
  background-color: rgba(0, 0, 0, .6);
}
.highlight {
  background-color: #033b66;
}
</style>
