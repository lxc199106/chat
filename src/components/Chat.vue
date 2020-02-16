<template>
  <div>
    <div class="container">
      <div class="chat-container">
        <div class="chat-container-top" ref="chatMessage">
          <div class="chat-container-top-message" v-for="(value,index) in message_list" :key='index'>
            <div
              :class="value.send_user.username == user.username?'right-container chat-container-text-name' : 'left-container chat-container-text-name'">
              {{value.send_user.cname}}</div>
            <div
              :class="value.send_user.username==user.username?'right-container right-border-r chat-container-text-message':'left-container left-border-r chat-container-text-message'">
              <div v-show="value.message_type=='chat'">{{value.message}}</div>
              <el-image v-show="value.message_type=='expression'" style="width:120px;height:120px" :src="value.message">
              </el-image>
            </div>
          </div>
        </div>

        <el-progress :percentage="percentage" :format="format" :stroke-width="3" color="rgba(172, 23, 185, 0.6)">
        </el-progress>

        <div class="chat-container-bottom">
          <el-input type="textarea" :rows="2" placeholder="请输入消息内容" v-model="message"> </el-input>
          <div class="icon-container">
            <img src="../assets/icon_icon.png" alt="" style="width:100%;height:100%" @click="showExpression()">
          </div>
          <el-button type="success" @click="sendMessage">发送</el-button>
        </div>

      </div>
    </div>
    <el-dialog :visible.sync="dialogVisible" width="50%" :showClose="false">
      <!-- tree start -->
      <div class="expression-icon-container">
        <div class="expression-icon-container-img expression-img" v-for="(val,index) in expression_icon_list"
          :key="index">

          <el-image style="width: 100%; height: 100%;border-radius: 5px;" :src="val.url" :fit="fit"
            @click="checkExpression(val.url)"></el-image>
        </div>
      </div>
    </el-dialog>
  </div>
