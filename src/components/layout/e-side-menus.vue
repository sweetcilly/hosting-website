<template>
  <div v-if="asMobile">
    <v-expansion-panels accordion v-model="curIdx">
      <v-expansion-panel v-for="(it, i) in list" :key="i">
        <v-expansion-panel-header>
          {{ it.text }}
        </v-expansion-panel-header>
        <v-expansion-panel-content>
          <component
            :is="it.comp"
            :info="info"
            :active="i == curIdx"
            v-if="it.comp && info.id && activeIdxList.includes(i)"
          ></component>
        </v-expansion-panel-content>
      </v-expansion-panel>
    </v-expansion-panels>
  </div>

  <div class="d-flex" v-else>
    <div class="bdr-1">
      <v-list flat class="mt-3" min-width="240">
        <v-list-item-group v-model="curIdx" color="primary">
          <v-list-item v-for="(it, i) in list" :key="i">
            <v-list-item-content>
              <v-list-item-title v-text="it.text"></v-list-item-title>
            </v-list-item-content>
          </v-list-item>
        </v-list-item-group>
      </v-list>
    </div>
    <div class="flex-1 pd-20" style="min-height: 60vh" v-if="info.id">
      <component
        :is="it.comp"
        :info="info"
        :active="curItem.comp == it.comp"
        v-show="curItem.comp == it.comp"
        v-for="(it, i) in activeList"
        :key="i"
      ></component>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    list: Array,
    info: Object,
  },
  data() {
    const { tab = 0 } = this.$route.query;
    let curIdx = tab * 1;
    return {
      curIdx,
      activeIdxList: [curIdx],
    };
  },
  watch: {
    curIdx(tab) {
      if (!this.activeIdxList.includes(tab)) {
        this.activeIdxList.push(tab);
      }
      this.$router.replace({
        query: {
          tab,
        },
      });
    },
    "$route.query.tab"(tab) {
      if (tab != this.curIdx && tab > -1) {
        this.curIdx = tab * 1;
      }
    },
  },
  computed: {
    asMobile() {
      return this.$vuetify.breakpoint.smAndDown;
    },
    curItem() {
      return this.list[this.curIdx] || {};
    },
    activeList() {
      return this.list.filter((_, i) => {
        return this.activeIdxList.includes(i);
      });
    },
  },
};
</script>