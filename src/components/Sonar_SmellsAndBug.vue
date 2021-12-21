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
          <v-col v-for="(item, key) in recordIssues" :key="key" cols="12">
            <v-card>
              <div>
                <div>
                  <v-card-title>{{ key }}</v-card-title>

                  <v-card
                    class="mt-1"
                    color="#F2DEDE"
                    v-for="(record, i) in item"
                    :key="i"
                    :href="getUrl(record.key)"
                  >
                    <v-card-text style="color: black; font-weight: bold"
                      ><v-chip
                        class="ma-2"
                        :color="colorMapper[record.severity]"
                        text-color="white"
                      >
                        {{ record.severity }} </v-chip
                      >{{ record.message }}</v-card-text
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
import { getSonarqubeInfo, getSonarqubeCodeSmell, getSonarqubeBug,} from "@/apis/repoInfo";

interface Measure {
  metric: string;
  component: string;
  value: string;
  bestValue: boolean;
}

interface CodeSmellOrBug {
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
      recordIssues: {} as CodeSmellOrBug,
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
      if(this.callMeasureValue == "code_smells" ){
        this.recordIssues = (await getSonarqubeCodeSmell(this.repoId)).data;
      }else{
        this.recordIssues = (await getSonarqubeBug(this.repoId)).data;
      }
      
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