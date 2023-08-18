<template>
  <div v-if="!seen" class="about">
    <el-button type="primary" @click="handleClick1"
      >特征和验证数据处理</el-button
    >
    <el-button type="primary" @click="handleClick2">A7Test数据处理</el-button>
  </div>
  <div class="content">
    <div v-if="seen" class="joker">
      <el-select v-model="value" size="large" filterable placeholder="Select">
        <el-option
          v-for="item in options"
          :key="item"
          :label="item"
          :value="item"
        />
      </el-select>
      <el-button
        type="primary"
        @click="submit"
        style="margin-top: 5px; margin-left: 20px"
      >
        Primary
      </el-button>
    </div>
    <div ref="echarts" class="chart"></div>
  </div>
</template>

<script lang="ts">
import { Options, Vue } from "vue-class-component";
import * as echarts from "echarts";
import { ref, Ref } from "vue";
import axios from "axios";

interface OptionData {
  millisecond: Date;
  meterNum: string;
  fhl: number;
  pwrUp: number;
  pwrDown: number;
}
@Options({
  components: {},
})
export default class HomeView extends Vue {
  seen:boolean = false;
  input1 = ref("");
  value = ref("");
  options = ref<string[]>([]);
  results: OptionData[] = [];
  colors: string[] = [
    "#5470C6",
    "#EE6666",
    "#73C0DE",
    "#91CC75",
    "#F49F42",
    "#F39C5F",
    "#F58158",
    "#EEDD78",
    "#8D98B3",
    "#95706D",
  ]; // Predefined colors for the lines

  mounted() {
    this.getAllMeterNum();
  }

  submit() {
    let meterNum = this.value;
    axios
      .get("/api/getAllData", {
        params: {
          meterNum: meterNum,
        },
      })
      .then((res: any) => {
        console.log("res", res);
        this.results = res.data;
        this.drawScatterChart();
      })
      .catch((err: any) => {
        console.log("err", err);
      });
  }

  getAllMeterNum() {
    axios
      .get("/api/getAllMeter")
      .then((res: any) => {
        console.log("res", res);
        this.options = res.data;
      })
      .catch((err: any) => {
        console.log("err", err);
      });
  }
  handleClick1() {
    // window.location.href = "http://114.55.227.234:8081/";
    window.location.href = "http://192.168.8.2:8081/";
  }
  handleClick2() {
    this.seen = true;
  }
  drawScatterChart() {
    const chart = echarts.init(this.$refs.echarts as HTMLDivElement);
    const legendData: string[] = []; // Collect meterNum values for legend
    const series: any[] = []; // Store the data series

    // Group the results by meterNum
    const groupedResults = this.results.reduce(
      (groups: any, item: OptionData) => {
        const group = groups[item.meterNum] || [];
        group.push(item);
        groups[item.meterNum] = group;
        return groups;
      },
      {}
    );

    Object.keys(groupedResults).forEach((meterNum, index) => {
      const group = groupedResults[meterNum];

      // Generate a unique name for each legend entry
      const legendName = `meterNum ${meterNum}`;
      legendData.push(legendName);

      // Define the data for the current meterNum
      const fhlData = group.map((item: OptionData) => item.fhl);
      const pwrUpData = group.map((item: OptionData) => item.pwrUp);
      const pwrDownData = group.map((item: OptionData) => item.pwrDown);

      // Define the series data for the current meterNum
      const seriesData = [
        {
          name: legendName,
          type: "line",
          data: pwrDownData, // Use pwrDown values as the y-axis data for pwrDown line
          lineStyle: {
            color: this.colors[index % this.colors.length], // Use predefined colors cyclically
          },
        },
        {
          name: legendName,
          type: "line",
          data: pwrUpData, // Use pwrUp values as the y-axis data for pwrUp line
          lineStyle: {
            color: this.colors[index % this.colors.length], // Use predefined colors cyclically
          },
        },
      ];

      series.push(...seriesData);
    });

    const option = {
      legend: {
        data: legendData, // Display the legend entries
      },
      xAxis: {
        type: "category",
        // Use the first group's fhl values as the x-axis data
        // This assumes that all groups have the same fhl values
        data: groupedResults[Object.keys(groupedResults)[0]].map(
          (item: OptionData) => item.fhl
        ),
      },
      yAxis: {},
      series: series, // Use the generated series data
    };

    chart.setOption(option);
  }
}
</script>

<style>
.about{
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
}
.content {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  height: 100vh;
  width: 100vw;
}

.joker {
  display: flex;
  flex-direction: row;
}

.chart {
  width: 100vw;
  height: 800px;
}
</style>
