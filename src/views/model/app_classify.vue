<template>
  <div v-loading="loading" element-loading-text="分析中...">
    <el-row :gutter="20">
      <el-col :span="12">
        <el-card class="box-card" style="height:330px" shadow="hover">
          <div slot="header" class="clearfix">
            <span>应用分类</span>
          </div>
          
          <el-form label-width="70px" class="demo-form-inline">
            <el-row>
              <el-form-item label="模型">
                <el-select placeholder="请选择" v-model="params.model">
                  <el-option
                    v-for="item in Infos.models"
                    :label="item.name"
                    :value="item.name"
                    :key="item.name"
                  ></el-option>
                </el-select>
              </el-form-item>
            </el-row>
            <el-row>
              <el-form-item label="数据类型">
                <el-select placeholder="请选择" v-model="params.datatype">
                  <el-option
                    v-for="item in Infos.datatypes"
                    :label="item.name"
                    :value="item.name"
                    :key="item.name"
                  ></el-option>
                </el-select>
              </el-form-item>
            </el-row>
            <el-form-item label="数据集">
              <el-select
                filterable
                placeholder="请选择"
                v-model="params.dataset"
              >
                <el-option
                  v-for="item in datasets"
                  :label="item.name"
                  :value="item.name"
                  :key="item.name"
                ></el-option>
              </el-select>
            </el-form-item>
            <el-form-item>
              <el-button type="primary" @click="ClassifyBtn"
                >开始分类</el-button
              >
            </el-form-item>
          </el-form>
        </el-card>
      </el-col>
      <el-col :span="12" >
        <el-card style="height:330px" shadow="hover"></el-card>
      </el-col>
    </el-row>

    <el-table
      :data="Infos.models"
      stripe
      border
      style="width: 100%"
      max_height="450"
      :default-sort="{ prop: 'id', order: 'descending' }"
    >
      <el-table-column prop="name" label="模型名称" width="180" align="center">
      </el-table-column>
      <el-table-column prop="info" label="模型介绍" align="center">
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
export default {
  mounted() {
    window.Vue = this;
    this.loadDatasets();
  },
  data() {
    return {
      loading: false,
      dialogVisible: false,
      resp: { data: [], time: "" },
      datasets: [],
      formInline: {
        user: "",
        model: "",
      },
      params: {
        model: "",
        datatype: "",
        dataset: "",
      },
      Infos: {
        models: [
          { name: "LS-LSTM", info: "这里是LSTM的介绍" },
          { name: "LS-CapsNet", info: "这里是CapsNet的介绍" },
          { name: "FS-Net", info: "这里是fsNet的介绍" },
        ],
        datatypes: [{ name: "tcp" }, { name: "tls" }, { name: "http" }],
      },
    };
  },
  methods: {
    ClassifyBtn() {
      this.loading = true;
      this.$axios
        .get("http://211.65.197.130:8099/lstm", {
          params: {
            model: this.params.model,
            datatype: this.params.datatype,
            dataset: this.params.dataset,
          },
        })
        .then((resp) => {
          console.log(resp);
          if (resp.data.code === 200) {
            this.resp.data = resp.data.data;
            this.resp.time = resp.data.time;

            this.$router.push({
              path: "show-result",
              query: { response: this.resp },
            });
          } else {
            alert("分类失败");
            this.loading = false;
          }
        })
        .catch(function (error) {
          alert("分类失败");
          this.loading = false;
        });
    },
    // 获取并展示数据集
    loadDatasets() {
      this.$axios
        .get("http://211.65.197.130:8099/findAllFile", {
          params: {
            root: "/dataset",
          },
        })
        .then((resp) => {
          console.log(resp);
          if (resp.data.code === 200) {
            this.datasets = resp.data.data;
            console.log(this.datasets);
          }
        })
        .catch(function (error) {
          alert("加载失败");
          console.log(error);
          window.location.reload();
        });
    },
  },
};
</script>

<style>
.box-card {
  align-self: center;
  align-content: center;
  /* width: 1000px; */
}
</style>