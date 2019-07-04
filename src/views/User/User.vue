<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>活动管理</el-breadcrumb-item>
      <el-breadcrumb-item>活动列表</el-breadcrumb-item>
      <el-breadcrumb-item>活动详情</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 输入框 -->
    <el-row :gutter="20">
      <el-col :span="6">
        <el-input
          placeholder="请输入内容"
          class="input-with-select"
          v-model="keyword"
          @keyup.enter.native="search"
          @blur="blur"
        >
          <el-button slot="append" icon="el-icon-search" @click="search"></el-button>
        </el-input>
      </el-col>
      <el-col :span="6">
        <el-button type="success" @click="openAddUserModal">添加用户</el-button>
      </el-col>
    </el-row>

    <!-- data  用来绑定这个表格要展示的数据对象 -->
    <!-- stripe 设置隔行变色 -->
    <el-table :data="userList" stripe style="width: 100%">
      <!-- prop 设置当前列中要展示的数据的属性名 -->
      <!-- label 当前列的表头文字 -->
      <el-table-column prop="username" label="姓名" width="180"></el-table-column>
      <el-table-column prop="email" label="邮箱" width="180"></el-table-column>
      <el-table-column prop="mobile" label="电话"></el-table-column>

      <!-- 如果当前列中不会把数据直接展示出来，那么就没有必要为当前列设置prop属性 -->
      <el-table-column label="用户状态">
        <!-- 在这里，无法直接获取到每一行的数据，这个数据在el-table表格组件中 -->
        <!-- 如果要获取每一行的数据，那么我们就需要通过作用于插槽的方式，把数据接收到 -->
        <template v-slot="{row}">
          <el-switch
            v-model="row.mg_state"
            active-color="#13ce66"
            @change="toggleState(row)"
            inactive-color="#ff4949"
          ></el-switch>
        </template>
      </el-table-column>
      <el-table-column label="操作">
        <template v-slot="{row}">
          <el-button
            type="primary"
            plain
            size="mini"
            icon="el-icon-edit"
            @click="openEditUserModal(row.id)"
          ></el-button>
          <el-button type="danger" icon="el-icon-delete" @click="delUser(row.id)" size="mini" plain></el-button>
          <el-button
            type="success"
            icon="el-icon-check"
            size="mini"
            @click="openAssignRolseModal(row)"
            plain
          >分配角色</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页器 -->
    <el-pagination
      background
      layout="prev, pager, next"
      :total="total"
      :page-size="pageSize"
      :current-page="currentPage"
      @current-change="onPageChange"
    ></el-pagination>
    <!-- 添加模态框 -->
    <el-dialog title="添加用户" :visible.sync="addModalShow">
      <el-form :model="form" :rules="rules" ref="form">
        <el-form-item label="用户名" label-position="left" label-width="100px" prop="username">
          <el-input v-model="form.username"></el-input>
        </el-form-item>
        <el-form-item label="密码" label-position="left" label-width="100px" prop="password">
          <el-input v-model="form.password"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" label-position="left" label-width="100px" prop="email">
          <el-input v-model="form.email"></el-input>
        </el-form-item>
        <el-form-item label="电话" label-position="left" label-width="100px" prop="mobile">
          <el-input v-model="form.mobile"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="addModalShow = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </div>
    </el-dialog>
    <!-- 修改模态框 -->
    <el-dialog title="修改用户" :visible.sync="editModalShow">
      <el-form :model="editform" :rules="editrules" ref="editform">
        <el-form-item label="用户名" label-position="left" label-width="100px" prop="username">
          <el-tag type="info" v-text="editform.username"></el-tag>
        </el-form-item>
        <el-form-item label="邮箱" label-position="left" label-width="100px" prop="email">
          <el-input v-model="editform.email"></el-input>
        </el-form-item>
        <el-form-item label="电话" label-position="left" label-width="100px" prop="mobile">
          <el-input v-model="editform.mobile"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="editModalShow = false">取 消</el-button>
        <el-button type="primary" @click="editUser">确 定</el-button>
      </div>
    </el-dialog>
    <!-- 分配角色模态框 -->
    <el-dialog title="分配角色" :visible.sync="AssignRolseModalShow">
      <el-form :model="assignRolseFrom">
        <el-form-item label="用户名" label-position="left" label-width="100px">
          <el-tag type="info" v-text="assignRolseFrom.username"></el-tag>
        </el-form-item>
        <el-form-item label="活动区域" label-width="100px">
          <el-select placeholder="请选择" v-model="assignRolseFrom.rid">
            <el-option
              v-for="item in roleList"
              :key="item.id"
              :label="item.roleName"
              :value="item.id"
            ></el-option>
            <!-- <el-option value="1"></el-option> -->
          </el-select>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="AssignRolseModalShow = false">取 消</el-button>
        <el-button type="primary" @click="updataRolse">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>
