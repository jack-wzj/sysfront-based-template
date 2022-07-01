<template>
  <div :id="id" :class="className" :style="{ height: height, width: width }" />
</template>

<script>
import * as echarts from "echarts";
import resize from "./mixins/resize";

export default {
  mixins: [resize],
  props: {
    className: {
      type: String,
      default: "chart",
    },
    id: {
      type: String,
      default: "chart",
    },
    width: {
      type: String,
      default: "250px",
    },
    height: {
      type: String,
      default: "250px",
    },
  },
  data() {
    return {
      files: null,
      dataNum: [],
      dataType: [],
      chart: null,
    };
  },

  watch: {
    dataNum: {
      handler(newVal) {
        this.dataNum = newVal;
        this.initChart();
      },
      deep: true,
    },
  },
  mounted() {
    window.Vue = this;
    this.getData();
    this.initChart();
  },

  beforeDestroy() {
    if (!this.chart) {
      return;
    }
    this.chart.dispose();
    this.chart = null;
  },

  methods: {
    calSize(val) {
      // 根据files名称统计数据
      let data = {};
      console.log("files:", this.files);
      this.files.forEach((file) => {
        let type_name =
          ["http", "tls", "tcp"].indexOf(file.name.split("_")[0]) === -1
            ? "other"
            : file.name.split("_")[0];
        if (data[type_name]) {
          data[type_name]++;
        } else {
          data[type_name] = 1;
        }
      });
      
      this.dataNum = [];
      for (let key in data) {
        this.dataNum.push({
          name: key,
          value: data[key],
        });
      }
      console.log("dataNum:", this.dataNum);
    },
    getData() {
      this.$axios
        .get("http://211.65.197.130:8099/findAllFile", {
          params: {
            root: "/dataset",
          },
        })
        .then((resp) => {
          // console.log(resp);
          if (resp.data.code === 200) {
            this.files = resp.data.data;
            this.calSize();
            console.log("get files:", this.files);
          }
        })
        .catch(function (error) {
          alert("加载失败");
          console.log(error);
        });
    },
    initChart() {
      this.chart = echarts.init(document.getElementById(this.id));
      var options = {
        dataset: {
          source: this.dataNum,
        },
        xAxis: {
          type: "category",
        },
        yAxis: {},
        series: [
          {
            type: "bar",
          },
        ],
      };
      this.chart.setOption(options);
    },
  },
};

// window.addEventListener("resize",function() {
//         this.chart.resize();
// });
</script>
