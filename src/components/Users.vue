<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 内容区卡片 -->
    <el-card class="box-card">
      <!-- 搜索 -->
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input
            placeholder="请输入搜索内容"
            v-model="queryParams.query"
            :clearable="true"
            @clear="getUserInfos"
            @keyup.enter.native="getUserInfos"
          >
            <el-button slot="append" icon="el-icon-search" @click="getUserInfos"></el-button>
          </el-input>
        </el-col>
        <el-col :span="6">
          <el-button type="primary" @click="addUserDialog=true">添加用户</el-button>
        </el-col>
      </el-row>
      <!-- 表格 -->
      <el-table :data="userInfos" border style="width: 100%">
        <el-table-column type="index" label="序号" width="60"></el-table-column>
        <el-table-column prop="username" label="用户名"></el-table-column>
        <el-table-column prop="mobile" label="手机号码" width="140"></el-table-column>
        <el-table-column prop="role_name" label="角色" width="130"></el-table-column>
        <el-table-column prop="email" label="邮箱" width="150"></el-table-column>
        <el-table-column label="状态" width="70">
          <el-switch v-model="info.row.mg_state" slot-scope="info"></el-switch>
          <!--在此获得每个用户的信息，用户信息已经被插槽传递到此位置了，请自动接收即可
              每个用户数据具体是通过"row"关键字定义的
          -->
          <!-- <span slot-scope="info">{{info.row}}</span> -->
        </el-table-column>
        <el-table-column label="操作" width="230">
          <template slot-scope="info">
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="showEditDialog(info.row.id)"
            ></el-button>
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="delUser(info.row.id)"
            ></el-button>
            <el-tooltip content="分配角色" placement="top" :enterable="false">
              <el-button type="warning" icon="el-icon-setting" size="mini"></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>

      <!-- //分页条 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryParams.pagenum"
        :page-sizes="[3, 5, 10, 20]"
        :page-size="queryParams.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="queryParams.total"
      ></el-pagination>
      <!-- 添加用户的dialog弹窗 -->
      <el-dialog title="添加用户" :visible.sync="addUserDialog" width="50%" @close="addUserDialogClose">
        <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="80px">
          <el-form-item label="用户名" prop="username">
            <el-input v-model="addForm.username"></el-input>
          </el-form-item>
          <el-form-item label="密码"  prop="password">
            <el-input v-model="addForm.password" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="addForm.email" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="手机号" prop="mobile">
            <el-input v-model="addForm.mobile" autocomplete="off"></el-input>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="addUserDialog = false">取 消</el-button>
          <el-button type="primary" @click="addUser">确 定</el-button>
        </div>
      </el-dialog>
      <!-- 修改用户的dialog弹窗-->
      <el-dialog title="修改用户" :visible.sync="editDialog" width="50%">
        <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="80px">
          <el-form-item label="用户名"  prop="username">
            <el-input v-model="editForm.username" :disabled="true"></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="editForm.email" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="手机号"  prop="mobile">
            <el-input v-model="editForm.mobile" autocomplete="off"></el-input>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="editDialog = false">取 消</el-button>
          <el-button type="primary" @click="editUser">确 定</el-button>
        </div>
      </el-dialog>
    </el-card>
  </div>
</template>
<script>
export default {
  // 生命周期函数
  created() {
    this.getUserInfos()
  },
  methods: {
    // 修改用户状态
    async changeState(id, state) {
      const { data: res } = await this.$http.put(`users/${id}/state/${state}`)
      if (res.meta.status !== 200) {
        return this.$message.error('修改用户状态失败')
      }
      this.$message({
        message: '修改用户状态成功',
        type: 'success',
        duration: 1500
      })
    },
    // 获取分页数据
    // 每页信息变化回调函数处理
    handleSizeChange(val) {
      // val： 代表变化后的每页信息显示条数
      this.queryParams.pagesize = val
      this.getUserInfos()
    },
    // 当前页码变化回调处理
    handleCurrentChange(val) {
      // val代表变化后的 页码
      this.queryParams.pagenum = val
      this.getUserInfos()
    },
    // 获取首屏用户数据
    async getUserInfos() {
      const { data: res } = await this.$http.get('users', {
        params: this.queryParams
      })
      console.log(res)
      // 获取数据失败
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      // 记录总条数变量
      this.queryParams.total = res.data.total
      // 将获取到的数据赋值给userInfos 成员
      this.userInfos = res.data.users
    },
    // 关闭dialog弹框
    addUserDialogClose() {
      this.$refs.addFormRef.resetFields()
    },
    // 进行客户端form表单验证
    addUser() {
      this.$refs.addFormRef.validate(async vaild => {
        // 校验成功的处理
        if (vaild) {
          const { data: res } = await this.$http.post('users', this.addForm)
          // 提交不成功处理
          if (res.meta.status !== 201) {
            return this.$message.error(res.meta.msg)
          }
          // 关闭弹框，提示成功信息，把新用户添加进去
          this.addUserDialog = false
          this.$message.success(res.meta.msg)
          this.getUserInfos()
        }
      })
    },
    // 删除用户
    delUser(id) {
      this.$confirm('确认删除该用户?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(async () => {
          // axios触发api接口删除
          const { data: res } = await this.$http.delete('users/' + id)
          if (res.meta.status !== 200) {
            return this.$message.error(res.meta.msg)
          }
          // 删除成功
          this.$message.success(res.meta.msg)
          this.getUserInfos()
        })
        .catch(() => {})
    },
    // 修改用户
    async showEditDialog(id) {
      this.editDialog = true
      // 把修改的用户添加到form表单中
      // 根据id查询被修改用户信息
      const { data: res } = await this.$http.get('users/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      // 接收被修改的用户信息
      this.editForm = res.data
    },
    // 接收信息入库存储
    async editUser() {
      const { data: res } = await this.$http.put(
        'users/' + this.editForm.id,
        this.editForm
      )
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      // 修改用户相关信息
      this.editDialog = false
      this.$message.success(res.meta.msg)
      this.getUserInfos()
    }
  },
  data() {
    // 自定义校验器
    // var checkMobile = (rule,value被校验数据，callback回调函数) =>{}
    var checkMobile = (rule, value, callback) => {
      // 校验手机号码规则
      if (!/^1[35789]\d{9}$/.test(value)) {
        // 校验失败
        return callback(new Error('手机号码不正确'))
      }
      // 校验通过
      callback()
    }
    return {
      // 添加用户相关, dialog弹框是否显示，true显示，false隐藏
      addUserDialog: false,
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      // form 表单校验
      addFormRules: {
        username: [{ required: true, message: '用户名必填', trigger: 'blur' }],
        password: [{ required: true, message: '密码必填', trigger: 'blur' }],
        mobile: [
          { required: true, message: '手机号码必填', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      // 修改用户相关, dialog弹框是否显示，true显示，false隐藏
      editDialog: false,
      editForm: {
        username: '',
        email: '',
        mobile: ''
      },
      // form 表单校验
      editFormRules: {
        mobile: [
          { required: true, message: '手机号码必填', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      // 接受用户数据
      userInfos: [],
      // 获取用户数据时，用到的查询条件参数
      queryParams: {
        query: '',
        pagenum: 1, // 当前页码
        pagesize: 3, // 每页显示总条数
        total: 0 // 记录总条数
      }
    }
  }
}
</script>
<style lang="less" scoped>
</style>
