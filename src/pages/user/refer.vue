<style lang="scss">
.refer-header {
  background: #2b0c74 url(/img/bg/refer-1.jpeg) no-repeat;
  background-size: 100%;
  .wrap-1 {
    background: url(/img/bg/refer-2.png) no-repeat 90% center;
    background-size: contain;
  }
}
.page-refer {
  .link-wrap {
    max-width: 600px;
    margin: 50px auto 10px;
  }
  table {
    border-spacing: 0;
    font-size: 15px;
    thead {
      background: #aebed5;
      color: #fff;
    }
    tbody {
      font-size: 14px;
    }
    td {
      padding: 10px;
    }
    tr:nth-child(even) {
      background: #e8f0fa;
    }
  }
}
</style>

<template>
  <div class="page-refer">
    <div class="refer-header">
      <div class="wrap-1">
        <div class="con-1">
          <div class="white-0 fw-b" :class="asMobile ? 'fz-30' : 'fz-35'">
            <p>Refer Friends.</p>
            <p>Enter Web3.0 Together.</p>
          </div>
          <div class="link-wrap d-flex al-c bg-white-9 bdrs-5">
            <span class="pd-10 gray fz-14 mr-auto line-1 shrink-1">
              {{ shareUrl }}
            </span>
            <v-btn plain color="primary" small @click="onCopy">Copy</v-btn>
            <v-btn plain color="primary" small @click="onInvite">Share</v-btn>
          </div>
        </div>
      </div>
    </div>

    <div class="wrap-1">
      <div class="con-2">
        <v-card outlined>
          <div class="pd-20 bdb-1 d-flex al-c">
            <span>My Referrals</span>
            <v-btn icon small class="ml-2" @click="getList" :loading="loading">
              <v-icon>mdi-refresh</v-icon>
            </v-btn>
          </div>
          <div class="pd-20">
            <div
              style="background: #4f80ac"
              class="pd-20 white-0 d-flex space-btw"
            >
              <div class="ta-c" v-for="(it, i) in statisList" :key="i">
                <p class="op-7 mb-5" :class="asMobile ? 'fz-12' : 'fz-13'">
                  {{ it.label }}
                </p>
                <v-badge
                  :content="'+' + it.badge"
                  :value="it.badge > 0"
                  color="error"
                >
                  <p class="fz-25 fw-b">{{ it.value || 0 }}</p>
                </v-badge>
              </div>
            </div>

            <div class="ov-a mt-5 bd-1b ov-h">
              <table class="w100p ta-c" style="min-width: 260px">
                <thead>
                  <tr>
                    <td>#</td>
                    <td>E-mail</td>
                    <td>CreatedAt</td>
                    <td>Deployed</td>
                    <td>Status</td>
                  </tr>
                </thead>
                <tbody class="op-9">
                  <tr v-for="(it, i) in list" :key="i">
                    <td>{{ 1 + i }}</td>
                    <td>{{ it.email }}</td>
                    <td>{{ new Date(it.inviteAt).format() }}</td>
                    <td>
                      <act-e-link no-copy :domain="it.domain" />
                    </td>
                    <td>
                      {{ it.valid ? "valid" : "invalid" }}
                    </td>
                  </tr>
                </tbody>
              </table>
              <div class="pa-3 ta-c" v-if="!list.length">
                <div class="op-5 fz-14 mt-8">
                  {{ loading ? "Loading..." : "No Referrals Now" }}
                </div>
              </div>
            </div>

            <div class="mt-6" v-if="pageLen > 1">
              <v-pagination
                @input="onPage"
                v-model="page"
                :length="pageLen"
                prev-icon="mdi-menu-left"
                next-icon="mdi-menu-right"
                :total-visible="7"
              ></v-pagination>
            </div>
          </div>
        </v-card>
      </div>
    </div>

    <v-dialog v-model="popInvite" max-width="450" v-if="code">
      <e-dialog-close @click="popInvite = false" />
      <div class="pd-30 mt-2">
        <div v-show="isBgLoad">
          <div class="d-flex f-center al-c">
            <div class="mt-3 bd-1 pos-r">
              <div class="pos-r" ref="imgWrap">
                <img
                  @load="isBgLoad = true"
                  src="img/bg/act-share.png"
                  style="width: 200px"
                  class="ev-n"
                />
                <img v-if="qrImg" :src="qrImg" class="pos-a qr-img" />
              </div>
              <div class="pos-mask share-img-wrap" v-if="shareImg">
                <img :src="shareImg" class="w100p" />
              </div>
            </div>
          </div>

          <div class="mt-5 gray ta-c" v-if="isTouch">Long press to save</div>
          <div v-else class="mt-10 ta-c">
            <v-btn color="primary" @click="onSaveImg">Preservation</v-btn>
          </div>
        </div>
      </div>
    </v-dialog>
  </div>
