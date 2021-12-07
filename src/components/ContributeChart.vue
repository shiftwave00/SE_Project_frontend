<template>
  <v-container>
    <v-row class="d-flex justify-end">
      <div class="text-center my-3">
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
                  <v-btn class="pa-0" color="white" @click="changeInfoData(item)">{{
                    item.name
                  }}</v-btn>
                </v-list-item-content>
              </v-list-item>
            </template>
          </v-list>
        </v-menu>
      </div>
    </v-row>
    <v-divider></v-divider>
    <v-row>
      <template v-for="(item, index) in data">
        <v-col lg="4" class="d-flex justify-center" :key="item.total">
          <v-card :key="index" width="450" height="380" class="my-2">
            <v-card-title class="mb-1">
              <v-avatar size="60">
                <img alt="user" :src="item.author.avatar_url" />
              </v-avatar>
              <div class="justify-space-around">
                <v-card-subtitle class="text-left py-0">
                  <a
                    class="text-subtitle-1 font-weight-regular"
                    target="aboutblank"
                    :href="item.author.html_url"
                    >{{ item.author.login }}</a
                  >
                </v-card-subtitle>
                <div class="d-flex justify-space-around">
                  <v-card-subtitle class="text-left py-0"
                    ><a class="grey--text" target="aboutblank" :href=item.commitsHtmlUrl>
                      commits {{ item.total }}</a
                    ></v-card-subtitle
                  >
                  <v-card-subtitle class="text-left py-0 green--text"
                    >{{ item.totalAdditions }} ++</v-card-subtitle
                  >
                  <v-card-subtitle class="text-left py-0 red--text"
                    >{{ item.totalDeletions }} --</v-card-subtitle
                  >
                </div>
              </div>
            </v-card-title>

            <v-divider class="mx-4"></v-divider>
            <ve-line
              height="300px"
              :legend-visible="false"
              :loading="loading"
              :y-axis="yaxis"
              :data="chartData[index]"
              :settings="chartSettings"
              :colors="['#fb8532']"
            ></ve-line>
          </v-card>
        </v-col>
      </template>
    </v-row>
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
  },
  data() {
    return {
      loading: false,
      selected: "commit",
      activities: [
        { mode: "c", name: "commit", lineHight: 40 },
        { mode: "a", name: "additions", lineHight: 10000 },
        { mode: "d", name: "deletions", lineHight: 1000 },
      ],
      yaxis: {
        max: 40,
      },
      data: {} as { author: string; total: number; weeks: Rows[] }[],
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
      chartData: [] as ChartData[],
    };
  },
  methods: {
    changeInfoData(activities: any) {
      this.loading = true;
      this.chartData = [];
      this.selected = activities.name;
      this.yaxis.max = activities.lineHight;
      this.data.forEach((item, index) => {
        this.chartData.push({
          rows: item.weeks,
          columns: ["ws", activities.mode],
        });
      });
      this.loading = false;
    },
    getContributeInfo() {
      getContributeInfo(this.repoId)
        .then((response) => {
          this.data = response.data;
          this.data.forEach((item, index) => {
            this.chartData.push({
              rows: item.weeks,
              columns: ["ws", "c"],
            });
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

<style>
a {
  text-decoration: none;
}
</style>