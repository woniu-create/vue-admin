<template>
  <el-container class="home-container">
    <el-header>
      <div>
        <img src="../assets/logo.png" alt="" />
        <span>后台管理系统</span>
      </div>
      <el-button type="info" @click="logout">退出</el-button>
    </el-header>
    <el-container>
      <el-aside :width="isCollapse ? '64px' : '200px'">
        <div class="toggle-button" @click="toggleCollapse">|||</div>
        <el-menu class="el-menu-vertical-demo" background-color="#373d41" text-color="#fff" active-text-color="#3991ff"
          unique-opened
          :collapse = "isCollapse"
          :collapse-transition = "false"
          router
          :default-active="activePath"
        >
          <el-submenu :index="item.id+''" v-for="item in muneList" :key="item.id">
            <template slot="title">
              <i :class="iconsObj[item.id]"></i>
              <span>{{ item.authName }}</span>
            </template>
            <el-menu-item :index="'/'+subItem.path" v-for="subItem in item.children" :key="subItem.id" @click="saveNavState('/' + subItem.path)">
              <template slot="title">
                <i class="el-icon-menu"></i>
                <span>{{ subItem.authName }}</span>
              </template>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <el-main><router-view></router-view></el-main>
    </el-container>
  </el-container>
</template>
<script>
export default {
  data() {
    return {
      muneList: [],
      iconsObj: {
        125: 'iconfont icon-user',
        103: 'iconfont icon-tijikongjian',
        101: 'iconfont icon-shangpin',
        102: 'iconfont icon-danju',
        145: 'iconfont icon-baobiao'
      },
      isCollapse: false,
      activePath: ''
    }
  },
  created() {
    this.getMenuList()
    this.activePath = window.sessionStorage.getItem('activePath')
  },
  methods: {
    logout() {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    async getMenuList() {
      const { data: res } = await this.$http.get('menus')
      console.log(res)
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.muneList = res.data
    },
    toggleCollapse() {
      this.isCollapse = !this.isCollapse
    },
    saveNavState(activePath) {
      window.sessionStorage.setItem('activePath', activePath)
      this.activePath = activePath
    }
  }
}
</script>
<style lang="less" scoped>
.home-container {
  height: 100%;
}
.el-header {
  background-color: #373d41;
  display: flex;
  color: #fff;
  align-items: center;
  font-size: 20px;
  justify-content: space-between;
  padding-left: 0;
  > div {
    display: flex;
    align-items: center;
    span {
      margin-left: 15px;
    }
    img {
      width: 50px;
    }
  }
}

.el-aside {
  background-color: #373d41;
  // color: #333;
  // text-align: center;
  // line-height: 200px;
  .el-menu {
    border-right: none;
  }
}

.el-main {
  background-color: #e9eef3;
  // color: #333;
  // text-align: center;
  // line-height: 160px;
}
.iconfont {
  margin-right: 10px;
}
.toggle-button {
  text-align: center;
  color: #fff;
  background-color: #4a5064;
  font-size: 10px;
  line-height: 24px;
  letter-spacing: 0.2em;
  cursor: pointer;
}
</style>
