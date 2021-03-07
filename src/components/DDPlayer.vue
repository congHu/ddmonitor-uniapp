<template>
  <view class="player-container">
    <video
      class="video-player"
      :id="'live'+liveid"
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
      <cover-view class="danmu" v-show="showDanmu" v-for="(d,i) in danmuList" :key="i">
        {{d}}
      </cover-view>
    </video>
    <view class="toolbar">
      <view class="btn" @click="loadUrl">刷</view>
      <view :class="'btn' + (isMuted ? ' muted' : '')" @click="toggleMute">静</view>
      <view :class="'btn' + (showDanmu ? '' : ' muted')" @click="toggleDanmu">弹</view>
      <view class="upname">{{upname}}</view>
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
      url: '',
      isMuted: false,
      videoCtx: null,
      upname: '',
      socketTask: null,
      socketTimer: null,
      danmuList: [],
      showDanmu: true
    }
  },
  props: {
    liveid: {
      type: String,
      default: "0"
    },
    qn: {
      type: Number, 
      default: 250
    },
  },
  mounted() {
    this.videoCtx = uni.createVideoContext("live"+this.$props.liveid)
    this.loadUrl()
  },
  // watch: {
  //   liveid(newVal, oldVal) {
  //     if (newVal != oldVal) this.loadUrl()
  //   }
  // },
  methods: {
    loadUrl() {
      console.log('this.$props.liveid',this.$props.liveid)
      if (this.socketTask) {
        this.socketTask.close()
      }
      this.danmuList = []
      if (this.socketTimer) clearInterval(this.socketTimer)
      if (this.videoCtx) this.videoCtx.stop()
      this.url = ''
      this.upname = ''
      let that = this
      uni.request({
        url: 'https://api.live.bilibili.com/room/v1/Room/playUrl?cid='+this.$props.liveid+'&platform=web&qn='+this.$props.qn,
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
        url: 'https://api.live.bilibili.com/xlive/web-room/v1/index/getInfoByRoom?room_id=' + this.$props.liveid,
        success(res) {
          // console.log(res)
          try {
            that.upname = res.data.data.anchor_info.base_info.uname
            const roomid = res.data.data.room_info.room_id

            that.socketTask = socket.connectSocket({
              url: 'wss://broadcastlv.chat.bilibili.com/sub',
              complete() {}
            })

            that.socketTask.onOpen((res) => {
              console.log("socket open")
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
              that.socketTimer = setInterval(() => {
                that.socketTask.send({
                  data: new Uint8Array([0,0,0,16,0,16,0,1,0,0,0,2,0,0,0,1]).buffer
                })
              }, 30000);
            })
            that.socketTask.onMessage((data) => {
              // console.log("socket msg", roomid)
              const bytes = new Uint8Array(data.data)
              if (bytes[7] == 2) {
                try {
                  const unziped = pako.inflate(bytes.slice(16,bytes.length))
                  // console.log(unziped)
                  const danmu = that.getDanmu(unziped)
                  if (danmu) {
                    console.log("danmu", roomid, danmu)
                    // that.videoCtx.sendDanmu({
                    //   text: danmu,
                    //   color: '#ffffff'
                    // })
                    that.danmuList.push(danmu)
                    if (that.danmuList.length > 5) that.danmuList.shift()
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
              console.log('ws close', roomid);
            })
            that.socketTask.onError((err) => {
              console.log("ws err", err)
            })
            

          }catch{
          }
        }
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
      let sp = data.subarray(16, data.length)
      // console.log(sp[sp.length-1])
      let jstr = new TextDecoder('utf-8').decode(sp)
      // console.log(typeof jstr)
      try{
        const jo = JSON.parse(jstr)
        console.log(jo)
        if (jo.cmd == "DANMU_MSG") return jo.info[1]
        return null
      }catch{
        let match1 = jstr.match(/\{\"cmd\":\"DANMU_MSG\"/)
        if (match1) {
          // console.log(match1)
          let match2 = jstr.match(/\"info\":\[.*\],\"(.*?)\"\,/)
          if (match2) {
            // console.log(match2)
            return match2[1].length <= 20 ? match2[1] : null
          }
        }
        return null
      }
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
      this.showDanmu = !this.showDanmu
    }
  }
}
</script>

<style>
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
.muted {
  background-color: rgba(0, 0, 0, .6);
}
.danmu {
  font-size: 13px;
  background-color: rgba(0, 0, 0, .6);
}
</style>