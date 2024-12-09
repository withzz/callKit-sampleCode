<script setup>
import { ref } from 'vue'
import NIM from "@yxim/nim-web-sdk/dist/SDK/NIM_Web_NIM"
import { NECall } from '@xkit-yx/call-kit'

const neCall = NECall.getInstance()
window.neCall = neCall;
const appkey = ref('')
const accId = ref('')
const token = ref('')
const callee = ref('')
const message = ref('')
const switchViewFlag = ref(1)
const enableVideoFlag = ref(1)
const enableTypeFlag = ref(1)
const enableAudoFlag = ref(1)

const login = () => {
  const im = NIM.getInstance({
    appKey: appkey.value, // im appkey
    token: token.value, // im token
    account: accId.value, // im account
    logLevel: 'debug',
    onconnect: () => {
      neCall.setup({
        nim: im, // im 实例用户需要先初始化
        appkey: appkey.value, // 应用的 appKey，G2 平台的 appKey
        debug: true, // 是否需要开启日志，默认开启
      })
      // 来电监听
      neCall.on('onReceiveInvited', (value) => {
        // 用户可以触发视图，比如弹起邀请页面
        console.log(value)
        message.value = `收到 ${value.callerAccId} 呼叫，可以点击接听按钮了`

      })
      // 通话结束的回调。通话被取消、拒绝、通话中挂断，即所有导致通话结束的行为都会触发此回调
      neCall.on('onCallEnd', (value) => {
        console.log(value)
        message.value = ''
      })
      // 话单回调
      neCall.on('onRecordSend', (value) => {
        console.log(value)
      })
      // 主被叫加入通道的回调，通话建立后触发
      neCall.on('onCallConnected', (value) => {
        console.log('onCallConnected: ', value)
      })
      // 通话方式切换
      neCall.on('onCallTypeChange', (NECallTypeChangeInfo) => {
        console.log("onCallTypeChange: ", NECallTypeChangeInfo)
        if (NECallTypeChangeInfo.callType === "1") {
          NECall.getInstance().enableLocalVideo(false)
          enableTypeflag.value = 0
        } else {
          NECall.getInstance().enableLocalVideo(true)
          enableTypeflag.value = 1
        }
      })
    },
    ondisconnect: () => {
      neCall?.destroy()
    },
  })
}

const call = async () => {
  // 发起呼叫
  const localView = document.getElementById('NE_local')
  const remoteView = document.getElementById('NE_remote')
  neCall.setLocalView(localView)
  neCall.setRemoteView(remoteView, callee.value)
  neCall.setTimeout(30)//设置呼叫超时取消时间，单位：秒
  neCall.call({
    accId: callee.value, // 被叫 im 的 id
    callType: '2', //呼叫类型，1表示语音通话，2表示视频通话 
  })
}

const switchView = () => {
  neCall.rtcController.localStream.stop('video')
  neCall.rtcController.remoteStreams[0].stop('video')
  if (switchViewFlag.value) {
    const localView = document.getElementById('NE_local')
    const remoteView = document.getElementById('NE_remote')
    neCall.rtcController.localStream.play(remoteView)
    neCall.rtcController.remoteStreams[0].play(localView)

    neCall.setLocalView(remoteView)
    neCall.setRemoteView(localView)
    switchViewFlag.value = 0
  } else {
    const localView = document.getElementById('NE_local')
    const remoteView = document.getElementById('NE_remote')
    neCall.rtcController.localStream.play(localView)
    neCall.rtcController.remoteStreams[0].play(remoteView)
    neCall.setLocalView(localView)
    neCall.setRemoteView(remoteView)
    switchViewFlag.value = 1
  }
}

const accept = () => {
  // 接受邀请
  const localView = document.getElementById('NE_local')
  const remoteView = document.getElementById('NE_remote')
  neCall.setLocalView(localView)
  neCall.setRemoteView(remoteView, callee.value)
  // 接受邀请，需要先设置视图
  neCall.accept()
  message.value = ''
}

const hangup = () => {
  // 挂断
  neCall.hangup()
  message.value = ''
}

const switchCallType = () => {
  if (enableTypeFlag.value) {
    NECall.getInstance().switchCallType({ callType: 1, state: 1 })
    NECall.getInstance().enableLocalVideo(false)
    enableTypeFlag.value = 0
  } else {
    NECall.getInstance().switchCallType({ callType: 2, state: 1 })
    NECall.getInstance().enableLocalVideo(true)
    enableTypeFlag.value = 1
  }
}

const enableLocalVideo = () => {
  if (enableVideoFlag.value) {
    NECall.getInstance().muteLocalVideo(true)
    enableVideoFlag.value = 0
  } else {
    NECall.getInstance().muteLocalVideo(false)
    enableVideoFlag.value = 1
  }
}

const enableLocalAudio = () => {
  if (enableAudoFlag.value) {
    NECall.getInstance().muteLocalAudio(true)
    enableAudoFlag.value = 0
  } else {
    NECall.getInstance().muteLocalAudio(false)
    enableAudoFlag.value = 1
  }
}
</script>

<template>
  <div>
    <p style="color: red;">{{ message }}</p>
    呼叫组件简易demo，使用步骤如下：
    1、输入appkey等账号信息
    2、后点击登录
    3、呼叫对方或者收到对方呼叫之后点击接听
    <div>
      <button @click="login">登录</button>
      <button @click="call">呼叫</button>
      <button @click="hangup">取消</button>
      <button @click="accept">接听</button>
      <button @click="hangup">挂断</button>
      <button @click="switchView">切换视图</button>
      <button @click="switchCallType">切换通话类型</button>
      <button @click="enableLocalVideo">开关摄像头</button>
      <button @click="enableLocalAudio">开关麦克风</button>
    </div>
    <div>
      appkey:
      <input type="text" v-model="appkey" />
      accId:
      <input type="text" v-model="accId" />
      token:
      <input type="text" v-model="token" />
      callee:
      <input type="text" v-model="callee" />
    </div>
    <div id="NE_local" style="width:320px; height:240px; background-color:red;">
    </div>
    <div id="NE_remote" style="width:320px; height:240px; background-color:green;">
    </div>
  </div>
</template>