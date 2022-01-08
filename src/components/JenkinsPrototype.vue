<template>
  <div id="app">
    <v-app id="inspire">
      <v-card>
        <v-card-title>
          {{ info.name }}
        </v-card-title>
        <v-card-subtitle class="text-left font-weight-bold">{{
          info.description
        }}</v-card-subtitle>
        <v-card-subtitle class="text-left font-weight-bold"
          >Score: {{ info.healthReport[0].score}}</v-card-subtitle
        >
        <v-divider></v-divider>
        <v-card-subtitle class="text-left font-weight-bold">
          {{ info.healthReport[0].description }}
        </v-card-subtitle>

        <v-data-table :headers="headers" :items="buildIssue.builds">
          <template v-slot:[`item.result`]="{ item }">
            <v-chip :class="resultColor[item.result].color">
              {{ item.result }}
            </v-chip>
          </template>
        </v-data-table>
      </v-card>
    </v-app>
  </div>
</template>


<script lang="ts">
import Vue from "vue";
import { getJenkinsInfoAsync, getJenkinsJobIssue } from "@/apis/repoInfo";

interface Builds{
  result: string;
  displayName: string;
  number: number;
  duration: number;
  timestamp: number;
  //////////////////
  dateFormat: Date;
}

interface JobIssue{
  builds: Builds[];
}

interface HealthReport {
  description: string;
  score: number;
}
interface Info {
  desciption: string;
  name: string;
  healthReport: HealthReport[];
}
export default Vue.extend({
  props: {
    repoId: Number,
  },
  data() {
    return {
      info: {} as Info,
      buildIssue: {} as JobIssue,
      headers: [
        {
          text: "建置歷程",
          align: "start",
          sortable: false,
          value: "displayName",
        },
        { text: "建置結果", value: "result" },
        { text: "建置時間", value: "dateFormat" },
        { text: "花費時間(秒)", value: "duration" },
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
  created() {
    //console.log(this.repoId);
    this.getJenkinsInfoAsync();
    this.getJenkinsJobIssue();
  },
  methods: {
    async getJenkinsInfoAsync() {
      //console.log("running");
      this.info = (await getJenkinsInfoAsync(this.repoId)).data;
      //console.log(this.info);
    },
    async getJenkinsJobIssue(){
      this.buildIssue = (await getJenkinsJobIssue(this.repoId)).data;
      this.buildIssue.builds.forEach(issue => {
        issue.duration = issue.duration/1000;
        issue.dateFormat = new Date(issue.timestamp);
        //console.log(issue.dateFormat);
      });
      //console.log(this.buildIssue.builds[0].dateFormat);
    }
  },
});
</script>
