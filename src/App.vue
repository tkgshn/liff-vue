<template>
<div id='app'>
  <img class="userIcon" alt="userIcon" :src="userPictureURL" />
  <input v-model="message" placeholder="送信するメッセージを入力" />
  <button class="btn-square" v-if="isInClient" @click="sendMessage">
    メッセージを送信する
  </button>
  <HelloWorld v-if="isLoggedIn" v-bind:msg="'Hello, ' + userName" />
  <button class="btn-square" v-if="!isInClient" @click="logout">
    ログアウト
  </button>
</div>
</template>

<script>
import HelloWorld from "./components/HelloWorld.vue";
import liff from "@line/liff";

export default {
  name: "App",
  components: {
    HelloWorld,
  },
  data() {
    return {
      userName: undefined,
      userPictureURL: undefined,
      isInClient: undefined,
      isLoggedIn: false,
      message: "",
    };
  },
  created() {
    liff
      .init({
        liffId: process.env.VUE_APP_LIFF_ID,
      })
      .then(() => {
        this.isLoggedIn = liff.isLoggedIn();
        if (!liff.isInClient() && !liff.isLoggedIn()) {
          liff.login();
        } else {
          this.isInClient = liff.isInClient();
          liff.getProfile().then((profile) => {
            this.userName = profile.displayName;
            this.userPictureURL = profile.pictureUrl;
          });
        }
      });
  },
  methods: {
    logout: function () {
      console.log(this.message);
      if (liff.isLoggedIn()) {
        liff.logout();
        location.reload();
      }
    },
    sendMessage: function () {
      liff
        .sendMessages([
          {
            type: "text",
            text: this.message,
          },
        ])
        .then(() => {
          console.log("message sent");
          liff.closeWindow()
        })
        .catch((err) => {
          console.log("error", err);
        });
    },
  },
};
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
.userIcon {
  width: 100%;
}
.btn-square {
  display: inline-block;
  padding: 0.5em 1em;
  text-decoration: none;
  background: #42b983;
  color: #fff;
  border-bottom: solid 4px #627295;
  border-radius: 3px;
}
.btn-square:active {
  -webkit-transform: translateY(4px);
  transform: translateY(4px);
  border-bottom: none;
}
</style>