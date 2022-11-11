<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png"> 
  </div>
</template>

<script> 
import SockJS from "sockjs-client";
import Stomp from "webstomp-client";

export default {
  name: 'App', 
  data(){
    return {
      received_messages: [],
      send_message: null,
      connected: false
    };
  },  
  methods:{
    send() {
      console.log("Send message:" + this.send_message);
      if (this.stompClient && this.stompClient.connected) {
        const msg = { name: this.send_message };
        this.stompClient.send("/app/hello", JSON.stringify(msg), {});
      }
    },
    connect() {
      this.socket = new SockJS("http://localhost:9000/websocket");
      this.stompClient = Stomp.over(this.socket);
      this.stompClient.connect(
        {},
        frame => {
          this.connected = true;
          console.log(frame,"connected");
          this.stompClient.subscribe("/ws/topic", tick => {
            console.log(tick);
            this.received_messages.push(JSON.parse(tick.body).content);
          });
        },
        error => {
          console.log(error,"eror");
          this.connected = false;
        }
      );
    },
    disconnect() {
      if (this.stompClient) {
        this.stompClient.disconnect();
      }
      this.connected = false;
    },
    tickleConnection() {
      this.connected ? this.disconnect() : this.connect();
    }
  },
  mounted() {
    this.connect();
  }

}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
