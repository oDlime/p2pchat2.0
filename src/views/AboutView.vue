<template>
  <div>
    <div class="box">
      <span v-show="isiniting">加载中</span>
      <div v-show="!isiniting">
        <span>ID: {{ rid }}</span> <br /><br />
        <el-form size="small" inline label-width="60" style="width: 90%;">
          <el-form-item label="对方ID：">
            <el-input v-model="tarid" class="miniinp"></el-input>
          </el-form-item>
          <el-form-item>
            <el-button @click="doconn()">连接</el-button>
          </el-form-item>
          <br /><br />
          <el-form-item label="信息" v-show="connid">
            <el-input v-model="message" class="miniinp"></el-input>
          </el-form-item>
          <el-form-item v-show="connid">
            <el-button @click="dosend()">发送</el-button>
          </el-form-item>
        </el-form>
        <br /><br />
        连接状态：{{ connstate }}<br />
        <span v-show="connid">目标：{{ connid }}</span>
      </div>
    </div>
    <div class="messlistbox">
      <div class="messitem" v-for="(value,index) in messlist" :key="index">
        <span class="name">{{ value.name }}</span>:
        <span class="mess">{{ value.message }}</span>
      </div>
    </div>
    <!-- <el-button @click="test()">test</el-button> -->
  </div>
</template>

<script setup>
import { ref } from "vue";
import Peer from "peerjs";
// 状态
let isiniting = ref(true);
let connstate = ref("未连接");

//属性
let baseID = "23d932a7-cfb4-4e5e-b7b2-077987136fbf";
let rid = getuuid();
let ruleid = baseID+rid;
let myid = ref("");
let tarid = ref("");
let connid = ref("");
let message = ref("");
let tarconn;
let messlist = ref([]);
const peer = new Peer(ruleid);
peer.on("open", (id) => {
  myid.value = id;
  isiniting.value = false;
});
function doconn() {
  let conn = peer.connect(baseID+ tarid.value);
  connstate.value = "正在连接";
  conn.on("open", () => {
    connstate.value = "已连接";
    connid.value = conn.peer;
    tarconn = conn;
  });
  conn.on("data", (data) => {
    messlist.value.unshift({
      name: "对方",
      message: data,
    });
  });
}
function dosend() {
  if(!message.value) return;
  tarconn.send(message.value);
  messlist.value.unshift({
    name: "我",
    message: message.value,
  });
  message.value = ""
}
peer.on("connection", (conn) => {
  conn.on("data", (data) => {
    messlist.value.unshift({
      name: "对方",
      message: data,
    });
  });
  conn.on("open", () => {
    connstate.value = "已连接";
    connid.value = conn.peer;
    tarconn = conn;
  });
});
window.addEventListener('keydown', (event) => {
  if(event.key=="Enter"){
    if(!connid.value) return;
    dosend();
  }
});
function getuuid() {
    var d = new Date().getTime();
    if (window.performance && typeof window.performance.now === "function") {
        d += performance.now();
    }
    var uuid = 'xxxx'.replace(/[xy]/g, function (c) {
        var r = (d + Math.random() * 16) % 16 | 0; 
        d = Math.floor(d / 16);
        return (c == 'x' ? r : (r & 0x3 | 0x8)).toString(16);
    });
    return uuid;
}
function test(){
  console.log(getuuid());
}

</script>

<style scoped>
.box {
  border: 1px solid #292929;
  padding: 10px;
  width: 90%;
  margin: 0px auto;
  color: #979797;
}
.messlistbox {
  color: #979797;
}
.messlistbox {
  text-align: left;
  width: 90%;
  margin: 0px auto;
  margin-top: 30px;
}
.messlistbox .messitem {
  margin-top: 10px;
}
.messlistbox .messitem .name {
  color: #c0bebe;
}
.miniinp{
  /* width: 120px; */
}
/* .messlistbox .messitem .mess {} */
</style>
