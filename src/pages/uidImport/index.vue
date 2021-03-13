<template>
  <view class="content">
    <view class="title">从你的uid获取公开的关注列表来选择导入UP主</view>
    <input type="number" v-model="uid" placeholder="你的uid" @confirm="submitUid" />
    <button @click="submitUid">查询</button>
    <!-- <view style="display:flex">
      <button @click="selectAll">全选</button>
      <view style="flex:1"></view>
      <button @click="finish">完成</button>
    </view> -->
    <view v-for="item in result" :key="item.room_id" :class="'list-item' + (selectedItems.indexOf(item.short_id > 0 ? item.short_id : item.room_id) >= 0 ? ' selected' : '')" @click="itemSelect(item.short_id > 0 ? item.short_id : item.room_id)">
      <view :class="'up-item' + (item.live_status == 1 ? ' active':'')">
          <img-cache lazy-load class="head-image" :src="item.face"></img-cache>
          <view class="title">
            <view class="text">{{item.live_status == 1 ? '直播中':'未开播'}} {{item.uname}}</view>
            <view class="text">{{item.title}}</view>
          </view>
        </view>
    </view>
    <view v-if="submited && result.length == 0" style="text-align:center;font-size:13px;color:#808080">
      无查询结果
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      uid: '',
      submited: false,
      result: [],
      selectedItems: []
    }
  },
  methods: {
    submitUid() {
      if (isNaN(parseInt(this.uid))) {
        uni.showToast({
          icon: 'none',
          title: '无效的uid'
        })
        return
      }

      uni.hideKeyboard()

      uni.hideLoading()
      this.submited = false
      this.result = []

      let that = this
      let mids = []

      let pageGetMids = function(pn) {
        uni.request({
          url: 'https://api.bilibili.com/x/relation/followings?vmid='+that.uid+'&pn='+pn+'&ps=50&order=desc&jsonp=jsonp',
          success(res) {
            try {
              const list = res.data.data.list
              if (list.length == 0)  {
                if (mids.length > 0) {
                  uni.request({
                    url: 'https://api.live.bilibili.com/room/v1/Room/get_status_info_by_uids',
                    method: 'POST',
                    data: JSON.stringify({uids:mids}),
                    success(res) {
                      try {
                        for (let mid in res.data.data) {
                          that.result.push(res.data.data[mid])
                        }
                      }catch{}
                      uni.hideLoading()
                      that.submited = true
                    }
                  })
                }else{
                  uni.hideLoading()
                  that.submited = true

                }
                return
              }
              list.forEach(e => {
                mids.push(e.mid)
              })
              pageGetMids(pn + 1)
            }catch{
              uni.hideLoading()
              that.submited = true
            }
          },
          fail() {
            uni.hideLoading()
            that.submited = true
          }
        })
      }

      uni.showLoading({
        title: '查询中'
      })
      pageGetMids(1)
      
    },
    itemSelect(roomid) {
      // console.log(roomid, this.selectedItems.has(roomid))
      const idx = this.selectedItems.indexOf(roomid)
      if (idx >= 0) {
        this.selectedItems.splice(idx, 1)
      }else{
        this.selectedItems.push(roomid)
      }
    },
    selectAll() {
      this.result.forEach(e => {
        if (this.selectedItems.indexOf(roomid) < 0) {
          this.selectedItems.push(e.room_id)
        }
      })
    },
    finish() {
      if (this.selectedItems.length == 0) {
        uni.showToast({
          icon: 'none',
          title: '请选择至少一项'
        })
        return
      }
      let saveids = uni.getStorageSync('saveids')
      if (saveids) {
        saveids = saveids.split(" ")
      }else{
        saveids = ["47377","8792912","21652717","213"]
      }
      console.log(this.selectedItems)
      // uni.setStorageSync('saveids', Array.from(new Set([...this.selectedItems, ...saveids])).join(" "))
      this.selectedItems.forEach(e => {
        if (saveids.indexOf(e.toString()) < 0) {
          saveids.push(e)
        }
      })
      uni.setStorageSync('saveids', saveids.join(" "))
      uni.navigateBack()
    }
  },
  onNavigationBarButtonTap() {
    this.finish()
  }
}
</script>

<style>
body {
  background-color: #31363b;
  color: white;
}
.title {
  font-size: 13px;
}
input {
  border: 1px solid #808080;
  padding: 8px;
}
button {
  background-color: #1b1e20;
  color: white;
  border: 1px solid #808080;
  font-size: 13px;
}
.list-item {
  padding: 8px;
  border-bottom: 1px solid #808080;
  font-size: 14px;
}
.list-item.selected {
  background-color: #1b1e20;
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
</style>