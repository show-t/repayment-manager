<template>
  <div>
    <div v-for="item in menuItems" :key="item">
      <v-btn @click="movePage(item.to)" block x-large>{{ item.name }}</v-btn>
      <br />
    </div>
    <p>{{ loggedIn }}</p>
    <p v-if="loggedIn">{{context}}</p>
    <v-btn @click="send">テストメッセージ</v-btn>
    <!--
    <div v-if="loggedIn" class="card">
      <div class="card-content">
        <div class="media">
          <div class="media-left">
            <figure class="image is-48x48">
              <img :src="pictureUrl" />
            </figure>
          </div>
          <div class="media-content">
            <p class="title is-4">{{ displayName }}</p>
            <p class="subtitle is-6">@{{ userId }}</p>
          </div>
        </div>
        <div class="content">
          {{ statusMessage }}
        </div>
      </div>
    </div>
    -->
  </div>
</template>

<script lang="ts">
import Vue from "vue";
export default Vue.extend({
  data: function () {
    return {
      menuItems: [
        { name: "立替金支払いリクエスト", to: "/request" },
        { name: "立替金支払い完了報告", to: "/report" },
        { name: "精算リストをみる", to: "/view" },
      ],
      loggedIn: false,
      context:{}
    };
  },
  methods: {
    movePage: function (path: string) {
      this.$router.push(path);
    },
    send:function(){
      liff.sendMessages([
        {
          type: 'text',
          text: 'てすと'
        }
      ]).then(()=>{
        console.log('message sent')
        //liff.closeWindow()
      }).catch((err)=>{
        console.log('Error:', err)
          //liff.closeWindow()
      })
    }
  },
  created() {
    liff
      .init({
        liffId: this.$config.LIFF_ID,
      })
      .then(() => {
        this.loggedIn = liff.isLoggedIn();
        this.context = liff.getContext();
      });
  },
  mounted(){
    //this.getProfile()
  }
});
</script>