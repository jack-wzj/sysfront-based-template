<template>
  <div>
    <el-row>
      <el-alert
    :title= "runtime"
    type="success"
    show-icon>
  </el-alert>
    </el-row>
  
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
     <el-pagination
     @current-change="handleCurrentChange"
    layout="prev, pager, next"
    :total="results.length"
    :page-size="20">
    </el-pagination>
  </div>
</template>

<script>
export default {
  mounted() {
    window.Vue = this;
  },
  data() {
    return {
      currentPage: 1,
      pagesize: 20,
      results: this.$route.query.response.data,
      runtime: "分析用时 : " + this.$route.query.response.time + "s",
    };
  },
  methods: {
     handleCurrentChange: function (currentPage) {
      this.currentPage = currentPage;
    },
  },
};
</script>

