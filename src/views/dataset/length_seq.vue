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
        <el-table-column align="center" prop="id" label="操作">
          <template slot-scope="scope">
            <el-button type="primary" size="mini" @click="downFile(scope.row)">下载</el-button>
          </template>
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
    
  
    <el-dialog
    title="正在下载，请等待"
    :visible.sync="fileDown.loadDialogStatus"
    :close-on-click-modal="false"
    :close-on-press-escape="false"
    :show-close="false"
    width="20%"
  >
    <div style="text-align: center">
      <el-progress
        type="circle"
        :percentage="fileDown.percentage"
      ></el-progress>
    </div>
    <div slot="footer" class="dialog-footer">
      <el-button type="primary" @click="downClose">取消下载</el-button>
    </div>
  </el-dialog>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  data() {
    return {
      rootdir: "/dataset/Length-Sequnce",
      files: [],
      currentPage: 1,
      pagesize: 10,
      table_reload: false,
      
      fileDown: {
      loadDialogStatus: false, //弹出框控制的状态
      percentage: 0, //进度条的百分比
      source: {}, //取消下载时的资源对象
      },
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
    // 下载文件
    downFile(row) {
      //这里放参数
      var param = {fileDir: this.rootdir + '/' + row.name};
      this.fileDown.loadDialogStatus = true;
      this.fileDown.percentage = 0;
      const instance = this.initInstance();
      instance({
        method: "get",
        withCredentials: false,
        url: "http://211.65.197.130:8099/download",
        params: param,
        responseType: "blob",
      })
        .then((res) => {
          this.fileDown.loadDialogStatus = false;
          console.info(res);
          const content = res.data;
          if (content.size == 0) {
            this.loadClose();
            this.$alert("下载失败");
            return;
          }
          const blob = new Blob([content]);
          const fileName = row.name; //替换文件名
          // const fileName = "fileName"
          if ("download" in document.createElement("a")) {
            // 非IE下载
            const elink = document.createElement("a");
            elink.download = fileName;
            elink.style.display = "none";
            elink.href = URL.createObjectURL(blob);
            document.body.appendChild(elink);
            elink.click();
            setTimeout(function () {
              URL.revokeObjectURL(elink.href); // 释放URL 对象
              document.body.removeChild(elink);
            }, 100);
          } else {
            // IE10+下载
            navigator.msSaveBlob(blob, fileName);
          }
        })
        .catch((error) => {
          this.fileDown.loadDialogStatus = false;
          console.info(error);
        });
    },
    initInstance() {
      var _this = this;
      //取消时的资源标记
      this.fileDown.source = this.$axios.CancelToken.source();
      const instance = this.$axios.create({
        //axios 这个对象要提前导入 或者替换为你们全局定义的
        onDownloadProgress: function (ProgressEvent) {
          const load = ProgressEvent.loaded;
          const total = ProgressEvent.total;
          const progress = (load / total) * 100;
          console.log("progress=" + progress);
          //一开始已经在计算了 这里要超过先前的计算才能继续往下
          if (progress > _this.fileDown.percentage) {
            _this.fileDown.percentage = Math.floor(progress);
          }
          if (progress == 100) {
            //下载完成
            _this.fileDown.loadDialogStatus = false;
          }
        },
        cancelToken: this.fileDown.source.token, //声明一个取消请求token
      });
      return instance;
    },
    downClose() {
      //中断下载
      this.$confirm(
        "点击关闭后将中断下载，是否确定关闭？",
        this.$t("button.tip"),
        {
          confirmButtonText: this.$t("button.confirm"),
          cancelButtonText: this.$t("button.cancel"),
          type: "warning",
        }
      )
        .then(() => {
          //中断下载回调
          this.fileDown.source.cancel("log==客户手动取消下载");
        })
        .catch(() => {
          //取消--什么都不做
        });
    },
  },
};
</script>


