<style>
.bg-s0 {
  background: #75a6e7;
}
.bg-s1 {
  background: #84d199;
}
.bg-s2 {
  background: #be80e0;
}
</style>

<template>
  <v-card class="pd-20" outlined>
    <div v-if="!apps">
      <v-row>
        <v-col cols="12" :md="6" :lg="4" v-for="i in 3" :key="i">
          <v-skeleton-loader max-width="450" type="card"></v-skeleton-loader>
        </v-col>
      </v-row>
      <v-skeleton-loader class="mt-5" type="article" />
    </div>

    <div v-else-if="!apps.length" class="ta-c pd-20 mt-8 mb-8">
      <img src="img/empty/bill.svg" style="height: 200px" />
      <div class="gray mt-4">
        {{ $t(`${locales}NoAppDeployed`) }}
      </div>
    </div>

    <div v-else>
      <div class="mb-6 d-flex al-c">
        <v-btn
          :loading="loading"
          @click="getData"
          small
          class="ml-auto mr-1 mt-5"
          plain
        >
          <v-icon>mdi-refresh</v-icon>
        </v-btn>
        <!--  -->
        <v-select
          style="max-width: 300px"
          v-model="appId"
          :items="apps"
          item-text="projectName"
          item-value="projectId"
          label="All DApps"
          hide-details
          single-line
          :menu-props="{ offsetY: true }"
        ></v-select>
      </div>
      <v-row class="mb-6">
        <v-col cols="12" md="4" v-for="(row, i) in cardList" :key="i">
          <div class="pd-20 bdrs-5 white-0" :class="'bg-s' + i">
            <div class="d-flex al-c">
              <span>{{ row.title }}</span>
              <e-tooltip right max-width="300" v-if="row.tip">
                <v-icon slot="ref" color="#fff" size="14" class="pa-1 d-ib"
                  >mdi-help-circle-outline</v-icon
                >
                <span>{{ row.tip }}</span>
              </e-tooltip>
            </div>
            <div class="fz-14 mt-5">
              <div>
                <span class="op-7">{{ $t(`${locales}Today`) }}</span>
                <span class="fl-r">{{ row.today }}</span>
              </div>
              <div class="mt-2">
                <span class="op-7">{{ $t(`${locales}Yesterday`) }}</span>
                <span class="fl-r">{{ row.yesterday }}</span>
              </div>
            </div>
          </div>
        </v-col>
      </v-row>

      <v-row class="mb-6 ta-c">
        <v-col cols="4" v-for="(it, i) in summaryList" :key="i">
          <div class="el-label-1">{{ it.title }}</div>
          <div class="fz-25 mt-1">{{ it.value }}</div>
        </v-col>
      </v-row>

      <v-row class="mb-6">
        <v-col cols="12" md="6" v-for="(it, i) in chartList" :key="i">
          <statis-chart
            :reload="!loading"
            :appId="appId"
            :title="it.title"
            :type="it.type"
          ></statis-chart>
        </v-col>
      </v-row>

      <h4 class="mb-2">Retention Rate</h4>
      <v-skeleton-loader type="article" v-if="tableLoading"></v-skeleton-loader>
      <statis-table :list="tableList" v-else></statis-table>
    </div>
  </v-card>
</template>

<script>
import { mapState } from "vuex";

export default {
  data() {
    const { appId } = this.$route.query;
    return {
      locales: "dashboard.statistics.",
      appId,
      apps: null,
      info: null,
      loading: false,
      chartList: [
        {
          title: this.$tc(`dashboard.statistics.TotalUser`),
          type: "TOTAL_USERS",
        },
        { title: this.$t(`dashboard.statistics.NewUsers`), type: "NEW_USERS" },
        {
          title: this.$t(`dashboard.statistics.UniqueVisitor`),
          type: "UNIQUE_VISITOR",
        },
        { title: this.$t(`dashboard.statistics.PageView`), type: "PAGE_VIEW" },
      ],
      tableList: [],
      tableLoading: false,
    };
  },
  computed: {
    ...mapState({
      isFocus: (s) => s.isFocus,
    }),
    inTab() {
      return this.$route.path == "/dashboard/statistics";
    },
    summaryList() {
      if (!this.info) return [];
      return [
        {
          title: this.$tc(`dashboard.statistics.TotalUser`),
          value: this.info.totalUsers,
        },
        {
          title: "Total UV",
          value: this.info.totalUv || 0,
        },
        {
          title: "Total PV",
          value: this.info.totalPv || 0,
        },
      ];
    },
    cardList() {
      const info = this.info;
      if (!info) return [];
      return [
        {
          title: this.$t(`dashboard.statistics.NewUsers`),
          today: "+" + info.todayNewUser,
          yesterday: "+" + info.yesterdayNewUser,
        },
        {
          title: "Unique Visitor",
          today: info.todayUv,
          yesterday: info.yesterdayUv,
          tip: `A computer client that visits the site as a
                  Visitor. The same client is counted only once between 00:00
                  and 24:00 (UTC).`,
        },
        {
          title: "Page Views",
          today: info.todayPv,
          yesterday: info.yesterdayPv,
          tip: `Page views or clicks are counted once per user visit.`,
        },
      ];
    },
  },
  watch: {
    inTab(val) {
      if (this.appId && val) this.getApps();
    },
    appId(val) {
      this.$router.replace({
        query: {
          appId: val,
        },
      });
      this.getData();
    },
    "$route.query.appId"(val) {
      if (val) {
        this.appId = val;
      }
    },
  },
  mounted() {
    this.getApps();
  },
  methods: {
    async getApps() {
      try {
        const { data } = await this.$http.get("/analytics/user/project/list");
        this.apps = data;
        if (data.length) {
          const unvalid =
            data.filter((it) => it.projectId == this.appId).length == 0;
          if (unvalid) {
            const arr = [...data].sort((a, b) => b.queryCount - a.queryCount);
            this.appId = arr[0].projectId;
            console.log("appid", this.appId);
          } else {
            this.getData();
          }
        }
      } catch (error) {
        //
      }
    },
    async getData() {
      try {
        if (!this.appId) return;
        this.loading = true;
        const { data } = await this.$http.get(
          "/analytics/user/data/project/" + this.appId
        );
        this.info = data;
        this.tableLoading = true;
        const { data: list } = await this.$http.get(
          "/analytics/user/retention/project/" + this.appId
        );
        this.tableList = list.map((it) => {
          it.date = new Date(it.createAt * 1e3).format("date");
          return it;
        });
      } catch (error) {
        console.log(error);
      }
      this.tableLoading = false;
      this.loading = false;
    },
  },
};
</script>