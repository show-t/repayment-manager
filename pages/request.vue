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
                <v-dialog v-model="dialogConfirm">
                    <template v-slot:activator="{on,attrs}">
                        <v-btn
                            v-bind="attrs"
                            v-on="on"
                            block
                            x-large
                        >
                            送信する
                        </v-btn>
                    </template>
                    <v-card>
                        <v-card-title class="text-h5 grey lighten-2">
                            送信確認
                        </v-card-title>
                        <v-card-text>
                            <v-container>
                                <v-row><v-col>以下の内容で送信します</v-col></v-row>
                            </v-container>
                            <v-container>
                                <v-row><v-col>【立替日】</v-col><v-col>{{reqData.date}}</v-col></v-row>
                                <v-row><v-col>【内　容】</v-col><v-col>{{reqData.contents}}</v-col></v-row>
                                <v-row><v-col>【立替者】</v-col><v-col>{{reqData.repayer}}</v-col></v-row>
                                <v-row><v-col>【対象者】</v-col><v-col>{{reqData.targets}}</v-col></v-row>
                                <v-row><v-col>【金　額】</v-col><v-col>{{reqData.amount}}</v-col></v-row>
                                <v-row><v-col>【備　考】</v-col><v-col>{{reqData.remark}}</v-col></v-row>
                            </v-container>
                        </v-card-text>
                        <v-divider></v-divider>
                        <v-card-actions>
                            <v-spacer></v-spacer>
                            <v-btn
                                color="primary"
                                text
                                @click="submit"
                            >
                                OK
                            </v-btn>
                            <v-btn
                                text
                                color="primary"
                                @click="dialogConfirm = false"
                            >
                                Cancel
                            </v-btn>
                        </v-card-actions>
                    </v-card>
                </v-dialog>
            </v-card-actions>
            <v-dialog v-model="dialogSuccess">
                <v-card>
                        <v-card-title class="text-h5 grey lighten-2">
                            登録完了
                        </v-card-title>
                        <v-card-text>
                            <v-container>
                                <v-row><v-col>{{resData.ids.length}}件登録しました</v-col></v-row>
                            </v-container>
                            <v-container>
                                <v-row v-for="id in resData.ids" :key="id"><v-col>{{id}}</v-col></v-row>
                            </v-container>
                        </v-card-text>
                        <v-divider></v-divider>
                        <v-card-actions>
                            <v-spacer></v-spacer>
                            <v-btn
                                text
                                color="primary"
                                @click="dialogSuccess = false"
                            >
                                OK
                            </v-btn>
                        </v-card-actions>
                    </v-card>
            </v-dialog>
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
            resData:{
                ids:[],
            },
            users:[],
            endpoint: "https://script.google.com/macros/s/AKfycbzLSPfDaJBd85eoTZpptb3ceUvp34VQj_Y7nC7vNurczRiX7J3TR9AgabNQHoUIJKSr/exec",
            dialogConfirm: false,
            dialogSuccess: false,
        }
    },
    methods:{
        submit:function(){
            this.post()
            this.dialogConfirm = false
            
        },
        post:function(){
            var body:string = JSON.stringify(this.reqData)
            var params ={headers:{'Content-Type':'text/plain'}}
            axios.post(this.endpoint,body,params)
                .then(res=>{
                    console.log(res.data)
                    this.resData.ids = res.data
                    this.dialogSuccess = true
                    this.clearReqData()
                })
                .catch(err=>{
                    console.log(err)
                })
        },
        clearReqData(){
            this.reqData = {
                date:"",
                contents:"",
                repayer:"",
                targets:[],
                amount:"",
                remark:""
            }
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