<template>
  <el-container>
    <el-header>
      <div id="logo-box">
        <img src="../assets/img/heima.png" alt>
        <span>电商后台管理系统</span>
      </div>
      <el-button type="info" @click="layout">退出</el-button>
    </el-header>
    <el-container>
      <el-aside :width="menushow?'65px':'200px'">
        <div
          style="width:100%; height:25px; line-height:25px; background:#4a5064;color:#fff;text-align:center;font-size:12px; cursor:pointer; letter-spacing:0.1em;"
          @click="menushow=!menushow"
        >|||</div>
        <el-menu
          default-active="2"
          background-color="#333744"
          text-color="#fff"
          active-text-color="#409eff"
          :style="menushow?'width:65px;':'width:200px;'"
          :unique-opened="true"
          :collapse="menushow"
          :collapse-transition="false"
        >
          <el-submenu :index="item.id+''" v-for="(item,k) in menuList" :key="item.id">
            <template slot="title">
              <i :class="'iconfont icon-' + menuicon[k]"></i>
              <span>{{item.authName}}</span>
            </template>
            <el-menu-item
              :index="item.id +'-'+ item2.id"
              v-for="item2 in item.children"
              :key="item2.id"
            >
              <i class="el-icon-menu"></i>
              <span>{{item2.authName}}</span>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <el-main>
        <router-view />
      </el-main>
    </el-container>
  </el-container>
</template>
<script>
export default {
  //生命周期函数
  created() {
    //获得左侧按钮列表数据
    this.getMenuList()
  },
  data() {
    return {
      //设定左侧菜单收起展开
      menushow: false,
      menuList:[],  //接受左侧功能按钮数据
      menuicon:['users','tijikongjian','shangpin','danju','baobiao']
    }
  },
  methods: {
    //获得左侧功能按钮数据
    async getMenuList() {
      const { data: res } = await this.$http.get('menus')
      //获取数据失败
      if (res.meta.status != 200) {
        return this.$message.error(res.meta.msg)
      }
      //把获取好的数据赋值给menuList
      this.menuList = res.data
    },
    layout() {
      this.$confirm('确定退出登录?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(() => {
          window.sessionStorage.removeItem('token')
          this.$router.push('/login')
        })
        .catch(() => {})
    }
  }
}
</script>
<style lang="less" scoped>
.el-container {
  height: 100%;
  .el-header {
    width: 100%;
    height: 60px;
    background: #373d41;
    padding: 0;
    display: flex;
    align-items: center;
    justify-content: space-between;
    #logo-box {
      display: flex;
      color: #fff;
      font-size: 22px;
      align-items: center;
      user-select: none;
      img {
        width: 50px;
        height: 50px;
        margin-right: 10px;
      }
    }
    .el-button {
      margin-right: 20px;
    }
  }
  .el-aside {
    background-color: #333744;  
  }
  .el-main {
    background-color: #e9eef3;
  }
}
</style>
