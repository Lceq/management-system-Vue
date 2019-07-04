<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>活动管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-table ref="singleTable" :data="rightsFrom" style="width: 100%">
      <el-table-column type="index" width="50"></el-table-column>
      <el-table-column property="authName" label="权限名称" width="120"></el-table-column>
      <el-table-column property="path" label="路径" width="120"></el-table-column>
      <el-table-column property="level" label="层级">
        <template v-slot="{row}">
          <div>{{row.level | levelFilter}}</div>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>
<script>
export default {
  data() {
    return {
      rightsFrom: []
    };
  },
  filters: {
    levelFilter(value) {
      switch (value) {
        case "0":
          return "一级";
        case "1":
          return "二级";
        case "2":
          return "三级";
      }
    }
  },
  methods: {},
  async created() {
    let res = await this.$http({
      url: "rights/list"
    });

    // console.log(res);
    this.rightsFrom = res.data.data;
  }
};
</script>