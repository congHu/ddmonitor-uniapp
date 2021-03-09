<template>
  <!-- :style="'margin-top:'+safeAreaInsetsTop+'px'" -->
  <view class="content">
    <!-- <view class="topbar">
      <view class="btn" @click="goback">完成</view>
      <view class="title">列表</view>
    </view> -->
    <!-- <view class="scrollview"> -->
    <view class="list-item" hover-class="list-item-hover" v-for="i in 9" :key="i" @click="layoutItemClick(i)">
      窗口{{i}}：
      <span v-if="i <= liveids.length && !isNaN(liveids[i-1]) && liveids[i-1] > 0">
        <span v-if="liveInfos.hasOwnProperty(liveids[i-1])">
          <image style="width:40px;height:40px" :src="liveInfos[liveids[i-1]].face">
          {{liveInfos[liveids[i-1]].isLive == 1 ? '直播中':'未开播'}} {{liveInfos[liveids[i-1]].upname}} {{liveInfos[liveids[i-1]].title}}
        </span>
        <span v-else>加载...</span>
      </span>
      <span v-else>空</span>
    </view>
    <view class="list-title-view">
      <span class="list-title">你的列表</span>
      <span class="list-title-btn" @click="addByLiveid">添加</span>
      <!-- <span style="margin-left:8px;border: 1px solid #808080;padding:4px" @click="addByImport">导入</span> -->
    </view>
    <view class="list-item" hover-class="list-item-hover" v-for="(liveid,i) in saveids" :key="liveid" @click="saveItemClick(i)">
      <span v-if="liveInfos.hasOwnProperty(liveid)">
        <image class="head-image" :src="liveInfos[liveid].face">
        {{liveInfos[liveid].isLive == 1 ? '直播中':'未开播'}} {{liveInfos[liveid].upname}} {{liveInfos[liveid].title}}
      </span>
      <span v-else>加载...</span>
    </view>
    <!-- </view> -->
    
  </view>
</template>

<script>
export default {
  data() {
    return {
      safeAreaInsetsTop: 0,
      liveids: [],
      saveids: [],
      liveInfos: {},
    }
  },
  onLoad() {
    let liveids = uni.getStorageSync('liveids')
    console.log(typeof this.liveids)
    if (liveids) {
      this.liveids = liveids.split(" ")
    }
    let saveids = uni.getStorageSync('saveids')
    if (saveids) {
      this.saveids = saveids.split(" ")
    }else{
      this.saveids = [47377,8792912,21652717,213]
      uni.setStorageSync('saveids', this.saveids.join(" "))
    }
    
    this.liveids.forEach(liveid => {
      this.loadLiveInfo(liveid)
    })
    this.saveids.forEach(liveid => {
      this.loadLiveInfo(liveid)
    })
  },
  onShow() {
    const info = uni.getSystemInfoSync()
    this.safeAreaInsetsTop = info.safeAreaInsets.top
    // this.safeHeight = info.safeArea.height
  },
  methods: {
    loadLiveInfo(liveid) {
      if (liveid > 0 && !this.liveInfos.hasOwnProperty(liveid)) {
        let that = this
        uni.request({
          url: 'https://api.live.bilibili.com/xlive/web-room/v1/index/getInfoByRoom?room_id=' + liveid,
          success(res) {
            // console.log(res)
            try {
              that.$set(that.liveInfos, liveid, {
                title: res.data.data.room_info.title,
                upname: res.data.data.anchor_info.base_info.uname,
                face: res.data.data.anchor_info.base_info.face,
                isLive: res.data.data.room_info.live_status
              })
            }catch{
            }
          }
        })
      }
    },
    goback() {
      uni.navigateBack()
    },
    getLiveId(index) {
      if (index < this.liveids.length) {
        return this.liveids[index]
      }else{
        return 0
      }
    },
    addByLiveid() {
      plus.nativeUI.prompt("UP主直播房间id", (res) => {
        console.log(res)
        if (res.index == 0) {
          this.saveids.push(res.value)
          this.loadLiveInfo(res.value)
          uni.setStorageSync('saveids', this.saveids.join(" "))
        }
      })
    },
    addByImport() {
      plus.nativeUI.prompt("只抓取你的公开关注列表", (res) => {
        console.log(res)
        if (res.index == 0) {
          res.value
        }
      }, "你的uid")
    },
    layoutItemClick(i) {
      if (i <= this.liveids.length && !isNaN(this.liveids[i-1]) && this.liveids[i-1] > 0) {
        let buttons = []
        for (let j = 1; j <= 9; j++) {
          if (i != j) buttons.push({title: "交换窗口"+j})
        }
        buttons.push({title: "关闭此窗口", style: "destructive"})
        plus.nativeUI.actionSheet({
          title: '操作',
          buttons: buttons,
          cancel: '取消'
        }, e => {
          console.log(e.index)
          if (e.index > 0) {
            if (e.index >= 9) {
              this.$set(this.liveids, i-1, 0)
            }else{
              let idx = e.index - 1
              if (e.index >= i) {
                idx = e.index
              }
              if (idx > this.liveids.length) {
                while (idx > this.liveids.length) {
                  this.liveids.push(0)
                }
                // this.liveids[idx] = liveid
                // this.liveids[i-1] = 0
                this.$set(this.liveids, idx, this.liveids[i-1])
                this.$set(this.liveids, i-1, 0)
              }else{
                const tmp = this.liveids[idx]
                // this.liveids[idx] = liveid
                // this.liveids[i-1] = tmp
                this.$set(this.liveids, idx, this.liveids[i-1])
                this.$set(this.liveids, i-1, tmp)
              }
            }
            uni.setStorageSync("liveids", this.liveids.join(" "))
          }
        })
      }
    },
    saveItemClick(i) {
      let buttons = []
      for (let j = 1; j <= 9; j++) {
        buttons.push({title: "窗口"+j})
      }
      buttons.push({title: '从列表移除', style: "destructive"})
      plus.nativeUI.actionSheet({
        title: '添加到',
        buttons: buttons,
        cancel: '取消'
      }, e => {
        console.log(e.index)
        if (e.index > 0) {
          if (e.index >= 9) {
            this.saveids.splice(i, 1)
            uni.setStorageSync("saveids", this.saveids.join(" "))
          }else{
            while (e.index - 1 > this.liveids.length) {
              this.liveids.push(0)
            }
            this.$set(this.liveids, e.index-1, this.saveids[i])
            // this.liveids[e.index-1] = liveid
            uni.setStorageSync("liveids", this.liveids.join(" "))
            
          }
          
        }
      })
    }
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
.list-title-view {
  padding: 8px;
  border-bottom: 1px solid #808080;
}
.list-title {
  font-size: 18px;
  font-weight: bold;
}
.list-title-btn {
  margin-left: 8px;
  border: 1px solid #808080;
  padding: 4px;
  font-size: 13px
}
.head-image {
  width: 40px;
  height: 40px;
}
</style>