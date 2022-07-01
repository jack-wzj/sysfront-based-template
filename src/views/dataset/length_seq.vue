<template>
  <div>
    <el-row>
      <el-button type="primary"  icon="el-icon-arrow-left" @click="backto()" plain>上一页</el-button>
    </el-row>
    <el-row>
      <el-table
        :data="
          files.slice((currentPage - 1) * pagesize, currentPage * pagesize)
        "
        stripe
        border
        style="width: 100%"
        max_height="450"
        :default-sort="{ prop: 'id', order: 'descending' }"
      >
        <el-table-column align="center" width="130px" prop="name" label="标签">
          <template slot-scope="scope">
            <el-tag v-if="scope.row.name.split('_')[0] === 'tls'">TLS</el-tag>
            <el-tag
              type="success"
              v-else-if="scope.row.name.split('_')[0] === 'http'"
              >HTTP</el-tag
            >
            <el-tag
              type="warning"
              v-else-if="scope.row.name.split('_')[0] === 'tcp'"
              >TCP</el-tag
            >
            <el-tag type="info" v-else>Else</el-tag>
          </template>
        </el-table-column>
        <el-table-column align="center" prop="name, dir" label="文件名">
          <template slot-scope="scope">
            <el-link v-if="scope.row.dir===true" @click="jumpto(scope.row.name)">{{ scope.row.name }}</el-link>
            <span v-else>{{ scope.row.name }}</span>
          </template>
        </el-table-column>
        <el-table-column align="center" prop="time" label="更新时间">
          <template slot-scope="scope">
            <i class="el-icon-time" />
            <span>{{ scope.row.time }}</span>
          </template>
        </el-table-column>
        <el-table-column align="center" prop="size" label="文件大小">
        </el-table-column>
      </el-table>
    </el-row>
    <el-row>
      <!-- <el-pagination
        @current-change="handleCurrentChange"
        :current-page="currentPage"
        :page-size="pagesize"
        :total="files.length"
      >
      </el-pagination> -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="currentPage"
        :page-sizes="[10, 20]"
        :page-size="pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="files.length"
      >
      </el-pagination>
    </el-row>
  </div>
</template>

<script>
export default {
  data() {
    return {
      rootdir: "/dataset",
      files: [],
      currentPage: 1,
      pagesize: 10,
      table_reload: false,
    };
  },
  mounted: function () {
    this.ShowFiles();
  },
  methods: {
    // 表格展示rootdir下的所有文件信息
    ShowFiles() {
      this.$axios
        .get("http://211.65.197.130:8099/findAllFile", {
          params: {
            root: this.rootdir,
          },
        })
        .then((resp) => {
          // console.log(resp);
          if (resp.data.code === 200) {
            this.files = resp.data.data;
            console.log(this.files);
          }
        })
        .catch(function (error) {
          alert("加载失败");
          console.log(error);
          window.location.reload();
        });
    },
    // 后退按钮，返回上级目录更新表格
    backto() {
      let index = this.rootdir.lastIndexOf('/');
      if(index === 0){
        return
      }
      this.rootdir = this.rootdir.substring(0,index);
      this.ShowFiles();
    },
    jumpto(dirname) {
      this.rootdir += "/" + dirname,
      this.ShowFiles();
    },
    handleSizeChange: function (size) {
      this.pagesize = size;
    },
    handleCurrentChange: function (currentPage) {
      this.currentPage = currentPage;
    },
  },
};
</script>


