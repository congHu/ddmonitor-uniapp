<template>
  <view class="content">
    
  </view>
</template>

<script>
export default {

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