<template>
    <v-app>
     <v-card>
      <v-card-title>立替金支払い請求フォーム</v-card-title>
      <v-card-text>
        <p>立替日</p>   
        <v-menu>
            <template v-slot:activator={on}>
                <v-text-field
                    v-on="on"
                    readonly
                    v-model="reqData.date"
                    label=""
                    outlined
                    required
                >
                </v-text-field>
            </template>
            <v-date-picker
                v-model="reqData.date"
                no-title
                scrollable
            >
                <v-spacer></v-spacer>
                <v-btn
                    text
                    color="primary"
                >
                Cancel
                </v-btn>
            </v-date-picker>
        </v-menu>
        
        
        <p>内容</p>
        <v-text-field
          v-model="reqData.contents"
          label=""
          outlined
          required
        >
        </v-text-field>
        <p>立替者</p>
        <v-select
            v-model="reqData.repayer"
            :items="users"
            label=""
            outlined
        ></v-select>
        <p>対象者</p>
        <v-select
            v-model="reqData.targets"
            :items="users"
            chips
            label=""
            multiple
            outlined
          ></v-select>
        <p>金額 (※単価を記入)</p>
        <v-text-field
          v-model="reqData.amount"
          label=""
          outlined
          required
        >
        </v-text-field>
        <p>備考</p>
        <v-textarea
            v-model="reqData.remark"    
            name="input-7-4"
            label=""
            outlined
        ></v-textarea>
      </v-card-text>
      <v-divider></v-divider>
      <v-card-actions>
        <v-btn text @click="submit" block x-large>送信する</v-btn>
        <span v-if="success">送信成功！</span>
      </v-card-actions>
    </v-card>
    </v-app>
</template>
<script lang="ts">
import Vue from 'vue'
import axios from 'axios'
export default Vue.extend({
    data(){
        return{
            reqData:{
                date:"",
                contents:"",
                repayer:"",
                targets:[],
                amount:"",
                remark:""
            },
            users:[],
            endpoint: "https://script.google.com/macros/s/AKfycbzLSPfDaJBd85eoTZpptb3ceUvp34VQj_Y7nC7vNurczRiX7J3TR9AgabNQHoUIJKSr/exec"
        }
    },
    methods:{
        submit:function(){
            var body:string = JSON.stringify(this.reqData)
            var params ={headers:{'Content-Type':'text/plain'}}
            axios.post(this.endpoint,body,params)
                .then(res=>{
                    console.log(res)
                })
                .catch(err=>{
                    console.log(err)
                })
        }
    },
    mounted(){
        const params = {
            crossDomain: true
        }
        axios.get(this.endpoint)
            .then(res=>this.users = res.data.users)
    }
})
</script>