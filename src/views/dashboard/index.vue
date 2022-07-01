<template>
  <div class="container" style="min-height: 100%; padding-bottom: 100px">
    <el-card class="box-card" shadow="hover" :body-style="{ padding: '6px' }">
      <div slot="header" class="clearfix">
        <span>数据集概览</span>
        <el-button style="float: right; padding: 3px 0" type="text"
          >查看详情</el-button
        >
      </div>
      <el-row>
        <el-col :span="12">
        <el-card class="box-card" shadow="hover" :body-style="{padding:'10px'}">
          <el-table :data="tableData" height="250px">
            <el-table-column
              :prop="propItem.prop"
              :label="propItem.label"
              v-for="propItem in propList"
              :key="propItem.prop"
            >
              <template v-slot="{ row }">
                <span v-if="!propItem.component">{{ row[propItem.prop] }}</span>
                <component
                  v-else
                  v-bind:is="propItem.component"
                  :rowinfo="row"
                ></component>
              </template>
            </el-table-column>
          </el-table>
          </el-card>
        </el-col>
        <el-col :span="12">
        <el-card class="box-card" shadow="hover" :body-style="{padding:'10px'}">
          <div class="chart-container">
            <div slot="header" class="clearfix">
        <span style="font-size:16px">长度序列数据</span>
        </div>
            <chart height="100%" width="100%" />
          </div>
          </el-card>
        </el-col>
      </el-row>
    </el-card>

    <el-card class="box-card" :body-style="{padding:'0px'}">
      <div slot="header" class="clearfix">
        <span>模型概览</span>
        <el-button style="float: right; padding: 3px 0" type="text"
          >查看详情</el-button
        >
      </div>
      <el-carousel height="350px">
        <el-carousel-item v-for="item in images" :key="item">
          <!-- <h3 class="small">{{ item }}</h3> -->
          <img :src="item.url" />
        </el-carousel-item>
      </el-carousel>
    </el-card>
  </div>
</template>

<script>
import Chart from "./dataset_chart.vue";

export default {
  name: "MixChart",
  components: { Chart },
  data() {
    return {
      images: [
          { url: require('@/assets/LS_CapsNet.png')},
        ],

      item: "",
      propItem: "",
      propList: [
        {
          label: "姓名",
          prop: "name",
        },
        {
          label: "日期",
          prop: "date",
        },
        {
          label: "动态组件",
          prop: "lc-component",
          component: "el-tag",
        },
      ],
      row: "",
      tableData: [
        {
          date: "2016-05-02",
          name: "王小虎",
          address: "上海市普陀区金沙江路 1518 弄",
        },
        {
          date: "2016-05-04",
          name: "王小虎",
          address: "上海市普陀区金沙江路 1517 弄",
        },
        {
          date: "2016-05-01",
          name: "王小虎",
          address: "上海市普陀区金沙江路 1519 弄",
        },
      ],
    };
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

