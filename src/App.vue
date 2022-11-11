<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png"> 
  </div>
</template>

<script> 
import SockJS from "sockjs-client";
import Stomp from "webstomp-client";
import { Client } from '@stomp/stompjs';
export default {
  name: 'App', 
  data(){
    return {
      received_messages: [],
      send_message: null,
      connected: false,
      SOCKET_URL :'ws://localhost:9000/websocket'
    };
  },  
  methods:{
      testConnect(){
        const client = new Client({
        brokerURL: this.SOCKET_URL,
        reconnectDelay: 5000,
        heartbeatIncoming: 4000,
        heartbeatOutgoing: 4000,
        onConnect: function(){
          console.log("connected");
          client.subscribe('/topic/topic-1', function(message){
            console.log(message.body);
            
            
          });
          let obj = {
              "name":"Hello, STOMP",
              "email":"test@example.com"
            };
          client.publish({destination: "/app/message", body: JSON.stringify(obj),skipContentLengthHeader: true});
        },
        disconnect:function(){
          console.log("disconnect")
        }
      });

      client.activate();
    },
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
          let jsonObject = {
            "name":"Test",
            "data":"test"
          };
          this.stompClient.send('http://localhost:9000/app/message',JSON.stringify(jsonObject));
          this.stompClient.subscribe("http://localhost:9000/topic/topic-1", tick => {
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
    this.testConnect();
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
