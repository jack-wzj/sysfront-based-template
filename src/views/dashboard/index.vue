<template>
  <div class="container" style="min-height: 100%; padding-bottom: 100px">
    <el-card class="box-card" shadow="hover" :body-style="{ padding: '6px' }">
      <div slot="header" class="clearfix">
        <span>数据集概览</span>
      </div>
      <el-row>
        <el-col :span="12">
          <el-card
            class="box-card"
            shadow="hover"
            :body-style="{ padding: '10px' }"
          >
            <div class="chart-container">
              <div slot="header" class="clearfix">
                <span style="font-size: 16px">Pcap数据</span>
                <el-button @click="jumpTo('/dataset/pcap')" style="float: right; padding: 3px 0" type="text"
                  >查看详情</el-button
                >
              </div>
              <pcapChart height="100%" width="100%" />
            </div>
          </el-card>
        </el-col>
        <el-col :span="12">
          <el-card
            class="box-card"
            shadow="hover"
            :body-style="{ padding: '10px' }"
          >
            <div class="chart-container">
              <div slot="header" class="clearfix">
                <span style="font-size: 16px">长度序列数据</span>
                <el-button @click="jumpTo('/dataset/length-sequnce')" style="float: right; padding: 3px 0" type="text"
                  >查看详情</el-button
                >
              </div>
              <chart height="100%" width="100%" />
            </div>
          </el-card>
        </el-col>
      </el-row>
    </el-card>

    <el-card class="box-card" :body-style="{ padding: '0px' }">
      <div slot="header" class="clearfix">
        <span>模型概览</span>
        <el-button @click="jumpTo('/model/application-classify')" style="float: right; padding: 3px 0" type="text"
          >查看详情</el-button
        >
      </div>
      <el-carousel height="350px">
        <el-carousel-item align="center" v-for="(item) in images" :key="item.index">
          <!-- <h3 class="small">{{ item }}</h3> -->
          <img :src="item.url" />
        </el-carousel-item>
      </el-carousel>
    </el-card>
  </div>
  
</template>

<script>
import Chart from "./dataset_chart.vue";
import pcapChart from "./pcap_chart.vue";
// import fileDownload from "@/components/fileDownload";

export default {
  name: "MixChart",
  components: { Chart, pcapChart },
  data() {
    return {
      images: [
        { url: require("@/assets/LS_CapsNet.png") },
        { url: require("@/assets/LS-LSTM.png") },
        { url: require("@/assets/Fs-Net.jpg") },
        { url: require("@/assets/N-gram.png") },
      ],
      row: "",
  
    };
  },
  methods: {
    jumpTo(val) {
      this.$router.push({
        path: val,
      });
    },
    test() {
      fileDownload.downFile();
    }
  },
};
</script>

<style scoped>
.chart-container {
  position: relative;
  width: 100%;
  height: 250px;
}
img {
  max-width: 100%;
  max-height: 100%;
}
</style>

