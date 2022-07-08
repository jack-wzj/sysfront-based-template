<template>
  <div>
    <el-row>
      <el-alert
    :title= "runtime"
    type="success"
    show-icon>
  </el-alert>
    </el-row>
    <el-row>
      <div :id="id" :class="className" :style="{ height: height, width: width }"></div>
    </el-row>

    <el-row>
    <el-table :data="results.slice((currentPage - 1) * pagesize, currentPage * pagesize)" 
      stripe border style="width: 100%" max_height="450" :default-sort="{ prop:
      'id', order: 'descending' }" >
      <el-table-column type="index" label="ID" width="80"> </el-table-column>
      <el-table-column prop="type" label="预测结果"></el-table-column>
      <el-table-column prop="score" label="置信度">
        <template slot-scope="scope">
          <span>{{ scope.row.score }}%</span>
        </template>
      </el-table-column>
    </el-table>
    </el-row>
     <el-pagination
     @current-change="handleCurrentChange"
    layout="prev, pager, next"
    :total="results.length"
    :page-size="20">
    </el-pagination>
  </div>
</template>

<script>
import * as echarts from "echarts";
import resize from "./mixins/resize";

export default {
  mixins: [resize],
  props: {
    className: {
      type: String,
      default: "pieChart",
    },
    id: {
      type: String,
      default: "pieChart",
    },
    width: {
      type: String,
      default: "100%",
    },
    height: {
      type: String,
      default: "250px",
    },
  },
  mounted() {
    window.Vue = this;
    this.calSize();
    this.initChart();
    this.echartResize();
  },
  data() {
    return {
      currentPage: 1,
      pagesize: 10,
      results: this.$route.query.response.data,
      runtime: "分析用时 : " + this.$route.query.response.time + "s",
      pieChart: null,
      dataSize: [],
    };
  },
  beforeDestroy() {
    if (!this.pieChart) {
      return;
    }
    this.pieChart.dispose();
    this.pieChart = null;
  },
  watch: {
    results: {
      handler(newVal) {
        this.results = newVal;
        this.calSize();
        this.initChart();
      },
      deep: true,
    },
  },
  methods: {
     handleCurrentChange: function (currentPage) {
      this.currentPage = currentPage;
    },
    calSize() {
      // 根据files名称统计数据
      let data = {};
      this.results.forEach((file) => {
        if(data[file.type]) {
          data[file.type]++;
        } else {
          data[file.type] = 1;
        }
      });

      this.dataSize = [];
      for (let key in data) {
        this.dataSize.push({
          value: data[key],
          name: key,
        });
      }
      console.log("dataSize:", this.dataSize);
    },
    initChart() {
      this.pieChart = echarts.init(document.getElementById("pieChart"));
      var options = {
        dataset: {
          source: this.dataSize,
        },
        series: [
          {
            type: "pie",
            // radius: "70%",
            // data:this.dataSize,
            roseType: 'angle',
          },
        ],
      };
      this.pieChart.setOption(options);
      // window.onresize = this.pieChart.resize;
    },
    echartResize() {
      window.addEventListener("resize",() => {
        this.pieChart.resize();
      });
    },
    
  },
};


</script>

