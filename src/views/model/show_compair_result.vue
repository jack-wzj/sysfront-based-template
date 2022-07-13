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
        <el-col :span="8">
            <div id="lstm" :class="className" :style="{ height: height, width: width }"></div>
        </el-col>
      <el-col :span="8">
      <div id="capsnet" :class="className" :style="{ height: height, width: width }"></div>
      </el-col>
      <el-col :span="8">
      <div id="fsnet" :class="className" :style="{ height: height, width: width }"></div>
      </el-col>
    </el-row>

    <el-row>
    <el-table :data="results.slice((currentPage - 1) * pagesize, currentPage * pagesize)" 
      stripe border style="width: 100%" max_height="450" :default-sort="{ prop:
      'id', order: 'descending' }">
      <el-table-column type="index" label="ID" width="40"> </el-table-column>
    <el-table-column align="center" label="LS-LSTM">  <!--  v-if="'LS-LSTM' in results.keys()" -->
        <el-table-column prop="LS-LSTM.type" label="预测结果"></el-table-column>
      <el-table-column prop="LS-LSTM.score" label="置信度"></el-table-column>
      <template slot-scope="scope">
          <span>{{ scope.row.score }}%</span>
        </template>
    </el-table-column>
    <el-table-column align="center" label="Fs-Net">
        <el-table-column prop="Fs-Net.type" label="预测结果"></el-table-column>
      <el-table-column prop="Fs-Net.score" label="置信度"></el-table-column>
      <template slot-scope="scope">
          <span>{{ scope.row.score }}%</span>
        </template>
    </el-table-column>
    <el-table-column align="center" label="LS-CapsNet">
        <el-table-column prop="LS-CapsNet.type" label="预测结果"></el-table-column>
      <el-table-column prop="LS-CapsNet.score" label="置信度"></el-table-column>
      <template slot-scope="scope">
          <span>{{ scope.row.score }}%</span>
        </template>
    </el-table-column>
    <el-table-column align="center" props="final_tags" label="联合预测">
        <template slot-scope="scope">
          <el-tag >{{final_tags[scope.$index]}}</el-tag>
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
      lstm_pieChart: null,
      capsnet_pieChart: null,
      fsnet_pieChart: null,
      dataSize: {},
      final_tags: [],
    };
  },
  beforeDestroy() {
    if (this.lstm_pieChart) {
        this.lstm_pieChart.dispose();
        this.lstm_pieChart = null;
    }
    if (this.capsnet_pieChart) {
        this.capsnet_pieChart.dispose();
        this.capsnet_pieChart = null;
    }
    if (!this.fsnet_pieChart) {
        this.fsnet_pieChart.dispose();
        this.fsnet_pieChart = null;
    }
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
      let data = {"LS-LSTM": {}, "LS-CapsNet": {}, "Fs-Net": {}};

      this.results.forEach((file) => {
        // 计算final_tags
        if (file["LS-LSTM"].type === file["Fs-Net"].type)
            this.final_tags.push(file["LS-LSTM"].type);
        else if (file["LS-LSTM"].type === file["LS-CapsNet"].type)
            this.final_tags.push(file["LS-LSTM"].type);
        else if (file["LS-CapsNet"].type === file["Fs-Net"].type)
            this.final_tags.push(file["LS-CapsNet"].type);
        else if (parseFloat(file["LS-LSTM"].score) >= parseFloat(file["Fs-Net"].score))
            this.final_tags.push(file["LS-LSTM"].type);
        else
            this.final_tags.push( parseFloat(file["LS-LSTM"].score) >= parseFloat(file["Fs-Net"].score) ? file["LS-LSTM"].type : file["Fs-Net"].type);
        // 遍历data中的每一个key
        for (let key in data) {
            if (data[key][file[key].type]) {
                data[key][file[key].type]++;
            }
            else
                data[key][file[key].type] = 1;
        }
      });

      this.dataSize = {
        "LS-LSTM": [],
        "LS-CapsNet": [],
        "Fs-Net": [],
      };
      for (let model in data) {
        for (let key in data[model]) {
          this.dataSize[model].push({
            name: key,
            value: data[model][key],
          });
        }
      }
      console.log("dataSize:", this.dataSize);

        // 计算最终的标签
        for(let i = 0; i < this.results.length; i++){

        }
    },
    initChart() {
      this.lstm_pieChart = echarts.init(document.getElementById("lstm"));
      this.capsnet_pieChart = echarts.init(document.getElementById("capsnet"));
      this.fsnet_pieChart = echarts.init(document.getElementById("fsnet"));
      var lstm_options = {
        series: [
          {
            type: "pie",
            // radius: "70%",
            data:this.dataSize["LS-LSTM"],
            roseType: 'angle',
          },
        ],
      };
      var capsnet_options = {
        series: [
          {
            type: "pie",
            // radius: "70%",
            data:this.dataSize["LS-CapsNet"],
            roseType: 'angle',
          },
        ],
      };
      var fsnet_options = {
        series: [
          {
            type: "pie",
            // radius: "70%",
            data:this.dataSize["Fs-Net"],
            roseType: 'angle',
          },
        ],
      };
      this.lstm_pieChart.setOption(lstm_options);
      this.capsnet_pieChart.setOption(capsnet_options);
      this.fsnet_pieChart.setOption(fsnet_options);
      // window.onresize = this.lstm_pieChart.resize;
    },
    echartResize() {
      window.addEventListener("resize",() => {
        this.lstm_pieChart.resize();
        this.capsnet_pieChart.resize();
        this.fsnet_pieChart.resize();
      });
    },

  },
};


</script>

