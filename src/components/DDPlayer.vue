<template>
  <view class="player-container">
    <video
      class="video-player"
      :id="'live'+id"
      :src="url" 
      :autoplay="true"

      :muted="isMuted"
      
      :controls="false"

      :show-progress="false"
      :enable-progress-gesture="false"

      :show-mute-btn="true"
      :auto-pause-if-navigate="false"

      @ended="videoended"
      @error="videoerr"
    >
      <cover-view class="danmu" v-show="showDanmu && !showDanmuOptions" v-for="(d,i) in danmuList" :key="i" :style="danmuPositionStyle(i)">
        {{d}}
      </cover-view>
      <cover-view style="top:0px" class="danmu-options" v-show="showDanmu && showDanmuOptions">
        弹幕设置
      </cover-view>
      <cover-view style="top:32px" class="danmu-options" v-show="showDanmu && showDanmuOptions">
        位置
      </cover-view>
      <cover-view style="top:32px;left:32px" class="danmu-options btn" v-show="showDanmu && showDanmuOptions" @click="danmuPosOption('lt')">
        {{danmuPos=='lt'?'√':''}}左上
      </cover-view>
      <cover-view style="top:32px;left:80px" class="danmu-options btn" v-show="showDanmu && showDanmuOptions" @click="danmuPosOption('lb')">
        {{danmuPos=='lb'?'√':''}}左下
      </cover-view>
      <cover-view style="top:32px;left:128px" class="danmu-options btn" v-show="showDanmu && showDanmuOptions" @click="danmuPosOption('rt')">
        {{danmuPos=='rt'?'√':''}}右上
      </cover-view>
      <cover-view style="top:32px;left:172px" class="danmu-options btn" v-show="showDanmu && showDanmuOptions" @click="danmuPosOption('rb')">
        {{danmuPos=='rb'?'√':''}}右下
      </cover-view>
      <cover-view style="top:64px" class="danmu-options" v-show="showDanmu && showDanmuOptions">
        数量
      </cover-view>
      <cover-view style="top:64px;left:32px" class="danmu-options btn" v-show="showDanmu && showDanmuOptions" @click="danmuCountOption(2)">
        {{danmuCount==2?'√':''}}2条
      </cover-view>
      <cover-view style="top:64px;left:72px" class="danmu-options btn" v-show="showDanmu && showDanmuOptions" @click="danmuCountOption(5)">
        {{danmuCount==5?'√':''}}5条
      </cover-view>
      <cover-view style="top:64px;left:120px" class="danmu-options btn" v-show="showDanmu && showDanmuOptions" @click="danmuCountOption(10)">
        {{danmuCount==10?'√':''}}10条
      </cover-view>
      <cover-view style="top:96px" class="danmu-options" v-show="showDanmu && showDanmuOptions" @click="danmuOptionFinish">
        完成
      </cover-view>
    </video>
    <view class="toolbar">
      <view class="btn iconfont" @click="loadUrl">&#xe618;</view>
      <view :class="'btn iconfont' + (isMuted ? ' active' : '')" @click="toggleMute">&#xe747;</view>
      <view :class="'btn iconfont' + (showDanmu ? '' : ' active')" @click="toggleDanmu">&#xe696;</view>
      <view class="upname" @click="goChangeLiveId">#{{id+1}}: {{liveid==0||isLive==1?'':'(未开播)'}} {{upname?upname:'空'}}</view>
    </view>
  </view>
</template>

<script>
import socket from 'plus-websocket'
const pako = require('pako')
require('fast-text-encoding')


