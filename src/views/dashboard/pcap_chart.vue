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
      default: "pcapChart",
    },
    id: {
      type: String,
      default: "pcapChart",
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
      dataSize: [],
      pcapChart: null,
    };
  },

  watch: {
    dataSize: {
      handler(newVal) {
        this.dataSize = newVal;
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
    if (!this.pcapChart) {
      return;
    }
    this.pcapChart.dispose();
    this.pcapChart = null;
  },

  methods: {
    calSize(val) {
      // 根据files名称统计数据
      let data = {};
    //   console.log("files:", this.files);
      this.files.forEach((file) => {
        let type_name = file.name;
        if (file.size.indexOf("MB") > 0) {
          data[type_name] = parseFloat(file.size.split("\t")) * 1024;
        } else if (file.size.indexOf("GB") > 0) {
          data[type_name] = parseFloat(file.size.split("\t")) * 1024 * 1024;
        } else {
          data[type_name] = parseFloat(file.size.split("\t"));
        }
      });

      this.dataSize = [];
      for (let key in data) {
        this.dataSize.push({
          value: data[key],
          name: key,
        });
      }
    //   console.log("dataSize:", this.dataSize);
    },

    getData() {
      this.$axios
        .get("http://211.65.197.130:8099/findAllFile", {
          params: {
            root: "/dataset/Pcap",
          },
        })
        .then((resp) => {
          // console.log(resp);
          if (resp.data.code === 200) {
            this.files = resp.data.data;
            this.calSize();
            // console.log("get files:", this.files);
          }
        })
        .catch(function (error) {
          alert("加载失败");
          console.log(error);
        });
    },
    
    initChart() {
      this.pcapChart = echarts.init(document.getElementById(this.id));
      var options = {
        // dataset: {
        //   source: this.dataSize,
        // },
        series: [
          {
            type: "pie",
            // radius: "70%",
            data:this.dataSize,
            // roseType: 'angle',
          },
        ],
      };
      this.pcapChart.setOption(options);
    },
  },
};

// window.addEventListener("resize",function() {
//         this.pcapChart.resize();
// });
</script>