// mmyyy
export default {
  data() {
    return {
      userList: [],
      total: 0,
      // 每页显示条数
      pageSize: 3,
      // 当前页码
      currentPage: 1,
      keyword: "",
      AssignRolseModalShow: false,
      addModalShow: false,
      editModalShow: false,
      // 添加数据
      form: {
        username: "",
        password: "",
        email: "",
        mobile: ""
      },
      // 添加效验
      rules: {
        username: [
          { required: true, message: "请输入活动名称", trigger: "blur" },
          { min: 5, max: 10, message: "长度在 5 到 10 个字符", trigger: "blur" }
        ],
        password: [
          { required: true, message: "请输入活动名称", trigger: "blur" },
          { min: 6, max: 12, message: "长度在 6 到 12 个字符", trigger: "blur" }
        ],
        email: [
          {
            pattern: /\w+([-+.]\w+)*@\w+([-.]\w+)*\.\w+([-.]\w+)*/,
            message: "邮箱格式不正确",
            trigger: "change"
          }
        ],
        mobile: [
          {
            pattern: /^(0|86|17951)?(13[0-9]|15[012356789]|166|17[3678]|18[0-9]|14[57])[0-9]{8}$/,
            message: "手机号码格式不正确",
            trigger: "change"
          }
        ]
      },
      // 修改数据
      editform: {
        id: 0,
        username: "",
        mobile: "",
        email: ""
      },
      // 修改效验
      editrules: {
        username: [
          { required: true, message: "请输入活动名称", trigger: "blur" },
          { min: 5, max: 10, message: "长度在 5 到 10 个字符", trigger: "blur" }
        ],
        password: [
          { required: true, message: "请输入活动名称", trigger: "blur" },
          { min: 6, max: 12, message: "长度在 6 到 12 个字符", trigger: "blur" }
        ],
        email: [
          {
            pattern: /\w+([-+.]\w+)*@\w+([-.]\w+)*\.\w+([-.]\w+)*/,
            message: "邮箱格式不正确",
            trigger: "change"
          }
        ],
        mobile: [
          {
            pattern: /^(0|86|17951)?(13[0-9]|15[012356789]|166|17[3678]|18[0-9]|14[57])[0-9]{8}$/,
            message: "手机号码格式不正确",
            trigger: "change"
          }
        ]
      },
      // 分配角色的数据
      assignRolseFrom: {
        username: "",
        rid: ""
      },
      roleList: []
    };
  },
  methods: {
    // 分配角色模态框的确认按钮
    async updataRolse() {
      // this.AssignRolseModalShow = false;
      let res = await this.$http({
        url: `users/${this.assignRolseFrom.id}/role`,
        method: "put",
        data: {
          rid: this.assignRolseFrom.rid
        }
      });
      //   console.log(res);
      this.$message({
        type: "success",
        message: res.data.meta.msg,
        duration: 1000
      });
      this.isAssainRoleDialogShow = false;
    },

    // 分配角色
    // 把用户名显示在模态框上
    async openAssignRolseModal(row) {
      let res = await this.$http({
        url: `users/${row.id}`
      });
      // console.log(res);
      // 下拉框的数据
      let roleResult = await this.$http({
        url: "roles"
      });
      // console.log(roleResult.data.data);
      this.roleList = roleResult.data.data;
      this.assignRolseFrom = res.data.data;
      this.AssignRolseModalShow = true;
    },
    // render
    getUserList() {
      this.$http({
        url: "users",
        params: {
          query: this.keyword,
          pagesize: this.pageSize,
          pagenum: this.currentPage
        }
        // headers: {
        //   Authorization: localStorage.getItem("token")
        // }
      }).then(res => {
        // console.log(res);
        // this.userList = res.data.data.users;
        let {
          data: { data, meta }
        } = res;
        this.userList = data.users;
        // console.log(data.users, 11);
        this.total = data.total;
      });
    },
    // 分页器的分页
    onPageChange(page) {
      // console.log("页码发生改变了", page);
      this.currentPage = page;
      // 重新去后端请求数据
      this.getUserList();
    },
    // 搜索功能
    search() {
      this.getUserList();
      // console.log(this.keyword);
    },
    // 失去焦点显示所有数据
    blur() {
      this.getUserList();
    },
    // 删除功能
    async delUser(id) {
      try {
        await this.$confirm("此操作将永久删除该文件, 是否继续?", "提示", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        });
        this.$message({
          type: "success",
          message: "删除成功!"
        });
        let res = await this.$http({
          url: `users/${id}`,
          method: "delete"
          // headers: {
          //   Authorization: localStorage.getItem("token")
          // }
        });
        // console.log(res);
        if (res.data.meta.status === 200) {
          this.$message({
            type: "success",
            message: res.data.meta.msg,
            duration: 1000
          });
          this.currentpage = 1;
          this.getUserList();
        }
      } catch (err) {
        this.$message({
          type: "info",
          message: "已取消删除"
        });
      }
    },
    // 状态的修改
    async toggleState(user) {
      // console.log(user);
      let res = await this.$http({
        url: ` users/${user.id}/state/${user.mg_state}`,
        method: "put"
        // headers: {
        //   Authorization: localStorage.getItem("token")
        // }
      });
      // console.log(res);
      if (res.data.meta.status === 200) {
        this.$message({
          type: "success",
          message: res.data.meta.msg,
          duration: 1000
        });
      } else {
        this.$message({
          type: "err",
          message: res.data.meta.msg,
          duration: 1000
        });
        user.mg_state = !user.mg_state;
      }
      // let res= await
    },
    // 添加模态框是显示
    openAddUserModal() {
      this.addModalShow = true;
    },
    // 添加数据
    async addUser() {
      try {
        // 验证表单效验是否正确（true 正确）
        await this.$refs.form.validate();
        // console.log(valie);
        let res = await this.$http({
          url: `users`,
          method: "post",
          data: this.form
          // headers: {
          //   Authorization: localStorage.getItem("token")
          // }
        });
        // console.log(res);
        if ((res.data.meta.status = 201)) {
          this.$message({
            message: "恭喜你，这是一条成功消息",
            type: "success",
            duration: 1000
          });
          this.getUserList();
          this.addModalShow = false;
          this.$refs.form.resetFields();
        }
      } catch (err) {
        this.$message({
          message: "抱歉，这是一条失败消息",
          type: "error",
          duration: 1000
        });
      }
    },
    // 修改模态框的显示与隐藏
    async openEditUserModal(id) {
      // console.log(id);
      let res = await this.$http({
        url: `users/${id}`,
        method: "get"
        // headers: {
        //   Authorization: localStorage.getItem("token")
        // }
      });
      // console.log(res);
      this.editform = res.data.data;
      this.editModalShow = true;
    },
    // 修改功能
    async editUser() {
      // console.log(3333);

      try {
        // 验证表单效验是否正确（true 正确）
        await this.$refs.editform.validate();
        let res = await this.$http({
          url: `users/${this.editform.id}`,
          method: "put",
          data: {
            email: this.editform.email,
            mobile: this.editform.mobile
          }
          // headers: {
          //   Authorization: localStorage.getItem("token")
          // }
        });
        if (res.data.meta.status === 200) {
          this.$message({
            message: "成功",
            type: "success",
            duration: 1000
          });
          this.getUserList();
          this.editModalShow = false;
        } else {
          this.$message({
            message: "失败",
            tyep: "error",
            duration: 1000
          });
        }
        // console.log(res);
      } catch (err) {
        // console.log(err);
      }
    }
  },
  created() {
    this.getUserList();
  }
};
</script>
<style lang="less">
.el-main .el-breadcrumb {
  background-color: #d4dae0;
  font-size: 20px;
  height: 50px;
  line-height: 50px;
  padding-left: 10px;
}
</style>
