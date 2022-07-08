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
        <el-card style="height:330px" shadow="hover" :body-style="{ padding: '0px' }">
          <!-- <el-image v-if="select_model === 'LS-LSTM'" :fit='fill' :src="require('@/assets/LS-LSTM.png')"></el-image> -->
          <img v-if="select_model === 'LS-LSTM'" :src="require('@/assets/LS-LSTM.png')" />
          <img v-else-if="select_model === 'LS-CapsNet'" :src="require('@/assets/LS_CapsNet.png')" />
          <img v-else-if="select_model === 'Fs-Net'" :src="require('@/assets/Fs-Net.jpg')" />
      </el-card>
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
  props:{
    'select_model':String,
  },
  watch:{
    'params.model':function(val){
      this.select_model = val;
    }
  },
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
          { name: "LS-LSTM", info: "将递归神经网络应用于加密流量分类问题，并提出了流序列网络（FS-Net）。FS-Net 是一种端到端的分类模型，可以从原始流中学习代表性特征，然后将它们分类在一个统一的框架中。此外，采用了多层编码器-解码器结构，可以深入挖掘流的潜在序列特征，并导入可以增强特征有效性的重构机制。" },
          { name: "LS-CapsNet", info: "考虑了加密网络协议栈之间的差异，并在多协议环境中提出了一种基于复合深度学习的方法，该方法使用滑动多协议数据单元（multiPDU）长度序列作为特征，充分利用多协议数据单元长度中的马尔可夫属性排序并保持与 TLS-1.3 环境的适用性。" },
          { name: "Fs-Net", info: "与LS-CapsNet相比，基于长度敏感长短期记忆网络模型（LS-LSTM），但需要N-gram层的输出对齐。因此，它更适合高速网络环境下的全流量低延迟快速分析和长期采集场景。" },
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
            root: "/dataset/Length-Sequnce",
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
img {
  max-width: 100%;
  /* max-height: 100%; */
  vertical-align: middle;
}
</style>