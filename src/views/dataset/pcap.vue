<template>
  <div>
    <el-row justify="space-between" type="flex">
      <el-col :span="3">
        <el-button type="primary" icon="el-icon-arrow-left" @click="backto()" plain
          >上一页</el-button
        >
      </el-col>
      <el-col>
        <el-card style="float: right" :body-style="{ padding: '0px' }">
          <el-select
            placeholder="粒度"
            v-model="target_proto"
          >
            <el-option
              v-for="item in ['tcp', 'tls']"
              :label="item"
              :value="item"
              :key="item"
            ></el-option>
          </el-select>
          <el-button @click="getLenFeature()">构造长度特征数据</el-button>
        </el-card>
      </el-col>
    </el-row>
    <el-row>
      <el-table
        :data="
          pcaps.slice((currentPage - 1) * pagesize, currentPage * pagesize)
        "
        stripe
        border
        style="width: 100%"
        max_height="450"
        :default-sort="{ prop: 'id', order: 'descending' }"
        @selection-change="handleSelectionChange"
      >
        <el-table-column
          align="center"
          width="50px"
          prop="dir"
          label="选择"
          type="selection"
          :selectable="checkSelectable"
        >
        </el-table-column>
        <el-table-column align="center" prop="name, dir" label="文件名">
          <template slot-scope="scope">
            <el-link
              v-if="scope.row.dir === true"
              @click="jumpto(scope.row.name)"
              >{{ scope.row.name }}</el-link
            >
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
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="currentPage"
        :page-sizes="[10, 20]"
        :page-size="pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="pcaps.length"
      >
      </el-pagination>
    </el-row>
  </div>
</template>

<script>
import axios from "axios";
export default {
  data() {
    return {
      rootdir: "/pcapDir",
      target_proto: null,
      pcaps: [],
      checkedPcaps: [],
      currentPage: 1,
      pagesize: 10,
      pcap_checks: null,
      resDir: null,
    };
  },
  mounted: function () {
    window.Vue = this;
    this.loadPcaps();
  },
  methods: {
    loadPcaps() {
      this.$axios
        .get("http://211.65.197.130:8099/findAllFile", {
          params: {
            root: this.rootdir,
          },
        })
        .then((resp) => {
          // console.log(resp);
          if (resp.data.code === 200) {
            this.pcaps = resp.data.data;
            // this.updateTimes = resp.data.time
            this.pcap_checks = new Array(this.pcaps.length).fill("false");
            console.log(this.pcaps);
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
      let index = this.rootdir.lastIndexOf("/");
      if (index === 0) {
        return;
      }
      this.rootdir = this.rootdir.substring(0, index);
      this.loadPcaps();
    },
    jumpto(dirname) {
      this.rootdir += "/" + dirname;
      this.loadPcaps();
    },
    handleSizeChange: function (size) {
      this.pagesize = size;
    },
    handleCurrentChange: function (currentPage) {
      this.currentPage = currentPage;
    },
    handleSelectionChange(val) {
      this.checkedPcaps = val;
    },
    checkSelectable(row, index) {
      return row.dir === false;
    },
    open() {
      this.$alert("特征提取成功! 名称: " + this.resDir, "成功提示", {
        confirmButtonText: "确定",
        callback: (action) => {
          this.$message({
            type: "info",
            message: this.resDir,
          });
        },
      });
    },
    getLenFeature() {
      let pcap_checks = this.checkedPcaps;
      let pcap_names = [];
      for (let i = 0; i < pcap_checks.length; i++) {
        pcap_names.push(pcap_checks[i].name);
      }
      let index = this.rootdir.indexOf("/", 1) + 1;
      let tmp_dir = this.rootdir.substring(index);
      let target_proto = this.target_proto;

      console.log(pcap_names, tmp_dir, target_proto);
      // post request
      let params = new FormData();
      params.append("tmpDir", tmp_dir);
      params.append("targetProto", target_proto);
      params.append("targetDir", pcap_names);
      axios
        .post("http://211.65.197.130:8099/getLenSeqFromPcap", params)
        .then((resp) => {
          console.log(resp);
          if (resp.data.code === 200) {
            this.resDir = resp.data.data;
            this.open();
          }
        })
        .catch(function (error) {
          alert("加载失败");
          console.log(error);
          // window.location.reload();
        });
    },
  },
};
</script>