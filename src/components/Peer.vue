<template>
  <div class="row">localId:{{ localId }}</div>
  <div class="row">
    <input type="text" v-model="remoteId" />
    <button @click="connection">建立链接</button>
    <button @click="sendVideo">传输视频</button>
  </div>
  <div class="row">
    <input type="text" v-model="msg" />
    <button @click="sendMsg">发送消息</button>
  </div>
  <div class="row">
    <div>
      <span>本地视频</span>
      <video ref="video1" autoplay playsinline muted></video>
    </div>
    <div>
      <span>远程视频</span>
      <video ref="video2" autoplay playsinline muted></video>
    </div>
  </div>
  <div class="message">
    <div class="title">消息</div>
    <div v-for="(item, index) in messages" :key="index">{{ item }}</div>
  </div>
</template>
<script lang="ts" setup>
import Peer from "peerjs";

import { onMounted, ref } from "vue";

let peer = ref<Peer>();
const video1 = ref();
const video2 = ref();
// 本地id
const localId = ref("");
// 远程Id
const remoteId = ref("");

const remoteConn = ref();
const msg = ref();

const messages = ref<string[]>([]);

const getUserMedia = (constraints: MediaStreamConstraints) => {
  return navigator.mediaDevices.getUserMedia(constraints);
};

// 建立链接
function connection() {
  remoteConn.value = peer.value?.connect(remoteId.value);
  remoteConn.value?.on("open", () => {
    console.log("链接建立成功");
  });
}

// 推送本地视频给远端
async function getLocalStream(constraints: MediaStreamConstraints) {
  const stream = await getUserMedia(constraints);

  const call = peer.value?.call(remoteId.value, stream);

  video1.value.srcObject = stream;

  call?.on("stream", (remoteStream) => {
    video2.value.srcObject = remoteStream;
  });
}

// 远端

// 发送消息
const sendMsg = () => {
  if (!msg.value) return;
  remoteConn.value.send(msg.value);
};

// 发送视频
const sendVideo = () => {
  getLocalStream({
    video: true,
    audio: false,
  });
};

const initPeer = () => {
  peer.value = new Peer();
  peer.value.on("open", (id: string) => {
    localId.value = id;
  });

  peer.value.on("connection", (conn) => {
    conn.on("data", (data) => {
      console.log("data", data);
      messages.value.push(data as string);
    });
  });

  peer.value.on("call", async (call) => {
    const stream = await getUserMedia({
      video: true,
      audio: false,
    });
    video1.value.srcObject = stream;
    call.answer(stream);
    call.on("stream", (remoteStream) => {
      video2.value.srcObject = remoteStream;
    });
  });
};

onMounted(() => {
  initPeer();
  document.addEventListener(
    "WeixinJSBridgeReady",
    function () {
      (document.getElementById("remoteVideo") as any).play();
    },
    false
  );
});
</script>

<style lang="scss" scoped>
.message {
  border: 1px solid gainsboro;
  text-align: left;
  padding: 10px;
  .title {
    padding-bottom: 10px;
    border-bottom: 1px solid gainsboro;
  }
}
.row {
  display: flex;
}
video {
  width: 500px;
  height: 500px;
}
</style>
