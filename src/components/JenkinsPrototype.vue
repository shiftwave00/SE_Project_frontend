<template>
  <div id="app">
    <v-app id="inspire">
      <v-card>
        <v-card-title>
          Jenkins_JobName
          <v-spacer></v-spacer>
          <v-text>Jenkins_HealthReport(Description)</v-text>
          <v-spacer></v-spacer>
          <v-text>Jenkins_HealthReport(Score)</v-text>
        </v-card-title>
        <v-divider></v-divider>
        <v-card-subtitle class="text-left font-weight-bold">
          Job_Description
        </v-card-subtitle>
        <v-data-table :headers="headers" :items="desserts" :search="search">
          <template v-slot:[`item.result`]="{ item }">
            <v-chip :class="resultColor[item.result].color">
              {{ item.result }}
            </v-chip>
          </template>
        </v-data-table>
      </v-card>
    </v-app>
  </div>
</template>>


<script lang="ts">
import Vue from "vue";
export default Vue.extend({
  data() {
    return {
      headers: [
        {
          text: "建置歷程",
          align: "start",
          sortable: false,
          value: "name",
        },
        { text: "建置結果", value: "result" },
        { text: "建置時間", value: "timestamp" },
        { text: "花費時間(秒)", value: "duration" },
      ],
      desserts: [
        {
          name: "#6",
          result: "FAILURE",
          timestamp: "2022年01月06日 下午8:34",
          duration: 24,
        },
        {
          name: "#5",
          result: "FAILURE",
          timestamp: "2021年12月27日 下午7:51",
          duration: 24,
        },
        {
          name: "#4",
          result: "SUCCESS",
          timestamp: "2021年12月27日 下午5:42",
          duration: 37,
        },
        {
          name: "#3",
          result: "FAILURE",
          timestamp: "2021年12月27日 下午3:03",
          duration: 23,
        },
        {
          name: "#2",
          result: "SUCCESS",
          timestamp: "2021年12月27日 下午1:55",
          duration: 67,
        },
        {
          name: "#1",
          result: "SUCCESS",
          timestamp: "2021年12月27日 下午1:21",
          duration: 49,
        },
      ],
      resultColor: {
        FAILURE: {
          color: "red lighten-4 red--text text--darken-4",
        },
        SUCCESS: {
          color: "blue lighten-4 blue--text text--darken-4",
        },
      },
    };
  },
  methods: {},
});
</script>
