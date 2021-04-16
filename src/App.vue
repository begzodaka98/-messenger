<template>
  <div id="app">
    <div class="scroll-down" @click="scrollDown()"> V </div>
    <div v-if="author">
      <div v-for="(message, index) in chat" :key="index">
        <!-- Author part -->
        <div
          :class="['message', message.author == author ? 'own-message' : '']" 
          v-if="message.author && message.text"
        >
          <div style="display: flex; justify-content: space-between">
            <p class="author">{{ message.author }}</p>
            <p class="reply" @click="replyTo(message)">reply</p>
          </div>
          <!-- reply part -->
          <div v-if="message.reply" class="reply-content">
            <p class="author">{{ message.reply.author }}</p>
            <img style="width: 100px" v-if="message.reply.imageUrl" :src="message.reply.imageUrl" alt="">
            <p class="text">{{ message.reply.text }}</p>
          </div>
          <!-- message img -->
          <img class="img" v-if="message.imageUrl" :src="message.imageUrl" alt="">
          <!-- message text -->
          <p class="text">{{ message.text }}</p>
          <p class="time"> {{ new Date(message.time).toString().split('GMT')[0] }} </p>
        </div>
      </div>

      <div class="input">
        <div class="reply-author" v-if="Object.keys(reply).length">
          <p>Reply to : {{ reply.author }}</p>
          <p style="cursor: pointer" @click="reply = {}">X</p>
        </div>
        <!-- <input class="author-text" type="text" v-model="author" placeholder="Author"> -->
        <input style="width: 300px" class="imageUrl" type="text" v-model="imageUrl" placeholder="Image URL">
        <br> <br>
        <div style="display: flex;">
          <input class="text-text" type="text" v-model="text" placeholder="Write to message...">
          <button class="send" @click="send()"> > </button>
        </div>
        <br> <br>
        <br> <br>
        <br> <br>
        <br> <br>
      </div>
    </div>
    <login @setAuthor="setAuthor" :author="author" v-else />
  </div>
</template>

<script>
import $cookies from 'vue-cookies';
import firebase from 'firebase/app';
import 'firebase/firestore';
import Login from './components/Login.vue';

export default {
  components: { Login },
  name: 'App',
  data() {
    return {
      author: '',
      text: '',
      imageUrl: '',
      reply: {},
      chat: [],
    }
  },
  mounted() {
    this.getChat();
    let cookAuthor = $cookies.get('author');
    if(cookAuthor) {
      this.author = cookAuthor;
    }
  },
  methods: {
    replyTo(message) {
      this.reply = message;
    },
    scrollDown() {
      window.scroll(0, 999999999999)
    },
    setAuthor(str) {
      this.author = str;
    },

    getChat() {
      this.chat = [];
      firebase.firestore().collection('messages').get().then((querySnapshot) => {
        querySnapshot.forEach((doc) => {
          this.chat.push(doc.data())
        });
      });
    },
    send() {
      let docName = String(new Date().getTime())
      let message = {
        reply : this.reply,
        author: this.author,
        text: this.text,
        time: new Date().getTime(),
        imageUrl: this.imageUrl,
      }
      firebase.firestore().collection('messages').doc(docName).set(message)
      .then(() => {
        this.chat.push(message)
        this.text = '';
        this.imageUrl = '';
        this.reply = {};
      })
    },
  }

}
</script>

<style>
@media (max-width: 960px){
  .message{
    width: 75%;
    padding-right: 10px;
  }
  .img {
  display: flex;
  justify-content: center;
  width: 90%;
}
.scroll-down {
  cursor: pointer;
  padding: 10px;
  border-radius: 10px;
  color: white;
  background-color: rgba(128, 128, 128, 0.404);
  position: fixed;
  left: 95%;
  top: 90vh;
}
}
body {
  font-family: sans-serif;
  margin: 0px;
  padding: 0px;
  background-image: url('./assets/oboi.jpg');
  background-repeat: no-repeat;
  background-size: cover;
  background-attachment: fixed;
}

#page {
	min-height: 100vh;
	display: flex;
	flex-direction: column;
	justify-content: center;
	transition: .2s ease;
}
input{
  border: none;
  border-radius: 30px;
}
#app {
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  max-width: 600px;
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}
.scroll-down {
  cursor: pointer;
  padding: 10px;
  border-radius: 10px;
  color: white;
  background-color: grey;
  position: fixed;
  left: 80vw;
  top: 90vh;
}
input, button {
  padding-left: 5px;
  padding-right: 5px;
  outline: none !important;
}
.message {
  background-color: white;
  padding-left: 10px;
  padding-right: 40px;
  max-width: 400px;
  border: 1px solid grey;
  border-radius: 10px 10px 10px 0;
  margin: 5px;
}
.own-message {
  background-color: rgb(1, 137, 200);
  border-radius: 10px 10px 0 10px;
  margin-left: auto;
  color: white !important;
}
.own-message > * {
  color: white  !important;
}
.author {
  min-width: max-content;
  max-width: 400px;
  margin-left: 5px;
  margin-top: 5px;
  margin-bottom: 0;
}
.reply {
  cursor: pointer;
  margin-top: 5px;
  margin-bottom: -10px;
  margin-right: -20px;
  color: rgb(94, 7, 7);
}

.reply-content {
  border-radius: 0 10px 10px 0;
  background-color:rgba(192, 192, 192, 0.39);
  padding: 4px;
  border-radius: 3px;
  box-shadow: #2c3e50 1px 1px 1px 1px;
  opacity: 0.6;
}
.reply-content > * {
  margin-top: 8px !important;
  color: black !important;
}
.img {
  margin-top: 10px;
  margin-left: 20px;
  max-width: 400px;
}
.text {
  max-width: 400px;
  font-size: 20px;
  color: black;
  margin-left: 7px;
  margin-bottom: 10px;
}
.input {
  display: flex;
  flex-direction: column;
  margin-top: 100px;
}
.reply-author {
  display: flex;
  justify-content: space-between;
  border-radius: 14px;
  padding: 10px;
  margin-bottom: -10px;
  width: 400px;
  color: white;
  background-color: black;
}
.author-text {
  width: 120px;
  height: 30px;
}
.imageUrl{
  margin-top: 20px;
  width: 120px;
  height: 30px;
}
.text-text {
  margin-top: -15px;
  font-size: 20px;
  width: 400px;
  height: 30px;
}
.send {
  margin-top: -16px;
  margin-bottom: -2px;
  padding-left: 10px;
  padding-right: 10px;
  border-radius: 20px;
  font-weight: bold;
  background-color: rgb(22, 105, 173);
  color: rgb(255, 255, 255);
  font-size: 25px;
  border: none;
}
.time {
  padding-left: 20px;
  font-size: 10px;
  color: grey;
  text-align: right;
}
</style>