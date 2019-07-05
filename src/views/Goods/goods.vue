<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>活动管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-button type="success" plain style="margin-top:10px" @click="$router.push('/goods-add')">添加按钮</el-button>
    <!-- 表格 -->
    <el-table ref="singleTable" :data="goodsFrom" style="width: 100%">
      <el-table-column type="index" width="50"></el-table-column>
      <el-table-column property="goods_name" label="商品名称"></el-table-column>
      <el-table-column property="goods_price" label="商品价格"></el-table-column>
      <el-table-column property="goods_weight" label="商品重量"></el-table-column>
      <!--  -->
      <el-table-column property="add_time" label="商品时间">
        <template v-slot="{row}">{{ row.add_time | timeFilter }}</template>
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
      :page-size="pagenum"
      :current-page="pagesize"
      @current-change="onPageChange"
    ></el-pagination>
  </div>
</template>
<script>
import moment from "moment";
export default {
  filters: {
    timeFilter(value) {
      return moment.unix(value).format("YYYY-MM-DD HH:mm:ss");
    }
  },
  data() {
    return {
      goodsFrom: [],
      pagenum: 1,
      pagesize: 5,
      total: 0
    };
  },
  methods: {
    async getGoodsFrom() {
      let res = await this.$http({
        url: `goods`,
        params: {
          pagenum: this.pagenum,
          pagesize: this.pagesize
        }
      });
      this.goodsFrom = res.data.data.goods;
      this.total = res.data.data.total;
      // console.log(res);
    },
    onPageChange(currentPage) {
      this.pagenum = currentPage;
      this.getGoodsFrom();
    }
  },
  created() {
    this.getGoodsFrom();
  }
};
</script>
