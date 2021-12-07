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
                <SonarSmells
                  :repoId="repoId"
                  attribute="Bugs"
                  callMeasureValue="bugs"
                />
              </v-col>
              <v-col>
                <SonarSmells
                  :repoId="repoId"
                  attribute="Vulnerabilities"
                  callMeasureValue="vulnerabilities"
                />
              </v-col>
            </v-row>
            <v-row>
              <v-col>
                <SonarSmells
                  :repoId="repoId"
                  attribute="Code Smells"
                  callMeasureValue="code_smells"
                />
              </v-col>
              <v-col>
                <SonarSmells
                  :repoId="repoId"
                  attribute="Coverage"
                  callMeasureValue="coverage"
                />
              </v-col>
            </v-row>
            <v-row>
              <v-col>
                <SonarSmells
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
import SonarSmells from "@/components/Sonar_Smells.vue";
import { getSonarqubeInfo } from "@/apis/repoInfo";

export default Vue.extend({
  props: {
    repoId: Number,
  },
  components: {
    SonarSmells,
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
    }
  },
});
</script>

<style></style>