</template>

<script>
import { mapState } from "vuex";
import * as clipboard from "clipboard-polyfill/text";
import html2canvas from "html2canvas";
import canvas2image from "@/plugins/canvas2image";
import qrcode from "qrcode";

export default {
  computed: {
    asMobile() {
      return this.$vuetify.breakpoint.smAndDown;
    },
    ...mapState({
      userInfo: (s) => s.userInfo,
      isTouch: (s) => s.isTouch,
    }),
    shareUrl() {
      return location.origin + "/#/?invite=" + this.code;
    },
    statisList() {
      const {
        bindDomain,
        deployToday,
        deployTotal,
        invitesToday,
        invitesTotal,
      } = this.statisData;
      return [
        {
          label: "Total Referrals",
          value: invitesTotal,
          badge: invitesToday,
        },
        {
          label: "Total Deployments",
          value: deployTotal,
          badge: deployToday,
        },
        {
          label: "Added Domains",
          value: bindDomain,
          // badge: 1,
        },
      ];
    },
  },
  data() {
    return {
      statisData: JSON.parse(localStorage.referrals_data || "{}"),
      code: null,
      list: [],
      page: 1,
      pageLen: 1,
      loading: false,
      popInvite: false,
      sharePre:
        "I am participating in 4EVERLAND  First Landing event. Deploying projects to win your share of 50 million 4EVER, come and join here: ",
      copyTxt: "",
      isBgLoad: false,
      qrImg: "",
      shareImg: "",
    };
  },
  created() {
    this.getCode();
    this.getList();
  },
  watch: {
    userInfo() {
      this.getCode();
    },
    async code() {
      this.copyTxt = this.sharePre + this.shareUrl;
      const url = await qrcode.toDataURL(this.shareUrl);
      this.qrImg = url;
    },
    isBgLoad() {
      console.log("load");
      setTimeout(() => {
        this.genImg();
      }, 300);
    },
  },
  methods: {
    onPage() {
      this.getList();
    },
    async onCopy() {
      try {
        await clipboard.writeText(this.shareUrl);
        this.$toast("Copied to clipboard !");
      } catch (error) {
        this.$toast("Copied fail");
      }
    },
    onCopied() {
      this.$toast("Copied to clipboard !");
      this.popInvite = false;
    },
    onSaveImg() {
      this.genImg("save");
    },
    async genImg(act) {
      try {
        const el = this.$refs.imgWrap;
        let canvas = document.createElement("canvas");
        const scale = 3;
        const width = el.offsetWidth * scale;
        const height = el.offsetHeight * scale;
        canvas.width = width * scale;
        canvas.height = height * scale;
        canvas.getContext("2d").scale(scale, scale);
        canvas = await html2canvas(el, {
          canvas,
          width,
          height,
          scale,
        });
        // console.log(canvas);
        if (act == "save") {
          canvas2image.saveAsImage(
            canvas,
            width,
            height,
            "jpeg",
            "4everland-" + this.code
          );
        }
        const img = canvas2image.convertToJPEG(canvas, width, height);
        this.shareImg = img.src;
      } catch (error) {
        console.log(error);
        this.$toast("fail to generate share image");
      }
    },
    onInvite() {
      if (!this.code) return;
      this.popInvite = true;
    },
    async getCode() {
      this.code = this.userInfo.inviteCode;
      if (this.code) return;
      const { data } = await this.$http.get("/invite/code");
      this.code = data;
    },
    async getData() {
      const { data } = await this.$http.get("/invite/day/analytics");
      this.statisData = data;
      localStorage.referrals_data = JSON.stringify(data);
    },
    async getList() {
      try {
        this.loading = true;
        const params = {
          page: this.page - 1,
          size: 10,
        };
        const { data } = await this.$http.get("/invite/list", {
          params,
        });
        // console.log(data);
        this.list = [...data.page];
        this.pageLen = Math.max(1, Math.ceil(data.total / params.size));
        await this.getData();
      } catch (error) {
        //
      }
      this.loading = false;
    },
  },
};
</script>