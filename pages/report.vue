<template>
  <v-app>
    <v-card>
      <v-container>
        <v-select
          prepend-inner-icon="mdi-account-arrow-right"
          v-model="fromUser"
          :items="resData.users"
          label="FROM"
          outlined
          @change="filterUserData"
        ></v-select>
        <v-row>
          <v-col :align="align" :align-self="align">
            <v-btn
              outlined
              class="d-flex justify-space-between mb-6"
              @click="swapUser"
            >
              <v-icon>mdi-swap-vertical-bold</v-icon>
            </v-btn>
          </v-col>
        </v-row>
        <v-select
          prepend-inner-icon="mdi-account-arrow-left"
          v-model="toUser"
          :items="resData.users"
          label="TO"
          outlined
          @change="filterUserData"
        ></v-select>
      </v-container>
      <v-divider></v-divider>
      <v-container>
        <v-text-field
          class="right-input"
          outlined
          readonly
          prefix="¥"
          v-model="total"
          label="総支払い額"
        ></v-text-field>
      </v-container>
      <v-divider></v-divider>
      <v-spacer></v-spacer>
      <v-card>
        <v-container>
          {{ records.length }}件の支払いがあります
          <v-list>
            <v-list-group v-for="record in records" :key="record.Id">
              <template v-slot:activator>
                <v-list-item-content>
                  <v-list-item-title color="primary">
                    <v-row>
                      <v-col>{{ record.Id }}</v-col>
                      <v-col>¥{{ record.Amount }}</v-col>
                    </v-row>
                  </v-list-item-title>
                </v-list-item-content>
              </template>
              <v-card>
                <v-list-item>
                  <v-container>
                    <v-row
                      ><v-col>【立替日】</v-col
                      ><v-col>{{ record.Date }}</v-col></v-row
                    >
                    <v-row
                      ><v-col>【内　容】</v-col
                      ><v-col>{{ record.Contents }}</v-col></v-row
                    >
                    <v-row
                      ><v-col>【金　額】</v-col
                      ><v-col>¥{{ record.Amount }}</v-col></v-row
                    >
                    <v-row
                      ><v-col>【備　考】</v-col
                      ><v-col>{{ record.Remark }}</v-col></v-row
                    >
                  </v-container>
                </v-list-item>
                <v-btn
                  :loading="loading"
                  :disabled="loading"
                  color="indigo"
                  outlined
                  block
                  @click="report(record.Id)"
                >
                  送ったよ!
                </v-btn>
                <v-spacer></v-spacer>
              </v-card>
            </v-list-group>
          </v-list>
        </v-container>
      </v-card>
    </v-card>
    <v-dialog v-model="dialogSuccess">
      <v-card>
        <v-card-title class="text-h5 grey lighten-2"> 更新完了 </v-card-title>
        <v-card-text>
          <v-container>
            <v-row><v-col>更新処理が完了しました</v-col></v-row>
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
    <p>{{loggedIn}}</p>
  </v-app>
</template>
<script lang="ts">
import Vue from "vue";
import axios from "axios";
export default Vue.extend({
  data() {
    return {
      fromUser: "",
      toUser: "",
      records: [],
      resData: {
        users: [],
        list: {
          regends: {
            ja: [],
            en: [],
          },
          datas: [],
        },
      },
      total: 0,
      dialogConfirm: false,
      dialogSuccess: false,
      loading: false,
      align: "center",
      loggedIn :false
    };
  },
  created() {
    liff
      .init({
        liffId: this.$config.LIFF_ID,
      })
      .then(() => {
        console.log("LIFF INIT!");
        this.loggedIn = liff.isLoggedIn()
      });

    var url = this.$config.GAS_ENDPOINT + "?info=users,list";
    console.log(url);
    axios.get(url).then((res) => {
      console.log(res);
      this.resData.users = res.data.users;
      this.resData.list = res.data.list;
    });
  },
  methods: {
    filterUserData: function () {
      var records: any[] = this.resData.list.datas.filter((v) => {
        return (
          v[this.getIndexEn("Target")] == this.fromUser &&
          v[this.getIndexEn("Repayer")] == this.toUser &&
          v[this.getIndexEn("Paid")] == false
        );
      });
      this.records = records.reduce((acc1: any, record: any[]) => {
        return [
          ...acc1,
          record.reduce((acc2: any, field: any, idx: number) => {
            return {
              ...acc2,
              ...{ [this.resData.list.regends.en[idx]]: field },
            };
          }, {}),
        ];
      }, []);
      this.total = this.records.reduce((acc: any, record: any) => {
        return acc + record.Amount;
      }, 0);
    },
    getIndexEn: function (regend: string) {
      return (this.resData.list.regends.en as string[]).indexOf(regend);
    },
    report: function (id: string) {
      this.dialogConfirm = false;
      this.loading = true;
      var url = this.$config.GAS_ENDPOINT;
      var body: string = JSON.stringify({ action: "paid", Id: id });
      var params = { headers: { "Content-Type": "text/plain" } };
      axios
        .post(url, body, params)
        .then((res) => {
          console.log(res);
          if (res.data.success == true) {
            this.paid(id);
            this.filterUserData();
            this.dialogSuccess = true;
            this.notice([id]);
          }
        })
        .finally(() => {
          this.loading = false;
        });
    },
    swapUser: function () {
      var temp: string;
      temp = this.fromUser;
      this.fromUser = this.toUser;
      this.toUser = temp;
      this.filterUserData();
    },
    paid: function (id: string) {
      this.resData.list.datas.map((v: any) => {
        if (v[this.getIndexEn("Id")] == id) {
          v[this.getIndexEn("Paid")] = true;
        }
      });
    },
    notice: function (ids: string[]) {
      const url = this.$config.LINEBOT_ENDPOINT + "/send_report";
      console.log(url);
      const params = { headers: { "Content-Type": "text/plain" } };
      const context = liff.getContext();
      console.log(context);

      const toId =
        context.type == "utou"
          ? context.userId
          : context.type == "group"
          ? context.groupId
          : null;

      console.log(toId);

      var datas: any = this.resData.list.datas;
      var records = ids.map((id) => {
        return datas.reduce((acc: any, v: any) => {
          return v[0] == id ? v : acc;
        }, []);
      });

      var messages = records.map((record) => {
        var msg = "【" + record[0] + "】\n";
        msg += "FROM: @" + record[4] + "\n";
        msg += "TO  : @" + record[3] + "\n";
        msg += "¥" + record[5] + "\n";
        msg += record[2];
        return msg;
      });

      console.log(messages);

      var body: any = JSON.stringify({
        toId: toId,
        messages: ids,
      });

      if (toId != null) {
        axios
          .post(url, body, params)
          .then((res) => {
            console.log(res);
          })
          .catch((err) => {
            console.log("Error:", err);
          });
      }
    },
  },
});
</script>
<style>
.right-input input {
  text-align: right;
}
</style>