<template>
  <div>
    <div v-for="item in menuItems" :key="item"> 
      <v-btn @click="movePage(item.to)" block x-large>{{item.name}}</v-btn>
      <br>
    </div>
  </div> 
</template>

<script lang="ts">
import Vue from 'vue'
export default Vue.extend({
  data:function(){
    return{
      menuItems:[
        {name:"立替金支払いリクエスト",to:"/request"},
        {name:"立替金支払い完了報告",to:"/report"},
        {name:"精算リストをみる", to:"/view"}
      ]
    }
  },
  methods:{
    movePage: function(path :string){
      this.$router.push(path)
    }
  }
})
</script>