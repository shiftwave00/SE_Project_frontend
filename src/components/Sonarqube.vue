<template>
  <div>
    <v-row class="mt-5 justify-center">
      <v-col>
        <v-card outlined>
          <v-container fluid>
            <v-row>
              <v-col class="text-left"
                ><span>{{ projectName }}</span></v-col
              >
            </v-row>
            <v-row>
              <v-col>
                <SonarSmellAndBug
                  :repoId="repoId"
                  attribute="Bugs"
                  callMeasureValue="bugs"
                />
              </v-col>
              <v-col>
                <SonarSheet
                  :repoId="repoId"
                  attribute="Vulnerabilities"
                  callMeasureValue="vulnerabilities"
                />
              </v-col>
            </v-row>
            <v-row>
              <v-col>
                <SonarSmellAndBug
                  :repoId="repoId"
                  attribute="Code Smells"
                  callMeasureValue="code_smells"
                />
              </v-col>
              <v-col>
                <SonarSheet
                  :repoId="repoId"
                  attribute="Coverage"
                  callMeasureValue="coverage"
                />
              </v-col>
            </v-row>
            <v-row>
              <v-col>
                <SonarSheet
                  :repoId="repoId"
                  attribute="Duplications"
                  callMeasureValue="duplicated_lines_density"
                />
              </v-col>
            </v-row>
          </v-container>
        </v-card>
      </v-col>
    </v-row>
  </div>
</template>

<script lang="ts">
import Vue from "vue";
import SonarSmellAndBug from "@/components/Sonar_SmellsAndBug.vue";
import SonarSheet from "@/components/Sonar_Sheet.vue";
import { getSonarqubeInfo } from "@/apis/repoInfo";

export default Vue.extend({
  props: {
    repoId: Number,
  },
  components: {
    SonarSmellAndBug,
    SonarSheet,
  },
  data() {
    return {
      //measures: [] as Array<Measure>,
      projectName: String,
      //codeSmells: {} as CodeSmell
    };
  },
  created() {
    this.getSonarqubeInfo();
  },
  methods: {
    async getSonarqubeInfo() {
      const data = (await getSonarqubeInfo(this.repoId)).data;
      //this.measures = data["measures"];
      this.projectName = data["projectName"];
      //this.codeSmells = (await getSonarqubeCodeSmell(this.repoId)).data;
    },
  },
});
</script>

<style></style>
