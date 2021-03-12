<template>
 <!-- :style="'margin-top:'+safeAreaInsetsTop+'px'" -->
  <view class="content">
    <!-- <view class="topbar">
      <view class="btn" @click="goback">取消</view>
      <view class="title">更改布局方式</view>
    </view> -->
    <!-- <view class="scrollview"> -->
      <!--  -->
    <view v-for="i in 16" :key="i-1" class="layout-item" :style="'background-position: '+(((i-1)%4)*33.33)+'% '+(Math.floor((i-1)/4)*33.33)+'%'" @click="selectLayout(i-1)">
    </view>
    <view v-for="i in 3" :key="i+15" class="layout-item layout-ex1" :style="'background-position: '+(((i-1)%3)*50)+'% 0'" @click="selectLayout(i+15)">
    </view>
    <!-- </view> -->
    
  </view>
</template>

<script>
export default {
  data() {
    return {
      safeAreaInsetsTop: 0
    }
  },
  onShow() {
    const info = uni.getSystemInfoSync()
    this.safeAreaInsetsTop = info.safeAreaInsets.top
    // this.safeHeight = info.safeArea.height
  },
  methods: {
    goback() {
      uni.navigateBack()
    },
    selectLayout(layout) {
      console.log("layout", layout)
      uni.setStorage({
        key: 'layout',
        data: layout,
        success() {
          uni.navigateBack()
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
.layout-item {
  width: 25%;
  padding-bottom: 15%;
  background-image: url('/static/layouts.png');
  background-size: 400%;
  /* background-position: 33.33% 33.33%; */
  float: left;
}
.layout-ex1 {
  background-image: url('/static/layout_ex1.png');
  background-size: 300%;

}
</style>