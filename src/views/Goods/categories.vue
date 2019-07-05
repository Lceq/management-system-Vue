<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>活动管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-button type="success" plain style="margin-top:10px" @click="openAddCateDialog">添加按钮</el-button>
    <!-- 表格 -->
    <el-table ref="singleTable" style="width: 100%" :data="catoriesList">
      <!-- tree-grid -->
      <!-- file-icon指的就是叶子节点的图标
            folder-icon指的就是可展开的项的图标
            prop 指的就是当前列要显示的数据的属性名称
            label 指的就是表头 
            treeKey 节点的唯一表示
            parentKey 数据的父节点id
            childKey 当前节点的所有子节点存放的属性的名字 默认是children

            indent-size是用来设置每个层级之间的缩进距离的
            设置indent-size 需要同时制定 层级属性

            level-key:    设置为数据中标识层级的属性名即可
      -->
      <el-table-tree-column
        label="分类名称"
        prop="cat_name"
        file-icon="el-icon-document-checked"
        folder-icon="el-icon-folder"
        width="220"
        tree-key="cat_id"
        parent-key="cat_pid"
        level-key="cat_level"
        :indent-size="50"
      ></el-table-tree-column>

      <el-table-column label="是否有效">
        <template v-slot="{row}">{{row.cat_deleted ? "是" : "否"}}</template>
      </el-table-column>
      <el-table-column property="cat_level" label="层级">
        <template v-slot="{row}">
          <div>{{row.cat_level | levelfilter }}</div>
        </template>
      </el-table-column>
      <el-table-column label="操作">
        <template v-slot="{row}">
          <el-button type="primary" plain size="mini" icon="el-icon-edit"></el-button>
          <el-button type="danger" icon="el-icon-delete" size="mini" plain></el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页器 -->
    <el-pagination
      background
      layout="prev, pager, next"
      :total="total"
      :page-size=" pagesize"
      :current-page="pagenum"
      @current-change="onPageChange"
    ></el-pagination>
    <!-- 添加模态框 -->
    <el-dialog title="收货地址" :visible.sync="isShowAddCateDialog">
      <el-form ref="addCategoryFrom" :model="addCategoryFrom" label-width="100px">
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="addCategoryFrom.cat_name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="父级分类" prop="parentArr">
          <el-cascader v-model="addCategoryFrom.parentArr" :options="options" :props="defaultProps"></el-cascader>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="isShowAddCateDialog = false">取 消</el-button>
        <el-button type="primary" @click="addCat">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>
import { async } from "q";
// 当前页面中要用到 一个第三方的表格插件（ElementUI的插件）
// element-tree-grid

// 安装  npm i element-tree-grid -S

// 使用：
// 如果需要全局使用，可能很多页面里面都会用到这个组件，那么就把下面这两句代码放到main.js中
// const ElTreeGrid = require('element-tree-grid');
// Vue.component(ElTreeGrid.name, ElTreeGrid)
// 页面中使用该组件，只需要用  el-table-tree-column 标签即可

// 如果只是需要某个页面中使用，只需要进行局部注册组件即可
// 在使用到这个组件的页面中，引入该组件，然后进行 components 局部注册即可
// 当前页码用到了第三方插件  element-tree-grid
var ElTreeGrid = require("element-tree-grid");
// Vue.component(ElTreeGrid.name,ElTreeGrid);
export default {
  components: {
    [ElTreeGrid.name]: ElTreeGrid
  },
  data() {
    return {
      catoriesList: [],
      pagenum: 1,
      pagesize: 5,
      total: 0,
      isShowAddCateDialog: false,
      addCategoryFrom: {
        cat_name: "",
        parentArr: []
      },
      options: [],
      defaultProps: {
        value: "cat_id",
        label: "cat_name",
        checkStrictly: true
      }
    };
  },
  filters: {
    levelfilter(value) {
      switch (value) {
        case 0:
          return "一级";
        case 1:
          return "二级";
        case 2:
          return "三级";
      }
    }
  },
  methods: {
    // render列表
    async getCatoriesList() {
      let res = await this.$http({
        url: "categories",
        params: {
          type: 3,
          pagenum: this.pagenum,
          pagesize: this.pagesize
        }
      });
      // console.log(res);
      this.catoriesList = res.data.data.result;
      this.total = res.data.data.total;
    },
    // 打开模态框
    async openAddCateDialog() {
      let res = await this.$http({
        url: "categories",
        params: {
          type: 2
        }
      });
      // console.log(res);
      this.isShowAddCateDialog = true;
      this.options = res.data.data;
    },
    // 添加分类
    async addCat() {
      //   // 分类父级id
      //   // this.addCateFormData.parentArr  这个里面放的是所有的父分类的id
      //   // 我们只需要最后一个就可以
      let cat_id = this.addCategoryFrom.parentArr.pop() || 0;
      // console.log(cat_id);

      // 分类名称
      let cat_name = this.addCategoryFrom.cat_name;
      // console.log(cat_name);

      // 分类层级
      let cat_level = this.addCategoryFrom.parentArr.length;
      // console.log(cat_level);

      let res = await this.$http({
        url: "categories",
        method: "post",
        data: {
          cat_id,
          cat_name,
          cat_level
        }
      });
      // console.log(res);
      if (res.data.meta.status === 201) {
        this.$message({
          message: res.data.meta.msg,
          duration: 1000,
          type: "success"
        });
        this.isShowAddCateDialog = false;
        // 重置表单
        this.$refs.addCategoryFrom.resetFields();
        this.getCatoriesList();
      }
    },
    // 分页
    onPageChange(currentpage) {
      this.pagenum = currentpage;

      this.getCatoriesList();
    }
  },
  created() {
    this.getCatoriesList();
  }
};
</script>
