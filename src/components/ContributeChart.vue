<template>
  <v-container>
    <v-row class="d-flex justify-end">
      <div class="text-center my-3">
        <span class="font-weight-bold btn_label">Chart:</span>
        <v-menu offset-y transition="scale-transition">
          <template v-slot:activator="{ on, attrs }">
            <v-btn
              color="primary"
              dark
              v-bind="attrs"
              v-on="on"
              width="150"
              class="mr-5"
            >
              {{ selectedChart }}
            </v-btn>
          </template>
          <v-list>
            <template v-for="item in chartActivities">
              <v-list-item :key="item.name" link class="pa-0">
                <v-list-item-content class="pa-0">
                  <v-btn
                    class="pa-0"
                    color="white"
                    @click="changeChartType(item)"
                    >{{ item.name }}</v-btn
                  >
                </v-list-item-content>
              </v-list-item>
            </template>
          </v-list>
        </v-menu>
        <span class="font-weight-bold btn_label">Info:</span>
        <v-menu offset-y transition="scale-transition">
          <template v-slot:activator="{ on, attrs }">
            <v-btn color="primary" dark v-bind="attrs" v-on="on" width="150">
              {{ selected }}
            </v-btn>
          </template>
          <v-list>
            <template v-for="item in activities">
              <v-list-item :key="item.name" link class="pa-0">
                <v-list-item-content class="pa-0">
                  <v-btn
                    class="pa-0"
                    color="white"
                    @click="changeInfoData(item)"
                    >{{ item.name }}</v-btn
                  >
                </v-list-item-content>
              </v-list-item>
            </template>
          </v-list>
        </v-menu>
      </div>
    </v-row>
    <v-divider class="mb-10"></v-divider>
    <v-row v-if="selectedChart == 'line'">
      <v-col>
        <v-card max-width="500" class="my-2" height="550">
          <v-card-title class="white--text orange darken-4">
            User Directory
          </v-card-title>
          <v-card-text class="pt-4">
            可以依照右上角的選項去排序出contributor，點擊contributor的頭貼後，右方會顯示出他的曲線圖
          </v-card-text>
          <v-divider></v-divider>
          <v-virtual-scroll :items="data" :item-height="60" height="360">
            <template v-slot:default="{ item, index }">
              <v-list-item>
                <v-list-item-avatar>
                  <v-btn @click="selectContributor = index"
                    ><!--就是在這個btn，按下去就把當前index的值傳進去lineChart裡-->
                    <v-img
                      alt="user"
                      :src="userImg[index]"
                      height="40"
                      width="40"
                    />
                  </v-btn>
                </v-list-item-avatar>
                <v-list-item-content>
                  <div class="justify-space-around">
                    <v-list-item-title class="text-left py-0">
                      <a
                        class="text-subtitle-1 font-weight-regular"
                        target="aboutblank"
                        :href="item.author.html_url"
                        >{{ item.author.login }}</a
                      >
                    </v-list-item-title>
                    <div class="d-flex justify-space-around">
                      <v-list-item-subtitle class="text-left py-0"
                        ><a
                          class="grey--text"
                          target="aboutblank"
                          :href="item.commitsHtmlUrl"
                        >
                          commits {{ item.total }}</a
                        ></v-list-item-subtitle
                      >
                      <v-list-item-subtitle class="text-left py-0 green--text"
                        >{{ item.totalAdditions }} ++
                      </v-list-item-subtitle>
                      <v-list-item-subtitle class="text-left py-0 red--text"
                        >{{ item.totalDeletions }} --
                      </v-list-item-subtitle>
                    </div>
                  </div>
                </v-list-item-content>
              </v-list-item>
            </template>
          </v-virtual-scroll>
        </v-card>
      </v-col>

      <v-col>
        <v-card max-width="800" class="my-2" height="550">
          <ve-line
            height="550px"
            width="800px"
            :legend-visible="false"
            :loading="loading"
            :y-axis="yaxis"
            :data="lineChartData[selectContributor]"
            :settings="chartSettings"
            :colors="['#fb8532']"
          ></ve-line>
        </v-card>
      </v-col>

      <!-- 在lineChartData那邊改一下丟進去的東西應該就跑得出圖ㄌ -->
    </v-row>
    <ve-pie
      v-if="selectedChart == 'pie'"
      height="600px"
      :data="otherChartData"
      :settings="pieChartSettings"
    ></ve-pie>
    <ve-histogram
      v-if="selectedChart == 'histogram'"
      :data="otherChartData"
    ></ve-histogram>
  </v-container>
</template>

<script lang="ts">
import Vue from "vue";
import { getContributeInfo } from "@/apis/repoInfo";

interface Rows {
  ws: string;
  c: number;
  a: number;
  d: number;
}

interface ChartData {
  rows: any[];
  columns: string[];
}

export default Vue.extend({
  props: {
    repoId: Number,
    items:{
      type : Object
    }
  },
  data() {
    return {
      selectContributor: 0,
      loading: false,
      selected: "commit",
      selectedChart: "line",
      activities: [
        { mode: "c", name: "commit", lineHight: 40 },
        { mode: "a", name: "additions", lineHight: 10000 },
        { mode: "d", name: "deletions", lineHight: 1000 },
      ],
      chartActivities: [
        { name: "line" },
        { name: "pie" },
        { name: "histogram" },
      ],
      yaxis: {
        max: 40,
      },
      data: [] as { author: string; total: number; weeks: Rows[] }[],
      userImg: [] as string[],
      chartSettings: {
        labelMap: {
          ws: "day",
          c: "commits",
          a: "additions",
          d: "deletions",
        },
        dimension: ["ws"],
        area: true,
      },
      pieChartSettings: { radius: 160, offsetY: 300 },
      lineChartData: [] as ChartData[],
      otherChartData: { columns: ["author", "commit"], rows: [] } as ChartData,
    };
  },
  methods: {
    changeInfoData(activities: any) {
      this.loading = true;
      this.lineChartData = [];
      this.selected = activities.name;
      this.yaxis.max = activities.lineHight;
      this.data.forEach((item, index) => {
        this.lineChartData.push({
          rows: item.weeks,
          columns: ["ws", activities.mode],
        });
      });
      this.loading = false;
      switch (this.selected) {
        case "commit":
          this.otherChartData.columns = ["author", "commit"];
          break;
        case "additions":
          this.otherChartData.columns = ["author", "additions"];
          break;
        case "deletions":
          this.otherChartData.columns = ["author", "deletions"];
          break;
      }
    },
    changeChartType(chartActivities: any) {
      this.selectedChart = chartActivities.name;
    },
    getContributeInfo() {
      getContributeInfo(this.repoId)
        .then((response) => {
          this.data = response.data;
          this.data.forEach((item, index) => {
            this.lineChartData.push({
              rows: item.weeks,
              columns: ["ws", "c"],
            });
            this.otherChartData.rows.push({
              author: response.data[index].author.login,
              commit: response.data[index].total,
              additions: response.data[index].totalAdditions,
              deletions: response.data[index].totalDeletions,
            });
            if (response.data[index].author.avatar_url === null) {
              this.userImg.push("https://i.stack.imgur.com/frlIf.png");
            } else {
              this.userImg.push(response.data[index].author.avatar_url);
            }
          });
        })
        .catch((err) => {
          alert("系統錯誤");
        });
    },
  },
  mounted() {
    this.getContributeInfo();
  },
});
</script>

<style lang="scss">
a {
  text-decoration: none;
}
.btn_label {
  margin-right: 5px;
  font-size: large;
}
</style>
