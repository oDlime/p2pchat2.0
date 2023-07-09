<template>
  <div v-loading="loading" class="box">
    {{ lastID }}
    <el-form inline label-width="80px">
      <el-form-item label="接收者">
        <el-input v-model="targID"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="doCall()">视频通话</el-button>
      </el-form-item>
    </el-form>


    <video id="videoElement" autoplay></video>
  </div>
</template>

<script setup>
import {computed , ref} from "vue";
import Peer from "peerjs";
// 状态
let loading = ref(true);
//属性
let baseID = ref("23d932a7-cfb4-4e5e-b7b2-077987136fbf");
let lastID = ref(getuuid());
let ruleID = computed(() => {
  return baseID.value + lastID.value;
});
let targID = ref("");
let peer = new Peer(ruleID.value);
peer.on('open',(id)=>{
  console.log("我的ID："+id);
  loading.value=false;
})
function doCall(){
  var video = document.getElementById('videoElement');
  // 获取视频流
  if (navigator.mediaDevices.getUserMedia) {
    navigator.mediaDevices.getUserMedia({ video: true ,audio: true})
        .then(function(stream) {
          var call = peer.call(baseID.value+targID.value, stream);
          console.log("我要连:"+targID.value)
          call.on('stream', function(remoteStream) {
            video.srcObject = remoteStream;
            console.log("我准备好了")
          });
        })
        .catch(function(error) {
          console.log('无法获取视频流: ', error);
        });
  } else {
    console.log('浏览器不支持getUserMedia');
  }
}
var getUserMedia = navigator.getUserMedia || navigator.webkitGetUserMedia || navigator.mozGetUserMedia;
peer.on('call', function(call) {
  var video = document.getElementById('videoElement');
  // 获取视频流
  if (navigator.mediaDevices.getUserMedia) {
    navigator.mediaDevices.getUserMedia({ video: true,audio: true })
        .then(function(stream) {
          call.answer(stream); // Answer the call with an A/V stream.
          call.on('stream', function(remoteStream) {
            video.srcObject = remoteStream;
            console.log("我也准备好了")
          });
        })
        .catch(function(error) {
          console.log('无法获取视频流: ', error);
        });
  } else {
    console.log('浏览器不支持getUserMedia');
  }
});
function getuuid(m = "xxxx") {
  var d = new Date().getTime();
  if (window.performance && typeof window.performance.now === "function") {
    d += performance.now();
  }
  var uuid = m.replace(/[xy]/g, function (c) {
    var r = (d + Math.random() * 16) % 16 | 0;
    d = Math.floor(d / 16);
    return (c == "x" ? r : (r & 0x3) | 0x8).toString(16);
  });
  return uuid;
}
</script>

<style>
#videoElement{
  width: 100vw;
  height: 100vh;
}
.box{
  color: #999999;
}
</style>