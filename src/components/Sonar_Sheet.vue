<template>
  <v-dialog v-model="dialog" width="600">
    <template v-slot:activator="{ on, attrs }">
      <v-card outlined v-on="on" v-bind="attrs">
        <v-card-title class="mr-2">{{ attribute }}</v-card-title>
        <span>{{ getSingleMeasureDataValue(callMeasureValue) }}</span>
      </v-card>
    </template>

    <v-card class="mx-auto text-left">
      <v-container>
        <v-row dense>
          <v-col cols="12">
            <v-card>
              <div>
                <div>
                  <v-card-title>{{ coverage.baseComponent.key }}</v-card-title>
                  <v-card-subtitle
                    >{{
                      getSingleMeasureDataValue(callMeasureValue)
                    }}
                    %</v-card-subtitle
                  >
                  <div class="container">
                    <BubbleChart
                      :chartdata="chartData"
                      :options="coverageOptions"
                      v-if="attribute == 'Coverage'"
                    />
                    <BubbleChart
                      :chartdata="chartData"
                      :options="duplicationsOptions"
                      v-if="attribute == 'Duplications'"
                    />
                  </div>
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
import BubbleChart from "@/components/BubbleChart.vue";
import {
  getSonarqubeInfo,
  getCoverageSheet,
  getDuplicationSheet,
} from "@/apis/repoInfo";

interface Measure {
  metric: string;
  component: string;
  value: string;
  bestValue: boolean;
}

interface Measures {
  metric: string;
  value: string;
}

interface Components {
  key: string;
  qualifier: string;
  measures: Measures[];
}

interface SonarSheet {
  baseComponent: Components;
  components: Components[];
}

export default Vue.extend({
  props: {
    repoId: Number,
    attribute: String,
    callMeasureValue: String,
  },

  components: {
    BubbleChart,
  },

  data() {
    return {
      measures: [] as Array<Measure>,
      coverage: {} as SonarSheet,
      duplications: {} as SonarSheet,
      chartData: {
        datasets: [
          {
            label: [] as string[],
            data: [] as { x: number; y: number; r: number }[],
            backgroundColor: "rgba(0, 0, 200, 0.4)",
            hoverRadius: 1,
            hitRadius: 0,
            hoverBorderWidth: 0,
          },
        ],
      },
      coverageOptions: {
        tooltips: {
          enabled: true,
          mode: "single",
          callbacks: {
            label: function (tooltipItems: any, data: any) {
              return [
                "Complexity : " + tooltipItems.xLabel + " ",
                "Coverage : " + tooltipItems.yLabel + " ",
                "Uncover line : " +
                  data.datasets[tooltipItems.datasetIndex].data[
                    tooltipItems.index
                  ].r +
                  " ",
              ];
            },
            title: function (tooltipItems: any, data: any) {
              return [
                data.datasets[tooltipItems[0].datasetIndex].label[
                  tooltipItems[0].index
                ],
              ];
            },
          },
        },
        scales: {
          yAxes: [
            {
              scaleLabel: {
                display: true,
                labelString: "Coverage",
              },
              ticks: {
                beginAtZero: false,
                max: 100,
                min: -10,
              },
              gridLines: {
                display: true,
              },
            },
          ],
          xAxes: [
            {
              scaleLabel: {
                display: true,
                labelString: "Complexity",
              },
              ticks: {
                beginAtZero: true,
                max: 30,
              },
              gridLines: {
                display: false,
              },
            },
          ],
        },
        legend: {
          display: false,
        },
      },
      duplicationsOptions: {
        tooltips: {
          enabled: true,
          mode: "single",
          callbacks: {
            label: function (tooltipItems: any, data: any) {
              return [
                "Lines of Code : " + tooltipItems.xLabel + " ",
                "Duplicated Lines : " + tooltipItems.yLabel + " ",
                "Duplicated Blocks : " +
                  data.datasets[tooltipItems.datasetIndex].data[
                    tooltipItems.index
                  ].r +
                  " ",
              ];
            },
            title: function (tooltipItems: any, data: any) {
              return [
                data.datasets[tooltipItems[0].datasetIndex].label[
                  tooltipItems[0].index
                ],
              ];
            },
          },
        },
        scales: {
          yAxes: [
            {
              scaleLabel: {
                display: true,
                labelString: "Duplicated Lines",
              },
              ticks: {
                beginAtZero: true,
                max: 20,
                min: -10,
              },
              gridLines: {
                display: false,
              },
            },
          ],
          xAxes: [
            {
              scaleLabel: {
                display: true,
                labelString: "Lines of Codes",
              },
              ticks: {
                beginAtZero: true,
                max: 200,
              },
              gridLines: {
                display: true,
              },
            },
          ],
        },
        legend: {
          display: false,
        },
      },
      responsive: true,
      maintainAspectRatio: false,
      height: 200,
    };
  },

  created() {
    this.getSonarqubeInfo();
  },

  methods: {
    async getSonarqubeInfo() {
      const data = (await getSonarqubeInfo(this.repoId)).data;
      this.measures = data["measures"];
      this.coverage = (await getCoverageSheet(this.repoId)).data;
      this.duplications = (await getDuplicationSheet(this.repoId)).data;
      if (this.attribute === "Duplications") {
        this.setChartDataOfDuplication();
      } else {
        this.setChartDataOfCoverage();
      }
    },
    getSingleMeasureDataValue(measureName: string): string | undefined {
      const result = this.measures.find(
        (measure) => measure.metric === measureName
      );
      return result?.value;
    },
    setChartDataOfCoverage() {
      for (let i = 0; i < this.coverage.components.length; i++) {
        if (
          this.coverage.components[i].measures.length === 3 &&
          this.coverage.components[i].key != null
        ) {
          this.chartData.datasets[0].label.push(
            this.coverage.components[i].key.valueOf()
          );
          this.chartData.datasets[0].data.push({
            x: parseFloat(this.coverage.components[i].measures[0].value),
            y: parseFloat(this.coverage.components[i].measures[1].value),
            r: parseFloat(this.coverage.components[i].measures[2].value),
          });
        }
      }
    },
    setChartDataOfDuplication() {
      for (let i = 0; i < this.duplications.components.length; i++) {
        if (
          this.duplications.components[i].measures.length === 3 &&
          this.duplications.components[i].key != null
        ) {
          this.chartData.datasets[0].label.push(
            this.duplications.components[i].key.valueOf()
          );
          this.chartData.datasets[0].data.push({
            x: parseFloat(this.duplications.components[i].measures[0].value),
            y: parseFloat(this.duplications.components[i].measures[1].value),
            r: parseFloat(this.duplications.components[i].measures[2].value),
          });
        }
      }
    },
  },
});
</script>

<style></style>