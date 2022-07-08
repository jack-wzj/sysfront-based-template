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
        :default-sort="{ prop: 'time', order: 'descending' }"
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
        <el-table-column align="center" prop="name, dir" label="文件名" sortable="">
          <template slot-scope="scope">
            <el-link
              v-if="scope.row.dir === true"
              @click="jumpto(scope.row.name)"
              >{{ scope.row.name }}</el-link
            >
            <span v-else>{{ scope.row.name }}</span>
          </template>
        </el-table-column>
        <el-table-column align="center" prop="time" label="更新时间" sortable>
          <template slot-scope="scope">
            <i class="el-icon-time" />
            <span>{{ scope.row.time }}</span>
          </template>
        </el-table-column>
        <el-table-column align="center" prop="size" label="文件大小" sortable :sort-method="sortMethod">
        </el-table-column>
        <el-table-column align="center" prop="id" label="操作" width="130">
          <template slot-scope="scope">
            <el-button type="primary" size="mini" @click="downFile(scope.row)">下载</el-button>
          </template>
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
export default {
  data() {
    return {
      rootdir: "/dataset/Pcap",
      target_proto: null,
      pcaps: [],
      checkedPcaps: [],
      currentPage: 1,
      pagesize: 10,
      pcap_checks: null,
      resDir: null,

      fileDown: {
      loadDialogStatus: false, //弹出框控制的状态
      percentage: 0, //进度条的百分比
      source: {}, //取消下载时的资源对象
      },
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
      this.$axios
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
          
          let fileName = null; //替换文件名
          if (row.name.split(".").length > 1)
            fileName = row.name;
          else
            fileName = row.name + ".zip";
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
    // 文件大小单位换算并排序
    sortMethod(a, b) {
        let aa = null;
        let bb = null;
        if (a.size.indexOf("MB") > 0) {
          aa = parseFloat(a.size.split("\t")) * 1024;
        } else if (a.size.indexOf("GB") > 0) {
          aa = parseFloat(a.size.split("\t")) * 1024 * 1024;
        } else {
          aa = parseFloat(a.size.split("\t"));
        }
        if (b.size.indexOf("MB") > 0) {
          bb = parseFloat(b.size.split("\t")) * 1024;
        } else if (b.size.indexOf("GB") > 0) {
          bb = parseFloat(b.size.split("\t")) * 1024 * 1024;
        } else {
          bb = parseFloat(b.size.split("\t"));
        }
        return aa - bb;
      }
  },
};
</script>