</template>
<script>
import md5 from 'js-md5'
let base64 = require('js-base64').Base64;
export default {
  created () {
    this.getStorage();
  },
  data () {
    return {
      message: '',
      user: [],
      ws: [],
      message_list: [],
      dialogVisible: false,
      fit: 'fit',
      percentage: 0,
      progress: false,
      timer: [],
      expression_icon_list: [
        { 'url': require("../assets/icon/chat_icon_1.jpg") },
        { 'url': require("../assets/icon/chat_icon_2.jpg") },
        { 'url': require("../assets/icon/chat_icon_3.jpg") },
        { 'url': require("../assets/icon/chat_icon_4.jpg") },
        { 'url': require("../assets/icon/chat_icon_5.jpg") },
        { 'url': require("../assets/icon/chat_icon_6.jpg") },
        { 'url': require("../assets/icon/chat_icon_7.jpg") },
        { 'url': require("../assets/icon/chat_icon_8.jpg") },
        { 'url': require("../assets/icon/chat_icon_9.jpg") },
        { 'url': require("../assets/icon/chat_icon_10.jpg") },
        { 'url': require("../assets/icon/chat_icon_11.jpg") },
        { 'url': require("../assets/icon/chat_icon_12.jpg") },
        { 'url': require("../assets/icon/chat_icon_13.jpg") },
        { 'url': require("../assets/icon/chat_icon_14.jpg") },
        { 'url': require("../assets/icon/chat_icon_15.jpg") },
        { 'url': require("../assets/icon/chat_icon_16.jpg") },
        { 'url': require("../assets/icon/chat_icon_17.jpg") },
        { 'url': require("../assets/icon/chat_icon_18.jpg") },
        { 'url': require("../assets/icon/chat_icon_19.jpg") },
        { 'url': require("../assets/icon/chat_icon_20.jpg") },
        { 'url': require("../assets/icon/chat_icon_21.jpg") },
        { 'url': require("../assets/icon/chat_icon_22.jpg") },
        { 'url': require("../assets/icon/chat_icon_23.jpg") },
        { 'url': require("../assets/icon/chat_icon_24.jpg") },
        { 'url': require("../assets/icon/chat_icon_25.jpg") },
        { 'url': require("../assets/icon/chat_icon_26.jpg") },
        { 'url': require("../assets/icon/chat_icon_27.jpg") }
      ]
    }
  },
  methods: {
    open () {
      this.$prompt('请给自己取一个响亮又大气的名字(必须是中文！！)', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        showCancelButton: false,
        showClose: false,
        closeOnClickModal: false,
        closeOnPressEscape: false,
        inputPattern: /^[\u4e00-\u9fa5]{2,4}$/,
        inputErrorMessage: '名字格式不正确'
      }).then(({ value }) => {
        let num = Math.ceil(Math.random() * 10000);
        let username = md5(`${value}${num}`);
        let cname = value;
        this.user = { 'username': username, 'cname': cname };
        window.localStorage.setItem('__USER__', base64.encode(JSON.stringify(this.user)));
        this.openWebsocket();
      })
    },
    //建立websocket链接
    openWebsocket () {
      if ("WebSocket" in window) {
        // 打开一个 web socket
        this.ws = new WebSocket("ws://47.104.128.87:9998");

        this.ws.onopen = () => {
          // Web Socket 已连接上，使用 send() 方法发送数据
          /** 
           * 初次链接握手 绑定uid
          */
          let userInit = JSON.stringify({ 'type': 'init', 'username': this.user.username, 'cname': this.user.cname });
          this.ws.send(userInit);
          this.$message({
            type: 'success',
            message: '连接服务器成功！'
          })
        };

        this.ws.onmessage = (e) => {
          let data = e.data;
          let message = JSON.parse(data);
          let messageStr = "";
          switch (message.type) {
            case 'message':
              if (this.percentage == 10) {
                this.percentage = 60;
              }
              this.message_list.push(message);
              if (this.percentage == 60) {
                this.percentage = 80;
              }
              this.goBottom();
              if (this.percentage == 80) {
                this.percentage = 100;
              }
              this.timer = setInterval(() => {
                this.percentage = 0;
              }, 2000);
              break;
            case 'init':
              if (this.user.username == message.username) {
                messageStr = '“' + this.user.cname + '”进入聊天室';
              } else {
                messageStr = '“' + message.cname + '”上线了！';
              }
              this.$notify({
                title: '提醒',
                message: ('i', { style: 'color: teal' }, messageStr),
                position: 'bottom-right',
                showClose: false
              });
              break;
          }
        };

        this.ws.onclose = () => {
          // 关闭 websocket
          // alert("连接已关闭...");
        };
      } else {
        this.$message({
          type: 'error',
          message: '您的浏览器不支持我们的对话基础！您可以更换Google Chrome'
        });
      }


    },
    async getStorage () {
      const userInfo = await window.localStorage.getItem('__USER__');
      if (!userInfo) {
        this.open();
      } else {
        this.user = JSON.parse(base64.decode(userInfo));
        this.openWebsocket();
      }
    },
    sendMessage () {
      if (!this.message) {
        this.$message({
          type: 'error',
          message: '这是一条空消息！！'
        })
        return false;
      }
      let message = JSON.stringify({ 'type': 'message', 'send_user': this.user, 'message': this.message, 'message_type': 'chat' });
      // 进度条
      clearInterval(this.timer);
      if (this.percentage == 0) {
        this.percentage = 10;
      }
      this.ws.send(message);
      this.message = '';
    },
    // 滚动到最底部
    goBottom () {
      this.$nextTick(function () {
        let el = document.getElementsByClassName('chat-container-top')[0];
        el.scrollTop = el.scrollHeight; // 滚动高度
      })
    },
    // 展示聊天表情
    showExpression () {
      this.dialogVisible = true;
    },
    //选择表情发送
    checkExpression (url) {
      let message = JSON.stringify({ 'type': 'message', 'send_user': this.user, 'message': url, 'message_type': 'expression' });
      this.ws.send(message);
      this.dialogVisible = false;
    },
    // 进度条
    format () {
      return this.percentage == 100 ? '发送成功' : this.percentage == 0 ? '' : '发送中...';
    }
  }
}
</script>
<style lang="less">
@import url("../../public/css/chat.css");
</style>