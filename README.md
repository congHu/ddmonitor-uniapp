# ddmonitor-uniapp

## 说明

- 第一版就这么先出炉了。
- 使用时请注意宽带网速、流量消耗、电池电量、机身发热、系统卡顿等软硬件环境问题。
- 具备最基本的观看直播、接收弹幕功能，支持与PC版一样16种布局切换。
- 与PC版一样也**不会支持**账号登录获取信息、发送弹幕、发送礼物、互动打赏等功能。
- 支持横屏锁定切换。
- uni-app的video在APP端使用原生组件，视频使用硬解码，保证了流畅度，但是目前版本拖放功能较短时间内难以实现，因此采用"UP列表"二级页面列表进行切换。
- 没有（或者说暂未发现）对单独视频音轨的音量进行设置的API。目前只支持静音切换。

## 介绍

1. 在首页右上角点击"UP列表"进入列表界面，上方是窗口列表，下方是"你的列表"。
2. 在下方"你的列表"中点击添加，输入直播间id，添加UP主到"你的列表"当中。
3. 点击"你的列表"中的UP主，添加到指定窗口中。编辑完成后，返回首页，等待刷新即可。
4. 点击"窗口X"中的UP主，可以切换到其他窗口，或者关闭当前窗口。
5. 在首页右上角点击"更改布局"，选择布局并切换。

## TODO&BUG

- [ ] 图标按钮需要更换
- [ ] "UP列表"页面布局需要改进
- [ ] 弹幕在解析时出现了一些问题，导致不能获取所有弹幕。
- [ ] 弹幕位置的切换
- [ ] 从uid公开关注列表选择导入UP主
- [ ] iPhone全面屏未做测试

## 考虑是否需要的功能

- 直播缓存录像保存。（占用大量存储空间）
- 清晰度切换。（加重带宽网速负担，而且小屏幕非必要）


## 源码运行

使用HBuilderX，详见[https://uniapp.dcloud.io/quickstart?id=运行uni-app](https://uniapp.dcloud.io/quickstart?id=运行uni-app)

需要注意的是，本工程由vue-cli生成，在HBuilderX运行之前可能需要先安装好依赖库

```
npm install
```
或者
```
yarn
```

另外，代码中用到了`plus`相关API，因此该工程只能运行为手机APP正常使用。其他平台（H5、小程序）无法正常使用。

## 相关链接
- [https://github.com/zhimingshenjun/DD_Monitor](https://github.com/zhimingshenjun/DD_Monitor)
- [https://github.com/lovelyyoshino/Bilibili-Live-API](https://github.com/lovelyyoshino/Bilibili-Live-API)
