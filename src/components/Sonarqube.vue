<template>
  <v-row class="mt-5 justify-center">
    <v-col cols="12" lg="10">
      <v-card elevation="2" outlined>
        <v-container fluid>
          <v-row>
            <v-col class="text-left"
              ><span>{{ projectName }}</span></v-col
            >
          </v-row>
          <v-row>
            <v-col>
              <span class="mr-2">Bugs</span>
              <v-icon>mdi-bug</v-icon>
              <v-row class="justify-center mt-2"
                ><span class="mr-2">{{ getSingleMeasureDataValue("bugs") }}</span
              ></v-row>
            </v-col>
            <v-divider vertical></v-divider>
            <v-col>
              <span class="mr-2">Vulnerabilities</span>
              <v-icon>mdi-security</v-icon>
              <v-row class="justify-center mt-2"
                ><span class="mr-2">{{ getSingleMeasureDataValue("vulnerabilities") }}</span></v-row
              >
            </v-col>
            <v-divider vertical></v-divider>
            <v-col>
              <span>Code Smells</span>
              <v-row class="justify-center mt-2"
                ><span class="mr-2">{{ getSingleMeasureDataValue("code_smells") }}</span></v-row
              >
            </v-col>
            <v-divider vertical></v-divider>
            <v-col>
              <span>Coverage</span>
              <v-row class="justify-center mt-2"
                ><span class="mr-2">{{ getSingleMeasureDataValue("coverage") }}%</span></v-row
              >
            </v-col>
            <v-divider vertical></v-divider>
            <v-col>
              <span class="mr-2">Duplications</span>
              <v-icon>mdi-content-duplicate</v-icon>
              <v-row class="justify-center mt-2"
                ><span class="mr-2">{{ getSingleMeasureDataValue("duplicated_lines_density") }}</span></v-row
              >
            </v-col>
          </v-row>
        </v-container>
      </v-card>
    </v-col>
  </v-row>
</template>

<script lang="ts">
import Vue from "vue";
import { getSonarqubeInfo } from "@/apis/repoInfo";

interface Measure {
  metric: string;
  component: string;
  value: string;
  bestValue: boolean;
}

export default Vue.extend({
  props: {
    repoId: Number,
  },
  data() {
    return {
      measures: [] as Array<Measure>,
      projectName: String
    };
  },
  created() {
    this.getSonarqubeInfo();
  },
  methods: {
    async getSonarqubeInfo() {
      const data =  (await getSonarqubeInfo(this.repoId)).data
      this.measures = data["measures"] ;
      this.projectName = data["projectName"]
    },
    getSingleMeasureDataValue(measureName: string): string | undefined {
      const result = this.measures.find((measure) => measure.metric === measureName);
      return result?.value;
    },
  },
});
</script>

<style>
</style>