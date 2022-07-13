<template>
  <div v-loading="loading" element-loading-text="分析中...">
    <el-row :gutter="20">
      <el-col :span="9">
        <el-card class="box-card" style="height: 330px;" shadow="hover">
          <div slot="header" class="clearfix">
            <span style="font-weight: bold;">应用分类</span>
            <div style="float:right">
              <span style="font-size: 16px; color:rgb(75, 133, 144)">对比模式 </span>
              <el-switch
                v-model="compairMode"
                active-color="#13ce66"
                inactive-color="#ff4949">
              </el-switch>
            </div>
          </div>
          <el-form label-width="70px" class="demo-form-inline">
            <el-row>
              <el-form-item label="模型">
                <el-select placeholder="请选择" :disabled="compairMode" v-model="params.model">
                  <el-option
                    v-for="item in Infos.models"
                    :label="item.tag"
                    :value="item.tag"
                    :key="item.tag"
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
      <el-col :span="15">
      <el-carousel :autoplay="false" ref="carousel" height="350px">
        <el-carousel-item align="center" v-for="(item) in images" :key="item.url">
          <img :src="item.url" />
        </el-carousel-item>
      </el-carousel>
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
      <el-table-column prop="tag" label="模型标签" width="120" align="center">
      </el-table-column>
      <el-table-column prop="info" label="模型介绍" align="center">
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
export default {
  watch: {
    "params.model": function (val) {
      if (val === "LS-LSTM")
        this.setActiveItem(0);
      else if (val === "LS-CapsNet")
        this.setActiveItem(1);
      else if (val === "Fs-Net")
        this.setActiveItem(2);
    },
  },
  mounted() {
    window.Vue = this;
    this.loadDatasets();
  },
  data() {
    return {
      compairMode: false,
      images: [
        { url: require("@/assets/LS_CapsNet.png")},
        { url: require("@/assets/LS-LSTM.png")},
        { url: require("@/assets/Fs-Net.jpg")},
      ],

      loading: false,
      dialogVisible: false,
      resp: { data: [], time: "" },
      datasets: [],
      params: {
        model: "",
        datatype: "",
        dataset: "",
      },
      Infos: {
        models: [
          {
            name: "长度敏感的长短期记忆网络模", 
            tag: "LS-LSTM",
            info: "与LS-CapsNet相比，基于长度敏感长短期记忆网络模型（LS-LSTM），但需要N-gram层的输出对齐。因此，它更适合高速网络环境下的全流量低延迟快速分析和长期采集场景。",
          },
          {
            name: "长度敏感的胶囊神经网络模型",
            tag: "LS-CapsNet",
            info: "考虑了加密网络协议栈之间的差异，并在多协议环境中提出了一种基于复合深度学习的方法，该方法使用滑动多协议数据单元（multiPDU）长度序列作为特征，充分利用多协议数据单元长度中的马尔可夫属性排序并保持与 TLS-1.3 环境的适用性。",
          },
          {
            name: "流序列神经网络模型",
            tag: "Fs-Net",
            info: "将递归神经网络应用于加密流量分类问题，并提出了流序列网络（FS-Net）。FS-Net 是一种端到端的分类模型，可以从原始流中学习代表性特征，然后将它们分类在一个统一的框架中。此外，采用了多层编码器-解码器结构，可以深入挖掘流的潜在序列特征，并导入可以增强特征有效性的重构机制。",
          },
        ],
        datatypes: [{ name: "tcp" }, { name: "tls" }, { name: "http" }],
      },
    };
  },
  methods: {
    setActiveItem(index){
      this.$refs.carousel.setActiveItem(index);
    },
    async ClassifyBtn() {
      let _this = this;
      _this.loading = true;
      let resultInfo = {
        data: [],
        time: 0.0,
      };
      if(_this.compairMode){ // 对比模式
        for(let i = 0; i < _this.Infos.models.length; i++){
          await _this.$axios
          .get("http://211.65.197.130:8099/lstm", {
            params: {
              model: _this.Infos.models[i].tag,
              datatype: _this.params.datatype,
              dataset: _this.params.dataset,
            },
          })
          .then((resp) => {
            console.log(resp);
            if (resp.data.code === 200) {
              if(resultInfo.data.length === 0){
                // console.log(resp)
                for (let j = 0; j < resp.data.data.length; j++) {
                  resultInfo.data.push({});
                  if(_this.Infos.models[i].tag === 'LS-LSTM'){
                    resultInfo.data[j]['LS-LSTM'] = resp.data.data[j]
                  }
                  else if(_this.Infos.models[i].tag === 'LS-CapsNet'){
                    resultInfo.data[j]['LS-CapsNet'] = resp.data.data[j]
                  }
                  else if(_this.Infos.models[i].tag === 'Fs-Net'){
                    resultInfo.data[j]['Fs-Net'] = resp.data.data[j]
                  }
                }
              }
              else{
                console.log(resp)
                for (let j = 0; j < resp.data.data.length; j++) {
                  if(_this.Infos.models[i].tag === 'LS-LSTM'){
                    resultInfo.data[j]['LS-LSTM'] = resp.data.data[j]
                  }
                  else if(_this.Infos.models[i].tag === 'LS-CapsNet'){
                    resultInfo.data[j]['LS-CapsNet'] = resp.data.data[j]
                  }
                  else if(_this.Infos.models[i].tag === 'Fs-Net'){
                    resultInfo.data[j]['Fs-Net'] = resp.data.data[j]
                  }
                }
              }
              resultInfo.time = resultInfo.time + parseFloat(resp.data.time);
            } else {
              alert("分类失败");
              _this.loading = false;
            }
          })
          .catch(function (error) {
            alert("分类失败");
            _this.loading = false;
          });
        }
        this.$router.push({
          path: "show-compair-result",
          query: { response: resultInfo },
        });
      }
      else{ // 单例模式
        _this.$axios
        .get("http://211.65.197.130:8099/lstm", {
          params: {
            model: _this.params.model,
            datatype: _this.params.datatype,
            dataset: _this.params.dataset,
          },
        })
        .then((resp) => {
          console.log(resp);
          if (resp.data.code === 200) {
            _this.resp.data = resp.data.data;
            _this.resp.time = resp.data.time;
            _this.$router.push({
              path: "show-result",
              query: { response: _this.resp },
            });
          } else {
            alert("分类失败");
            _this.loading = false;
          }
        })
        .catch(function (error) {
          alert("分类失败");
          _this.loading = false;
        });
      }
      
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
  
  /* max-height: 100%;
  max-width: 100%; */
  height:100%;
  width:100%;
  /* vertical-align: middle;
  margin: 0; */
}

</style>