export default {
  data() {
    return {
      liveid: 0,
      url: '',
      isMuted: false,
      videoCtx: null,
      upname: '',
      isLive: 0,
      socketTask: null,
      socketTimer: null,
      danmuList: [],
      showDanmu: true,
      showDanmuOptions: false,
      danmuPos: 'lt',
      danmuCount: 5,
      isDestroying: false,
    }
  },
  props: {
    id: {
      type: Number,
      default: 0
    },
    // liveid: {
    //   type: String,
    //   default: "0"
    // },
    // qn: {
    //   type: Number, 
    //   default: 250
    // },
  },
  mounted() {
    this.videoCtx = uni.createVideoContext("live"+this.liveid)

    let layout = uni.getStorageSync("layout")
    if (isNaN(layout)) {
      layout = 3
    }
    let danmu = uni.getStorageSync("danmu-layout"+layout+"-"+this.$props.id)
    if (danmu) {
      let danmuOptions = danmu.split(" ")
      this.danmuPos = danmuOptions[0]
      this.danmuCount = danmuOptions[1]
    }

    // this.loadUrl()
  },
  beforeDestroy() {
    this.isDestroying = true
    if (this.socketTask) {
      this.socketTask.close()
    }
    this.danmuList = []
    if (this.socketTimer) clearInterval(this.socketTimer)
    if (this.videoCtx) this.videoCtx.stop()
    this.url = ''
    this.upname = ''
  },
  // watch: {
  //   liveid(newVal, oldVal) {
  //     if (newVal != oldVal) this.loadUrl()
  //   }
  // },
  methods: {
    loadUrl() {
      console.log('this.liveid',this.liveid)
      if (this.socketTask) {
        this.socketTask.close()
      }
      this.danmuList = []
      if (this.socketTimer) clearInterval(this.socketTimer)
      if (this.videoCtx) 
        this.videoCtx.stop()
      this.url = ''
      this.upname = ''
      if (this.liveid <= 0) return
      this.upname = '...'
      let that = this
      uni.request({
        url: 'https://api.live.bilibili.com/room/v1/Room/playUrl?cid='+this.liveid+'&platform=web&qn=80', //+this.$props.qn
        success(res) {
          console.log(res)

          // res.data.data.durl.forEach(durl => {
          //   that.lives[liveid].urls.push(durl.url)
          // })
          try {
            if (res.data.data.durl.length > 0) {
              // Vue.set(that.lives, liveid, {
              //   ...that.lives[liveid],
              //   url: durl[0].url
              // })
              // res.data.data.durl.forEach(durl => {
              //   that.liveData[i].urls.push(durl.url)
              // })
              that.url = res.data.data.durl[0].url
              if (that.url != '') that.videoCtx.play()
            }
          }catch(er) {
            console.log(er)
          }
          // that.lives[liveid].urls.push('http://devimages.apple.com/iphone/samples/bipbop/bipbopall.m3u8')
          // console.log(that.lives)

          // console.log(that.liveData)
        }
      })

      uni.request({
        url: 'https://api.live.bilibili.com/xlive/web-room/v1/index/getInfoByRoom?room_id=' + this.liveid,
        success(res) {
          // console.log(res)
          try {
            that.upname = res.data.data.anchor_info.base_info.uname
            console.log(res.data.data.room_info.live_status)
            that.isLive = res.data.data.room_info.live_status
            const roomid = res.data.data.room_info.room_id

            that.connectSocket(roomid)
            

          }catch{
          }
        }
      })
    },
    connectSocket(roomid) {
      let that = this
      that.socketTask = socket.connectSocket({
        url: 'wss://broadcastlv.chat.bilibili.com/sub',
        complete() {}
      })

      that.socketTask.onOpen((res) => {
        console.log("socket open", roomid)
        const reqStr = JSON.stringify({"roomid":roomid})
        let data = [
          0,0,0,reqStr.length+16,
          0,16,0,1,
          0,0,0,7,
          0,0,0,1
        ]
        // const payload = data.concat(...(new TextEncoder()).encode(reqStr))
        for (let i = 0; i < reqStr.length; i++) {
          data.push(reqStr.charCodeAt(i));
        }
        console.log(data)
        that.socketTask.send({
          data: new Uint8Array(data).buffer,
        })
        that.socketTask.send({
          data: new Uint8Array([0,0,0,16,0,16,0,1,0,0,0,2,0,0,0,1]).buffer
        })
        that.socketTimer = setInterval(() => {
          console.log("heartbeat")
          that.socketTask.send({
            data: new Uint8Array([0,0,0,16,0,16,0,1,0,0,0,2,0,0,0,1]).buffer
          })
        }, 30000);
      })
      that.socketTask.onMessage((data) => {
        if (that.isDestroying) return

        // console.log("socket msg", roomid)
        const bytes = new Uint8Array(data.data)
        if (bytes[7] == 2) {
          try {
            const unziped = pako.inflate(bytes.slice(16,bytes.length))
            // console.log(unziped)
            // let td = new TextDecoder('utf-8')
            // try{

            // }catch{

            // }
            const danmu = that.getDanmu(unziped)
            if (danmu) {
              // console.log("danmu", roomid, danmu)
              // that.videoCtx.sendDanmu({
              //   text: danmu,
              //   color: '#ffffff'
              // })
              that.danmuList.push(...danmu)
              if (that.danmuList.length > that.danmuCount) that.danmuList.splice(0,that.danmuList.length-that.danmuCount)
            }
          }catch{

          }
          
        }
        // else{
        //   const danmu = that.getDanmu(bytes)
        //   if (danmu) {
        //     console.log("danmu", roomid, danmu)
        //   }
        // }
      })
      that.socketTask.onClose((res) => {
        console.log('socket close', roomid, res);
        // if (!that.isDestroying) that.connectSocket(roomid)
      })
      that.socketTask.onError((err) => {
        console.log("socket err", err)
        // if (!that.isDestroying) that.connectSocket(roomid)
      })
    },
    toggleMute() {
      this.isMuted = !this.isMuted
    },
    doMute() {
      this.isMuted = true
    },
    videoended(e) {

    },
    videoerr(e) {

    },
    getDanmu(data) {
      // let jstr = ""
      // for (let i = 16; i < data.length; i++) {
      //   jstr += String.fromCharCode(data[i])
      // }
      // console.log(data, data.slice(16,data.length))
      // console.log(data.subarray(0,16), data[16])
      // let sp = data.subarray(16, data.length)
      // console.log(sp[sp.length-1])

      let len = 0
      let danmu = []
      while (len < data.length) {
        // let lenData = data.subarray(len,len+4)
        let nextLen = data[len+2]*256 + data[len+3]
        let jstr = new TextDecoder('utf-8').decode(data.subarray(len+16, len+nextLen))
        len += nextLen
        try{
          const jo = JSON.parse(jstr)
          // console.log(jo)
          if (jo.cmd == "DANMU_MSG") danmu.push(jo.info[1])
        }catch{}
      }

      return danmu

      // let jstr = new TextDecoder('utf-8').decode(sp)
      // // console.log(typeof jstr)
      // try{
      //   const jo = JSON.parse(jstr)
      //   console.log(jo)
      //   if (jo.cmd == "DANMU_MSG") return jo.info[1]
      //   return null
      // }catch{
      //   let match1 = jstr.match(/\{\"cmd\":\"DANMU_MSG\"/)
      //   if (match1) {
      //     // console.log(match1)
      //     let match2 = jstr.match(/\"info\":\[.*\],\"(.*?)\"\,/)
      //     if (match2) {
      //       // console.log(match2)
      //       return match2[1].length <= 20 ? match2[1] : null
      //     }
      //   }
      //   return null
      // }
      // try {
      //   const jo = JSON.parse(jstr.trim())
      //   // console.log(jo)
      //   if (jo.cmd == "DANMU_MSG") return jo.info[1]
      //   return null
      // }catch(err){
      //   // console.log(err)
      //   return null
      // }
    },
    danmuClose() {
      this.showDanmu = false
    },
    toggleDanmu() {
      if (this.showDanmu) {
        if (this.showDanmuOptions) {
          this.showDanmuOptions = false
          this.saveDanmuOptions()
        }else{
          this.showDanmu = false
        }
      }else{
        this.showDanmu = true
        this.showDanmuOptions = true
      }
      
    },
    keepPlay() {
      this.videoCtx.play()
    },
    pause() {
      this.videoCtx.pause()
    },
    goChangeLiveId() {
      let buttons = []
      if (this.liveid > 0) {
        buttons = [{title: "打开B站APP直播间"},{title: "更换UP主"},{title: "关闭此窗口", style: "destructive"}]
      }else{
        buttons = [{title: "更换UP主"}]
      }
      plus.nativeUI.actionSheet({
        title: '操作',
        buttons: buttons,
        cancel: '取消'
      }, e => {
        if (this.liveid > 0) {
          if (e.index == 1) {
            plus.runtime.openURL('bilibili://live/'+this.liveid)
          }else if (e.index == 2) {
            uni.navigateTo({
              url: '/pages/uplist/index?layoutId='+this.$props.id
            })
          }else if (e.index == 3) {
            this.liveid = 0
            this.$nextTick(()=>{
              this.loadUrl()
            })
            
          }
        }else{
          if (e.index == 1) {
            uni.navigateTo({
              url: '/pages/uplist/index?layoutId='+this.$props.id
            })
          }
        }
        
      })
      
    },
    danmuPositionStyle(i) {
      let styleString = ''
      if (this.danmuPos.startsWith('r')) {
        styleString += 'right:0;'
      }
      if (this.danmuPos.endsWith('t')) {
        styleString += 'top:' + (i*16) + 'px'
      }else if (this.danmuPos.endsWith('b')) {
        styleString += 'bottom:' + ((this.danmuCount - i - 1)*16) + 'px'
      }
      return styleString
    },
    danmuPosOption(pos) {
      this.danmuPos = pos
    },
    danmuCountOption(count) {
      this.danmuCount = count
      if (this.danmuList.length > this.danmuCount) {
        this.danmuList.splice(0, this.danmuList.length - this.danmuCount)
      }
    },
    danmuOptionFinish() {
      this.showDanmuOptions = false

      this.saveDanmuOptions()
    },
    saveDanmuOptions() {
      let layout = uni.getStorageSync("layout")
      if (isNaN(layout)) {
        layout = 3
      }
      uni.setStorageSync("danmu-layout"+layout+"-"+this.id, this.danmuPos + ' ' + this.danmuCount)
    }
  }
}
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
  line-height: 18px;
}
.player-container {
  flex: 1;
  display: flex;
  flex-direction: column;
  border-right: 1px solid #808080;
  border-bottom: 1px solid #808080;
  color: white;
}
.video-player {
  width: 100%;
  flex: 1;
}
.toolbar {
  display: flex;
  background-color: #31363b;
  overflow: hidden;
  border-top: 1px solid #808080;
  font-size: 13px;
}
.toolbar .btn {
  padding: 4px;
  border-right: 1px solid #808080;
}
.toolbar .upname {
  padding: 4px;
  flex: 1;
  text-overflow: ellipsis;
  white-space: nowrap;
  width: 0;
}
.active {
  background-color: rgba(0, 0, 0, .6);
}
.danmu {
  font-size: 13px;
  background-color: rgba(0, 0, 0, .6);
  line-height: 1.3;
  position: absolute;
}
.danmu-options {
  font-size: 15px;
  background-color: rgba(0, 0, 0, .6);
  line-height: 1.4;
  position: absolute;
}
.danmu-options.btn {
  /* border: 1px solid #808080; */
  /* color: #a0a0a0; */
}
</style>