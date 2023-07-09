<template>
  <div v-loading="isloading" class="mainbox">
    <div class="topinfo">
      <el-button @click="showinfo = !showinfo" size="small" class="btn">
        <span v-show="showinfo">展开</span>
        <span v-show="!showinfo">折叠</span> </el-button
      ><br />
      <span v-show="showinfo"
        >ID：{{ lastID }} <br />
        连接数：{{ conncount }}</span
      >
    </div>
    <div class="info" v-show="!showinfo">
      <br><br>
      <span>ID:{{ lastID }}</span
      >&nbsp;&nbsp;
      <span>连接数：{{ conncount }}</span>
      <br /><br />
      <el-form size="small" inline label-width="50">
        <el-form-item label="连接ID" >
          <el-input class="input" v-model="targID"> </el-input>
        </el-form-item>
        <el-form-item class="btnitem">
          <el-button @click="doconn()">连接</el-button> </el-form-item
        ><br />
        <el-form-item label="昵称" v-show="shownameset">
          <el-input class="input" v-model="name"></el-input>
        </el-form-item>
        <el-form-item v-show="shownameset" class="btnitem">
          <el-button @click="shownameset = false"
            >保存</el-button
          > </el-form-item
        ><br v-show="shownameset" />
        <el-form-item label="信息">
          <el-input class="input" v-model="message"></el-input>
        </el-form-item>
        <el-form-item class="btnitem">
          <el-button @click="btnSendHandler()">发送</el-button> </el-form-item
        ><br />
      </el-form>
    </div>
    <!-- <el-button @click="test()">test</el-button> -->
    <div class="msglist">
      <div class="msgitem" v-for="(item, index) in msglist" :key="index">
        <span class="name"
          >{{ item.name == "" ? item.source : item.name }}：</span
        >
        <span class="msg">{{ item.msg }}</span>
      </div>
      <br /><br /><br /><br /><br /><br /><br />
    </div>
    <div class="bottom" v-show="showinfo">
      <el-input class="bottominput" v-model="message"  @keyup.enter="btnSendHandler()" ></el-input>
      <el-button class="bottombtn" @click="btnSendHandler()">发送</el-button>
    </div>
  </div>
</template>

<script setup>
import Peer from "peerjs";
import { ref, computed, reactive } from "vue";
import { ElMessage } from "element-plus";
//状态
let isloading = ref(true);
let shownameset = ref(true);
let showinfo = ref(false);
// 属性
let baseID = ref("23d932a7-cfb4-4e5e-b7b2-077987136fbf");
let lastID = ref(getuuid());
let ruleID = computed(() => {
  return baseID.value + lastID.value;
});
let name = ref("");
let targID = ref("");
let conncount = ref(0);
let connlist = new Map();
let connidlist = () => {
  conncount.value = connlist.size;
  let idlist = [];
  connlist.forEach((conn) => {
    idlist.push(conn.peer.substr(-4));
  });
  return idlist;
};
let message = ref("");
let msglist = ref([]);
let perid = ref("");
let peer = new Peer(ruleID.value);
// 初始化 分配ID
peer.on("open", (id) => {
  console.log("Peer连接ID: " + id);
  isloading.value = false;
});
// 连接出错
peer.on("error", (e) => {
  console.log("on error: ", e);
});
// 主动连接
function doconn(targ = targID.value.trim()) {
  if (connidlist().includes(targ)) {
    return;
  }
  if (targ == lastID.value) {
    return;
  }
  let conn = peer.connect(baseID.value + targ);
  conn.on("open", () => {
    console.log("我主动连接了别人", conn.peer.substr(-4));
    connlist.set(conn.peer.substr(-4), conn);
    sendConnidlist();
  });
  conn.on("data", (data) => {
    msgHandler(data);
  });
}
//点击发送按钮
function btnSendHandler() {
  if (message.value == "") return;
  scrollListEnd();
  sendBroad({
    name: name.value,
    source: lastID.value,
    type: "msg",
    msg: message.value,
  });
  msglist.value.push({
    name: name.value,
    source: lastID.value,
    type: "msg",
    msg: message.value,
  });
  message.value = "";
}

// 被连接
peer.on("connection", (conn) => {
  console.log("有人连接了我", conn.peer.substr(-4));
  conn.on("data", (data) => {
    msgHandler(data);
  });
  connlist.set(conn.peer.substr(-4), conn);
  sendConnidlist();
});

// 发送广播
function sendBroad(msg) {
  msg = {
    type: "广播",
    data: msg,
  };
  connlist.forEach((conn) => {
    conn.send(JSON.stringify(msg));
  });
}
// 发送私聊
function sendPeer(conn, msg) {
  msg = {
    type: "私聊",
    data: msg,
  };
  conn.send(msg);
}

// 广播发送连接id列表
function sendConnidlist() {
  let ids = connidlist();
  sendBroad({
    name: name,
    source: lastID,
    type: "idlist",
    idlist: ids,
  });
}
// 处理消息
function msgHandler(msg) {
  msg = JSON.parse(msg);
  scrollListEnd();
  if (msg.type == "私聊") {
    return;
  }
  switch (msg.data.type) {
    case "idlist":
      conflist(msg.data.idlist);
      break;
    case "msg":
      console.log(msg.data);
      msglist.value.push(msg.data);
      break;
  }
}

// 通过idlist遍历连接多个peer
function conflist(idlist) {
  idlist.forEach((id) => {
    doconn(id);
  });
}
// 消息始终显示底部
function scrollListEnd() {
  let listend = document.getElementsByClassName("msglist")[0];
  listend.scrollTop = listend.scrollHeight;
}
// 回车发送
window.addEventListener("keydown", (event) => {
  if (event.key == "Enter") {
    btnSendHandler();
  }
});
function showMessage(msg, type = "info") {
  ElMessage({
    showClose: true,
    message: msg,
    type: type,
  });
}

function test() {
  showMessage("test");
}

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
<style scoped>
.mainbox {
  color: rgba(170, 170, 170, 0.933);
  display: flex;
  flex-flow: column;
  height: 100vh;
  overflow: hidden;
}
.input {
  width: 100px;
}
.info {
}
.msglist {
  padding: 10px;
  flex: 1;
  text-align: left;
  overflow: auto;
}
.bottom {
  display: inline-block;
  height: 40px;
  /* background: red; */
  position: fixed;
  bottom: 0px;
  left: -1px;
  width: 100vw;
  margin: 0px auto;
}
.bottom .bottominput {
  margin: 0px 2px;
  width: 80vw;
  background: #141414;
}
.bottom .bottombtn {
  background: #141414;
}
.topinfo {
  position: fixed;
  width: 100px;
  right: 0px;
  top: 0px;
  text-align: right;
  padding: 10px;
}
.topinfo .btn {
  background: #141414;
}
.btnitem{
  
}
</style>
