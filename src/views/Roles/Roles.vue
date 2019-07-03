<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>活动管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 表格 -->
    <el-table ref="singleTable" :data="rolesForm" style="width: 100%">
      <!-- 展开 -->
      <el-table-column type="expand">
        <template v-slot="{row}">
          <el-row type="flex" class="level1" v-for="level1 in row.children" :key="level1.id">
            <el-col :span="6">
              <el-tag @close="delRights(row,level1.id)" closable>{{level1.authName}}</el-tag>
              <i class="el-icon-arrow-right"></i>
            </el-col>
            <el-col>
              <el-row type="flex" class="level2" v-for="level2 in level1.children" :key="level2.id">
                <el-col :span="6">
                  <el-tag
                    @close="delRights(row,level2.id)"
                    type="success"
                    closable
                  >{{level2.authName}}</el-tag>
                  <i class="el-icon-arrow-right"></i>
                </el-col>
                <el-col>
                  <el-tag
                    class="level3"
                    v-for="level3 in level2.children"
                    :key="level3.id"
                    type="warning"
                    closable
                    @close="delRights(row,level3.id)"
                  >{{level3.authName}}</el-tag>
                </el-col>
              </el-row>
            </el-col>
          </el-row>
        </template>
      </el-table-column>
      <el-table-column type="index" width="50"></el-table-column>
      <el-table-column property="roleName" label="角色名称" width="120"></el-table-column>
      <el-table-column property="roleDesc" label="描述" width="120"></el-table-column>
      <el-table-column label="操作">
        <template v-slot="{row}">
          <el-button type="primary" plain size="mini" icon="el-icon-edit"></el-button>
          <el-button type="danger" icon="el-icon-delete" size="mini" plain></el-button>
          <el-button
            type="success"
            icon="el-icon-check"
            size="mini"
            plain
            @click="openrightsModal(row)"
          >分配权限</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分配权限模态框 -->
    <el-dialog title="提示" :visible.sync="rightsModalShow" width="30%">
      <!-- data 是用来绑定数据的
            show-checkbox是用来设置是否要展示 checkbox
            node-key 指的是当前节点的唯一表示
            default-expanded-keys 这是一个数组，表示默认让哪些节点展开
            default-checked-keys 这是一个数据，表示默认选中哪些节点
      props:  children是用来指定子级树的数据属性名，label以及节点要展示到文字的属性名-->
      <!-- default-expand-all  是否默认展开所有节点-->
      <el-tree
        :data="rightList"
        show-checkbox
        node-key="id"
        :default-checked-keys="checkedRights"
        :default-expand-all="true"
        :props="defaultProps"
        ref="rightsTree"
      ></el-tree>

      <span slot="footer" class="dialog-footer">
        <el-button @click="rolesModalShow = false">取 消</el-button>
        <el-button type="primary" @click="updateRoleRights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      rolesForm: [],
      // 模态框
      rightsModalShow: false,
      rightList: [],
      // 这个是用来设置 tree组件的 数据展示 子节点以及 展示的文字的
      defaultProps: {
        children: "children",
        //    // 当前节点展示的文字的属性名
        label: "authName"
      },
      checkedRights: [],
      // 用来存储要编辑的role的id
      currentEditRoleId: -1
    };
  },
  methods: {
    // render
    async getRolesList() {
      let res = await this.$http({
        url: "roles"
      });
      // console.log(res);
      this.rolesForm = res.data.data;
    },
    // 点击分配权限模态框显示
    async openrightsModal(row) {
      // 0. 把 当前正在编辑的id改成当前role的id
      this.currentEditRoleId = row.id;
      let res = await this.$http({
        url: "rights/tree"
      });
      // console.log(res);
      this.rightList = res.data.data;
      this.rightsModalShow = true;
      let level1Ids = [];
      let level2Ids = [];
      let level3Ids = [];
      // 获取二级权限的id，组合成数组
      row.children.forEach(level1 => {
        level1Ids.push(level1.id);
        level1.children.forEach(level2 => {
          level2Ids.push(level2.id);
          level2.children.forEach(level3 => {
            level3Ids.push(level3.id);
          });
        });
      });
      this.checkedRights = level3Ids;
    },
    // 确认按钮
    async updateRoleRights() {
      // 1.得到拼接的id
      let arr2 = this.$refs.rightsTree.getCheckedKeys();
      let arr1 = this.$refs.rightsTree.getHalfCheckedKeys();
      // console.log(arr2, arr1);
      let ids = [...arr2, ...arr1].join(",");
      // console.log(ids);
      // 角色授权
      // 2.发送axios
      let res = await this.$http({
        url: `roles/${this.currentEditRoleId}/rights`,
        method: "post",
        data: {
          rids: ids
        }
      });
      // console.log(res);/
      // // 3. 提示用户更新成功
      this.$message({
        type: "success",
        message: res.data.meta.msg,
        duration: 1000
      });
      // 4. 更新成功之后，重新获取列表数据
      this.getRolesList();
      // 模态框隐藏
      this.rightsModalShow = false;
    },
    // 删除功能
    // 一级删除，二级和三级
    // bug 点击删除后，会合起来
    async delRights(row, id) {
      console.log(id);
      //  拼接id
      let level1Ids = [];
      let level2Ids = [];
      let level3Ids = [];
      // 获取二级权限的id，组合成数组
      row.children.forEach(level1 => {
        level1Ids.push(level1.id);
        level1.children.forEach(level2 => {
          level2Ids.push(level2.id);
          level2.children.forEach(level3 => {
            level3Ids.push(level3.id);
          });
        });
      });
      let result = [...level1Ids, ...level2Ids, ...level3Ids];
      // 把id相同的删除，并拼接成字符串
      let ids = result.filter(v => v !== id).join();
      // 角色授权
      // 2.发送axios
      let res = await this.$http({
        url: `roles/${row.id}/rights`,
        method: "post",
        data: {
          rids: ids
        }
      });
      this.$message({
        type: "success",
        message: res.data.meta.msg,
        duration: 1000
      });
      this.getRolesList();
    }
  },
  created() {
    this.getRolesList();
  }
};
</script>
<style lang="less" scoped>
.level1 {
  border-bottom: 1px dashed #ccc;
  padding: 10px 0;
}

.level1:last-child {
  border-bottom: none;
}

.level2 {
  margin-bottom: 15px;
}

.level3 {
  margin-right: 10px;
  margin-bottom: 10px;
}
</style>
