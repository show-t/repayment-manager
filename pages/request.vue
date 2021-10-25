<template>
  <v-app>
    <v-form ref="form" v-model="valid" lazy-validation>
      <v-card>
        <v-card-title>立替金支払い請求フォーム</v-card-title>
        <v-card-text>
          <p>立替日</p>
          <v-menu>
            <template v-slot:activator="{ on }">
              <v-text-field
                prepend-inner-icon="mdi-calendar-range"
                v-on="on"
                readonly
                v-model="reqData.date"
                :rules="[rules.required]"
                label=""
                outlined
                required
              >
              </v-text-field>
            </template>
            <v-date-picker v-model="reqData.date" no-title scrollable>
              <v-spacer></v-spacer>
              <v-btn text color="primary"> Cancel </v-btn>
            </v-date-picker>
          </v-menu>
          <p>内容</p>
          <v-text-field
            prepend-inner-icon="mdi-subtitles"
            v-model="reqData.contents"
            :rules="[rules.required]"
            label=""
            outlined
            required
          >
          </v-text-field>
          <p>立替者</p>
          <v-select
            prepend-inner-icon="mdi-account-cash"
            v-model="reqData.repayer"
            :items="users"
            :rules="[rules.required]"
            label=""
            outlined
          ></v-select>
          <p>対象者</p>
          <v-select
            prepend-inner-icon="mdi-account-arrow-right"
            v-model="reqData.targets"
            :items="users"
            :rules="[rules.required, rules.counter]"
            chips
            label=""
            multiple
            outlined
          ></v-select>
          <p>金額 (※単価を記入)</p>
          <v-text-field
            prepend-inner-icon="mdi-cash-multiple"
            type="number"
            v-model="reqData.amount"
            :rules="[rules.required]"
            label=""
            prefix="¥"
            outlined
            required
          >
          </v-text-field>
          <p>備考 (※任意)</p>
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
            <template v-slot:activator="{ on, attrs }">
              <v-btn :disabled="!valid" v-bind="attrs" v-on="on" block x-large>
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
                  <v-row
                    ><v-col>【立替日】</v-col
                    ><v-col>{{ reqData.date }}</v-col></v-row
                  >
                  <v-row
                    ><v-col>【内　容】</v-col
                    ><v-col>{{ reqData.contents }}</v-col></v-row
                  >
                  <v-row
                    ><v-col>【立替者】</v-col
                    ><v-col>{{ reqData.repayer }}</v-col></v-row
                  >
                  <v-row
                    ><v-col>【対象者】</v-col
                    ><v-col>{{ reqData.targets }}</v-col></v-row
                  >
                  <v-row
                    ><v-col>【金　額】</v-col
                    ><v-col>¥{{ reqData.amount }}</v-col></v-row
                  >
                  <v-row
                    ><v-col>【備　考】</v-col
                    ><v-col>{{ reqData.remark }}</v-col></v-row
                  >
                </v-container>
              </v-card-text>
              <v-divider></v-divider>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="primary" text @click="submit"> OK </v-btn>
                <v-btn text color="primary" @click="dialogConfirm = false">
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
                <v-row
                  ><v-col>{{ resData.ids.length }}件登録しました</v-col></v-row
                >
              </v-container>
              <v-container>
                <v-row v-for="id in resData.ids" :key="id"
                  ><v-col>{{ id }}</v-col></v-row
                >
              </v-container>
            </v-card-text>
            <v-divider></v-divider>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn text color="primary" @click="dialogSuccess = false">
                OK
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-card>
    </v-form>
  </v-app>
</template>
<script lang="ts">
import Vue from "vue";
import axios from "axios";
export default Vue.extend({
  data() {
    return {
      reqData: {
        date: "",
        contents: "",
        repayer: "",
        targets: [],
        amount: "",
        remark: "",
      },
      rules: {
        required: (val: any) => !!val || "必須項目",
        counter: (val: any) => val.length > 0 || "1人以上選択",
      },
      resData: {
        ids: [],
      },
      users: [],
      dialogConfirm: false,
      dialogSuccess: false,
      valid: false,
    };
  },
  methods: {
    submit: function () {
      if ((this.$refs.form as any).validate()) {
        this.post();
        this.dialogConfirm = false;
      }
    },
    post: function () {
      var body: string = JSON.stringify({ action: "regist", ...this.reqData });
      var params = { headers: { "Content-Type": "text/plain" } };
      axios
        .post(this.$config.GAS_ENDPOINT, body, params)
        .then((res) => {
          console.log(res.data);
          this.resData.ids = res.data;
          this.dialogSuccess = true;
          this.clearReqData();
        })
        .catch((err) => {
          console.log(err);
        });
    },
    clearReqData() {
      (this.$refs.form as any).reset();
    },
  },
  created() {
    var url = this.$config.GAS_ENDPOINT + "?info=users";
    console.log(url);
    axios.get(url).then((res) => (this.users = res.data.users));
  },
  mounted() {
    (this.$refs.form as any).validate();
  },
});
</script>