<template>
  <v-dialog v-model="dialog" width="600">
    <template v-slot:activator="{ on, attrs }">
      <v-card outlined v-on="on" v-bind="attrs">
        <v-card-title class="mr-2">{{attribute}}</v-card-title>
        <span>{{getSingleMeasureDataValue(callMeasureValue)}}</span>
      </v-card>
    </template>

    <v-card class="mx-auto text-left">
      <v-container>
        <v-row dense>
          <v-col v-for="(item, key) in codeSmells" :key="key" cols="12">
            <v-card>
              <div>
                <div>
                  <v-card-title>{{ key }}</v-card-title>

                  <v-card
                    class="mt-1"
                    color="#F2DEDE"
                    v-for="(codeSmell, i) in item"
                    :key="i"
                    :href="getUrl(codeSmell.key)"
                  >
                    <v-card-text style="color: black; font-weight: bold"
                      ><v-chip
                        class="ma-2"
                        :color="colorMapper[codeSmell.severity]"
                        text-color="white"
                      >
                        {{ codeSmell.severity }} </v-chip
                      >{{ codeSmell.message }}</v-card-text
                    >
                  </v-card>
                </div>
              </div>
            </v-card>
          </v-col>
        </v-row>
      </v-container>
    </v-card>
  </v-dialog>
</template>


<script lang="ts">
import Vue from "vue";
import { getSonarqubeInfo, getSonarqubeCodeSmell } from "@/apis/repoInfo";

interface Measure {
  metric: string;
  component: string;
  value: string;
  bestValue: boolean;
}

interface CodeSmell {
  total: number;
  issues: Issue[];
}

interface Issue {
  key: string;
  severity: string;
  component: string;
  line: number;
  message: string;
}

export default Vue.extend({
  props: {
    repoId: Number,
    attribute: String,
    callMeasureValue: String
  },
  data() {
    return {
      measures: [] as Array<Measure>,
      projectName: String,
      codeSmells: {} as CodeSmell,
      colorMapper: {
        BLOCKER: "red",
        CRITICAL: "red",
        MAJOR: "red",
        MINOR: "green",
        INFO: "secondary",
      },
    };
  },
  created() {
    this.getSonarqubeInfo();
  },
  methods: {
    async getSonarqubeInfo() {
      const data = (await getSonarqubeInfo(this.repoId)).data;
      this.measures = data["measures"];
      this.projectName = data["projectName"];
      this.codeSmells = (await getSonarqubeCodeSmell(this.repoId)).data;
    },
    getSingleMeasureDataValue(measureName: string): string | undefined {
      const result = this.measures.find(
        (measure) => measure.metric === measureName
      );
      return result?.value;
    },
    getUrl(key: string) {
      return `http://zxjte9411.ml:9000/project/issues?id=PMS_109&open=${key}`;
    },
  },
});
</script>

<style></style>