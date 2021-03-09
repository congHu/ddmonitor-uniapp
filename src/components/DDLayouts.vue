<template>
  <view class="container">
    <view v-if="layout == 0" class="layout layout0">
      <DDPlayer :liveid="getLiveId(0)" ref="dd0" />
    </view>
    <view v-if="layout == 1" class="layout layout1">
      <DDPlayer :liveid="getLiveId(0)" ref="dd0" />
      <DDPlayer :liveid="getLiveId(1)" ref="dd1" />
    </view>
    <view v-if="layout == 2" class="layout layout2">
      <DDPlayer :liveid="getLiveId(0)" ref="dd0" />
      <DDPlayer :liveid="getLiveId(1)" ref="dd1" />
    </view>
    <view v-if="layout == 3" class="layout layout3">
      <DDPlayer :liveid="getLiveId(0)" ref="dd0" />
      <DDPlayer :liveid="getLiveId(1)" ref="dd1" />
      <DDPlayer :liveid="getLiveId(2)" ref="dd2" />
    </view>
    <view v-if="layout == 4" class="layout layout4">
      <DDPlayer  :liveid="getLiveId(0)" ref="dd0" />
      <DDPlayer  :liveid="getLiveId(1)" ref="dd1" />
      <DDPlayer  :liveid="getLiveId(2)" ref="dd2" />
    </view>
    <view v-if="layout == 5" class="layout layout5">
      <view class="layout layout5-1">
        <DDPlayer  :liveid="getLiveId(0)"  ref="dd0"/>
      </view>
      <view class="layout layout5-2">
        <DDPlayer  :liveid="getLiveId(1)"  ref="dd1"/>
        <DDPlayer  :liveid="getLiveId(2)"  ref="dd2"/>
      </view>
    </view>
    <view v-if="layout == 6" class="layout layout6">
      <view class="layout layout6-1">
        <DDPlayer  :liveid="getLiveId(0)"  ref="dd0"/>
      </view>
      <view class="layout layout6-2">
        <DDPlayer  :liveid="getLiveId(1)"  ref="dd1"/>
        <DDPlayer  :liveid="getLiveId(2)"  ref="dd2"/>
      </view>
    </view>
    <view v-if="layout == 7" class="layout layout7">
      <view class="layout layout7-1">
        <DDPlayer  :liveid="getLiveId(0)"  ref="dd0"/>
        <DDPlayer  :liveid="getLiveId(1)"  ref="dd1"/>
      </view>
      <view class="layout layout7-1">
        <DDPlayer  :liveid="getLiveId(2)"  ref="dd2"/>
        <DDPlayer  :liveid="getLiveId(3)"  ref="dd3"/>
      </view>
    </view>
    <view v-if="layout == 8" class="layout layout8">
      <DDPlayer :liveid="getLiveId(0)"  ref="dd0"/>
      <DDPlayer :liveid="getLiveId(1)"  ref="dd1"/>
      <DDPlayer :liveid="getLiveId(2)"  ref="dd2"/>
      <DDPlayer :liveid="getLiveId(3)"  ref="dd3"/>
    </view>
    <view v-if="layout == 9" class="layout layout9">
      <view class="layout layout9-1">
        <DDPlayer  :liveid="getLiveId(0)"  ref="dd0"/>
      </view>
      <view class="layout layout9-2">
        <DDPlayer  :liveid="getLiveId(1)"  ref="dd1"/>
        <DDPlayer  :liveid="getLiveId(2)"  ref="dd2"/>
        <DDPlayer  :liveid="getLiveId(3)"  ref="dd3"/>
      </view>
    </view>
    <view v-if="layout == 10" class="layout layout10">
      <view class="layout layout10-1">
        <DDPlayer  :liveid="getLiveId(0)"  ref="dd0"/>
      </view>
      <view class="layout layout10-2">
        <DDPlayer  :liveid="getLiveId(1)" ref="dd1"/>
        <DDPlayer  :liveid="getLiveId(2)" ref="dd2" />
        <DDPlayer  :liveid="getLiveId(3)" ref="dd3" />
      </view>
    </view>
    <view v-if="layout == 11" class="layout layout11">
      <view class="layout layout11-1">
        <DDPlayer  :liveid="getLiveId(0)" ref="dd1" />
        <DDPlayer  :liveid="getLiveId(1)" ref="dd1" />
        <DDPlayer  :liveid="getLiveId(2)" ref="dd2" />
      </view>
      <view class="layout layout11-2">
        <DDPlayer  :liveid="getLiveId(3)" ref="dd3" />
        <DDPlayer  :liveid="getLiveId(4)" ref="dd4" />
        <DDPlayer  :liveid="getLiveId(5)" ref="dd5" />
      </view>
    </view>
    <view v-if="layout == 12" class="layout layout12">
      <view class="layout layout12-1">
        <view class="layout layout12-1-1">
          <DDPlayer  :liveid="getLiveId(0)" ref="dd0" />
        </view>
        <view class="layout layout12-1-2">
          <DDPlayer  :liveid="getLiveId(1)" ref="dd1" />
          <DDPlayer  :liveid="getLiveId(2)" ref="dd2" />
        </view>
      </view>
      <view class="layout layout12-2">
        <DDPlayer  :liveid="getLiveId(3)" ref="dd3" />
        <DDPlayer  :liveid="getLiveId(4)" ref="dd4" />
        <DDPlayer  :liveid="getLiveId(5)" ref="dd5" />
      </view>
    </view>
    <view v-if="layout == 13" class="layout layout13">
      <view class="layout layout13-1">
        <DDPlayer  :liveid="getLiveId(0)" ref="dd0" />
        <DDPlayer  :liveid="getLiveId(1)" ref="dd1" />
        <DDPlayer  :liveid="getLiveId(2)" ref="dd2" />
        <DDPlayer  :liveid="getLiveId(3)" ref="dd3" />
      </view>
      <view class="layout layout13-2">
        <DDPlayer  :liveid="getLiveId(4)" ref="dd4" />
        <DDPlayer  :liveid="getLiveId(5)" ref="dd5" />
        <DDPlayer  :liveid="getLiveId(6)" ref="dd6" />
        <DDPlayer  :liveid="getLiveId(7)" ref="dd7" />
      </view>
    </view>
    <view v-if="layout == 14" class="layout layout14">
      <view class="layout layout14-1">
        <view class="layout layout14-1-1">
          <DDPlayer  :liveid="getLiveId(0)" ref="dd0" />
        </view>
        <view class="layout layout14-1-2">
          <DDPlayer  :liveid="getLiveId(1)" ref="dd1" />
          <DDPlayer  :liveid="getLiveId(2)" ref="dd2" />
          <DDPlayer  :liveid="getLiveId(3)" ref="dd3" />
        </view>
      </view>
      <view class="layout layout14-2">
        <DDPlayer  :liveid="getLiveId(4)" ref="dd4" />
        <DDPlayer  :liveid="getLiveId(5)" ref="dd5" />
        <DDPlayer  :liveid="getLiveId(6)" ref="dd6" />
        <DDPlayer  :liveid="getLiveId(7)" ref="dd7" />
      </view>
    </view>
    <view v-if="layout == 15" class="layout layout15">
      <view class="layout layout15-1">
        <DDPlayer  :liveid="getLiveId(0)" ref="dd0" />
        <DDPlayer  :liveid="getLiveId(1)" ref="dd1" />
        <DDPlayer  :liveid="getLiveId(2)" ref="dd2" />
      </view>
      <view class="layout layout15-2">
        <DDPlayer  :liveid="getLiveId(3)" ref="dd3" />
        <DDPlayer  :liveid="getLiveId(4)" ref="dd4" />
        <DDPlayer  :liveid="getLiveId(5)" ref="dd5" />
        
      </view>
      <view class="layout layout15-3">
        <DDPlayer  :liveid="getLiveId(6)" ref="dd6" />
        <DDPlayer  :liveid="getLiveId(7)" ref="dd7" />
        <DDPlayer  :liveid="getLiveId(8)" ref="dd8" />
      </view>
    </view>
  </view>
