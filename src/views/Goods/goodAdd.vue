<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>活动管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 进度条 -->
    <el-steps :active="activeStep" style="margin-top:20px">
      <el-step title="第一步" description="基本信息"></el-step>
      <el-step title="第二步" description="商品图片"></el-step>
      <el-step title="第三步" description="商品内容"></el-step>
    </el-steps>
    <el-tabs
      v-model="activeName"
      @tab-click="changeTabs"
      tab-position="left"
      style=" margin-top:20px"
    >
      <el-tab-pane label="基本信息" name="base">
        <el-form ref="goodAddForm" :model="goodAddForm" label-width="80px">
          <el-form-item label="商品名称">
            <el-input v-model="goodAddForm.goods_name"></el-input>
          </el-form-item>
          <el-form-item label="商品价格">
            <el-input v-model="goodAddForm.goods_price"></el-input>
          </el-form-item>
          <el-form-item label="商品重量">
            <el-input v-model="goodAddForm.goods_weight"></el-input>
          </el-form-item>
          <el-form-item label="商品数量">
            <el-input v-model="goodAddForm.goods_number"></el-input>
          </el-form-item>
          <el-form-item label="商品分类">
            <el-cascader v-model="goodAddForm.goods_cat" :options="options" :props="defaultProp"></el-cascader>
          </el-form-item>
          <el-form-item label="是否促销">
            <el-radio v-model="goodAddForm.is_promote" label="1">是</el-radio>
            <el-radio v-model="goodAddForm.is_promote" label="0">否</el-radio>
          </el-form-item>
        </el-form>
        <!-- 下一步按钮 -->
        <el-button type="primary" @click="next(1,'pic')">下一步</el-button>
      </el-tab-pane>
      <el-tab-pane label="商品图片" name="pic">
        <!-- 图片上传 -->
        <el-upload
          class="upload-demo"
          action="http://localhost:8888/api/private/v1/upload"
          list-type="picture-card"
          :headers="headers"
          :on-success="picUpdateSuccess"
        >
          <el-button size="small" type="primary">点击上传</el-button>
          <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
        </el-upload>

        <el-button type="primary" @click="next(2,'content')">下一步</el-button>
      </el-tab-pane>
      <el-tab-pane label="商品内容" name="content">
        <quill-editor v-model="goodAddForm.goods_introduce"></quill-editor>
        <el-button type="primary" @click="addGoods">确认</el-button>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>
<script>
import "quill/dist/quill.core.css";
import "quill/dist/quill.snow.css";
import "quill/dist/quill.bubble.css";
import { quillEditor } from "vue-quill-editor";

export default {
  components: {
    quillEditor
  },
  data() {
    return {
      activeStep: 0,
      activeName: "base",
      goodAddForm: {
        goods_name: "",
        goods_price: 0,
        goods_weight: 0,
        goods_number: 0,
        goods_cat: [],
        is_promote: "0",
        goods_introduce: "",
        pics: []
      },
      options: [],
      shopCategoryList: {},
      // 联动内容
      defaultProp: {
        value: "cat_id",
        label: "cat_name"
      },
      headers: {
        Authorization: localStorage.getItem("token")
      }
    };
  },
  async created() {
    // 商品分类管理
    // 商品数据列表

    let res = await this.$http({
      url: "categories",
      type: 3
    });
    // console.log(res);
    this.options = res.data.data;
  },
  methods: {
    picUpdateSuccess(res) {
      // console.log(res);
      // this.goodAddForm.pics = res.data.url;
      this.goodAddForm.pics.push({
        pic: res.data.tmp_path
      });
      // console.log(res.data.url);
    },
    changeTabs(tab) {
      this.activeStep = +tab.index;
    },
    next(index, activeName) {
      this.activeStep = index;
      this.activeName = activeName;
    },
    async addGoods() {
      // console.log(catArr);
      let newgoodAddForm = {
        goods_name: this.goodAddForm.goods_name,
        goods_cat: this.goodAddForm.goods_cat.join(),
        goods_price: this.goodAddForm.goods_price,
        goods_number: this.goodAddForm.goods_number,
        goods_weight: this.goodAddForm.goods_weight,
        goods_introduce: this.goodAddForm.goods_introduce,
        is_promote: this.goodAddForm.is_promote,
        pics: this.goodAddForm.pics
      };
      // console.log(newgoodAddForm);
      // this.goodAddForm.goods_cat = this.goodAddForm.goods_cat.join();

      let res = await this.$http({
        url: "goods",
        method: "post",
        data: newgoodAddForm
      });
      // console.log(res);
      if (res.data.meta.status == 201) {
        this.$router.push("/goods");
      }
    }
  }
};
</script>
<style >
/* ql-editor  */
/* .ql-editor.ql-blank{
  height: 200px;
  background-color: pink;
} */
/* .el-tab-pane.quill-editor {
  height: 200px;
} */
/* .quill-editor {
}
.ql-container .ql-snow {
  height: 300px;
}
.quill-editor {
  height: 200px;
} */
/* ql-snow */
.ql-container {
  height: 200px;
}
</style>
