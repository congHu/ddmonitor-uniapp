<template>
  <!-- :style="'margin-top:'+safeAreaInsetsTop+'px'" -->
  <view class="content">
    <!-- <view class="topbar">
      <view class="btn" @click="goback">完成</view>
      <view class="title">列表</view>
    </view> -->
    <!-- <view class="scrollview"> -->
    <view class="scroll-list" v-if="isNaN(layoutId)">
      <view class="list-title-view">
        <span class="list-title">窗口列表</span>
      </view>
      <view class="list-item" hover-class="list-item-hover" v-for="i in 9" :key="i" @click="layoutItemClick(i)">
        <view>窗口#{{i}}：</view>
        <view v-if="i <= liveids.length && !isNaN(liveids[i-1]) && liveids[i-1] > 0">
          <view v-if="liveInfos.hasOwnProperty(liveids[i-1])" :class="'up-item' + (liveInfos[liveids[i-1]].isLive == 1 ? ' active':'')">
            <img-cache lazy-load class="head-image" :src="liveInfos[liveids[i-1]].face"></img-cache>
            <view class="title">
              <view class="text">{{liveInfos[liveids[i-1]].isLive == 1 ? '直播中':'未开播'}} {{liveInfos[liveids[i-1]].upname}}</view>
              <view class="text">{{liveInfos[liveids[i-1]].title}}</view>
            </view>
            
          </view>
          <view v-else>加载...</view>
        </view>
        <view style="color:#a0a0a0" v-else>空</view>
      </view>
    </view>
    <view class="scroll-list">

      <view v-if="!isNaN(layoutId)" class="list-item" hover-class="list-item-hover" @click="layoutItemClick(layoutId+1)">
        <view>目前已选：</view>
        <view v-if="layoutId <= liveids.length && !isNaN(liveids[layoutId]) && liveids[layoutId] > 0">
          <view v-if="liveInfos.hasOwnProperty(liveids[layoutId])" :class="'up-item' + (liveInfos[liveids[layoutId]].isLive == 1 ? ' active':'')">
            <img-cache lazy-load class="head-image" :src="liveInfos[liveids[layoutId]].face"></img-cache>
            <view class="title">
              <view class="text">{{liveInfos[liveids[layoutId]].isLive == 1 ? '直播中':'未开播'}} {{liveInfos[liveids[layoutId]].upname}}</view>
              <view class="text">{{liveInfos[liveids[layoutId]].title}}</view>
            </view>
            
          </view>
          <view v-else>加载...</view>
        </view>
        <view style="color:#a0a0a0" v-else>空</view>
      </view>

      <view class="list-title-view">
        <span class="list-title">{{isNaN(layoutId)?'你的列表':('更换窗口#'+(layoutId+1)+'的UP主')}}</span>
        <span class="list-title-btn" @click="addByLiveid">添加</span>
        <span class="list-title-btn" @click="addByImport">导入</span>
      </view>

      <view class="list-item" hover-class="list-item-hover" v-for="(liveid,i) in saveids" :key="liveid" @click="saveItemClick(i)">
        <view :class="'up-item' + (liveInfos[liveid].isLive == 1 ? ' active':'')" v-if="liveInfos.hasOwnProperty(liveid)">
          <img-cache lazy-load class="head-image" :src="liveInfos[liveid].face"></img-cache>
          <view class="title">
            <view class="text">{{liveInfos[liveid].isLive == 1 ? '直播中':'未开播'}} {{liveInfos[liveid].upname}}</view>
            <view class="text">{{liveInfos[liveid].title}}</view>
          </view>
        </view>
        <view v-else>加载...</view>
      </view>
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
      layoutId: NaN,
    }
  },
  onLoad(options) {
    this.layoutId = parseInt(options.layoutId)
  },
  onShow() {
    const info = uni.getSystemInfoSync()
    this.safeAreaInsetsTop = info.safeAreaInsets.top
    // this.safeHeight = info.safeArea.height

    let liveids = uni.getStorageSync('liveids')
    if (liveids) {
      this.liveids = liveids.split(" ")
    }
    this.liveids.forEach(liveid => {
      this.loadLiveInfo(liveid)
    })

    let saveids = uni.getStorageSync('saveids')
    if (saveids) {
      this.saveids = saveids.split(" ")
    }else{
      this.saveids = [47377,8792912,21652717,213]
      uni.setStorageSync('saveids', this.saveids.join(" "))
    }
    this.saveids.forEach(liveid => {
      this.loadLiveInfo(liveid)
    })
  },
  methods: {
    loadLiveInfo(liveid,add=false) {
      if (liveid > 0 && !this.liveInfos.hasOwnProperty(liveid)) {
        let that = this
        uni.request({
          url: 'https://api.live.bilibili.com/xlive/web-room/v1/index/getInfoByRoom?room_id=' + liveid,
          success(res) {
            // console.log(res)
            try {
              if (add) {
                that.saveids.push(liveid)
                uni.setStorageSync('saveids', that.saveids.join(" "))
              }
              that.$set(that.liveInfos, liveid, {
                title: res.data.data.room_info.title,
                upname: res.data.data.anchor_info.base_info.uname,
                face: res.data.data.anchor_info.base_info.face,
                isLive: res.data.data.room_info.live_status
              })
            }catch{
              if (add) {
                uni.showToast({
                  icon: 'none',
                  title: '查询UP主失败'
                })
              }
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
      plus.nativeUI.prompt("如直播间地址：\nhttps://live.bilibili.com/1017\n，则直播间id为1017", (res) => {
        console.log(res)
        if (res.index == 0) {
          if (isNaN(res.value)) {
            uni.showToast({
              icon: 'none',
              title: '无效的UP主id'
            })
            return
          }
          this.loadLiveInfo(res.value, true)
        }
      }, "输入UP主直播房间id")
    },
    addByImport() {
      uni.navigateTo({
        url: '/pages/uidImport/index'
      })
    },
    layoutItemClick(i) {
      if (i <= this.liveids.length && !isNaN(this.liveids[i-1]) && this.liveids[i-1] > 0) {
        if (!isNaN(this.layoutId)) {
          plus.nativeUI.actionSheet({
            title: '操作',
            buttons: [{title: "打开B站APP直播间"},{title: "关闭此窗口", style: "destructive"}],
            cancel: '取消'
          }, e => {
            if (e.index == 1) {
              plus.runtime.openURL('bilibili://live/'+this.saveids[i])
            }else if (e.index == 2) {
              this.$set(this.liveids, i-1, 0)
            }
          })
        }else{
          let buttons = []
          buttons.push({title: "打开B站APP直播间"})
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
              if (e.index == 1) {
                plus.runtime.openURL('bilibili://live/'+this.liveids[i-1])
              }else if (e.index == 10) {
                this.$set(this.liveids, i-1, 0)
              }else{
                let idx = e.index - 2
                if (idx + 1 >= i) {
                  idx = e.index - 1
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
        
      }
    },
    saveItemClick(i) {
      if (!isNaN(this.layoutId)) {
        plus.nativeUI.actionSheet({
            title: '操作',
            buttons: [{title: "打开B站APP直播间"},{title: "添加到窗口#"+(this.layoutId+1)},{title: "关闭此窗口", style: "destructive"}],
            cancel: '取消'
          }, e => {
            if (e.index == 1) {
              plus.runtime.openURL('bilibili://live/'+this.saveids[i])
            }else if (e.index == 2) {
              while (this.layoutId > this.liveids.length) {
                this.liveids.push(0)
              }
              this.$set(this.liveids, this.layoutId, this.saveids[i])
              uni.setStorageSync("liveids", this.liveids.join(" "))
            }else if (e.index == 3) {
              this.saveids.splice(i, 1)
              uni.setStorageSync("saveids", this.saveids.join(" "))
            }
          })
      }else{
        let buttons = []
        buttons.push({title: '打开B站APP直播间'})
        // buttons.push({title: '复制直播id：'+this.saveids[i]})
        for (let j = 1; j <= 9; j++) {
          buttons.push({title: "添加到窗口"+j})
        }
        buttons.push({title: '从列表移除', style: "destructive"})
        plus.nativeUI.actionSheet({
          title: '操作',
          buttons: buttons,
          cancel: '取消'
        }, e => {
          console.log(e.index)
          if (e.index > 0) {
            if (e.index == 1) {
              // uni.setClipboardData({
              //   data: this.saveids[i],
              //   success() {
              //     uni.showToast({
              //       title: '已复制'
              //     })
              //   }
              // })
              plus.runtime.openURL('bilibili://live/'+this.saveids[i])
            }else if (e.index == 11) {
              this.saveids.splice(i, 1)
              uni.setStorageSync("saveids", this.saveids.join(" "))
            }else if (e.index > 1 && e.index < 11) {
              while (e.index - 2 > this.liveids.length) {
                this.liveids.push(0)
              }
              this.$set(this.liveids, e.index-2, this.saveids[i])
              // this.liveids[e.index-1] = liveid
              uni.setStorageSync("liveids", this.liveids.join(" "))
              
            }
            
          }
        })
      }
    }
  }
}
</script>

<style>
body {
  background-color: #31363b;
  color: white;
}
.content {
  display: flex;
  height: 100vh;
}
.scroll-list {
  flex: 1;
  overflow: scroll;
  border-top: 1px solid #808080;

}
.scroll-list:not(:last-child) {
  border-right: 1px solid #808080;
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
.up-item {
  display: flex;
  color: #a0a0a0;
}
.up-item.active {
  color: white;
}
.up-item .title {
  margin-left: 4px;
  width: calc(100% - 40px);
}
.up-item .title .text {
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}
.list-subtitle {
  font-size: 13px;
  border-bottom: 1px solid #808080;
  padding: 8px;
}
</style>