</template>

<script>
import DDPlayer from './DDPlayer.vue'
export default {
  components: { DDPlayer },
  data() {
    return {
      
    }
  },
  props: {
    layout: {
      type: Number,
      default: 0
    },
    liveids: {
      type: Array, 
      default: []
    }
  },
  methods: {
    getLiveId(index) {
      if (index < this.liveids.length) {
        return this.liveids[index]
      }else{
        return "0"
      }
    },
    refreshAll() {
      console.log("dd refresh")
      for (let i=0; i < 9; i++) {
        if (this.$refs['dd'+i]) {
          this.$refs['dd'+i].loadUrl()
        }
      }
    },
    muteAll() {
      for (let i=0; i < 9; i++) {
        if (this.$refs['dd'+i]) {
          this.$refs['dd'+i].doMute()
        }
      }
    },
    danmuCloseAll() {
      for (let i=0; i < 9; i++) {
        if (this.$refs['dd'+i]) {
          this.$refs['dd'+i].danmuClose()
        }
      }
    },
    keepPlayAll() {
      for (let i=0; i < 9; i++) {
        if (this.$refs['dd'+i]) {
          this.$refs['dd'+i].keepPlay()
        }
      }
    },
    pauseAll() {
      for (let i=0; i < 9; i++) {
        if (this.$refs['dd'+i]) {
          this.$refs['dd'+i].pause()
        }
      }
    }
  }
}
</script>

<style>
.container {
  /* height: 100%; */
  flex: 1;
  display: flex;
  flex-direction: column;
}
.layout {
  /* height: 100%; */
  flex: 1;
  display: flex;
}
.layout2,.layout3,.layout5-2,.layout6,.layout7,.layout8,.layout9-2,.layout10,.layout11,.layout12,.layout12-1-2,.layout13,.layout14,.layout14-1-2,.layout15 {
  flex-direction: column;
}
.layout5-2,.layout6-2,.layout10-2,.layout14-1-2,.layout14-2 {
  flex: 1
}
.layout5-1,.layout6-1,.layout10-1,.layout12-1-1 {
  flex: 2;
}
.layout9-1,.layout14-1,.layout14-1,.layout14-1-1 {
  flex: 3;
}
</style>