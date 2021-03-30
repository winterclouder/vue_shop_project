<template>
  <el-container class="home-container">
    <el-header class="">
    <div>
      <span>後台管理系統</span>
    </div>
      <el-button type="info" @click="logout">退出</el-button>
    </el-header>
    <el-container>
      <el-aside :width="collapse ? '64px' : '200px'">
        <div class="toggle-button" @click="toggleCollapse">|||</div>
        <el-menu background-color="#333744" :collapse-transition="false" :collapse="collapse" :default-active="activePath" router unique-opened text-color="#fff" active-text-color="#409EFE">
          <el-submenu :index="item.id.toString()" v-for="item in menuList"
            :key="item.id.toString()">
            <template slot="title">
              <i class="el-icon-location"></i>
              <span>{{item.authName}}</span>
            </template>
            <el-menu-item :index="'/' + subItem.path" v-for="subItem in item.children"
            :key="subItem.id.toString()" @click="saveNameState('/' + subItem.path)">
              <i class="el-icon-menu"></i>
              <span>{{subItem.authName}}</span>            
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <el-main>
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  created() {
    this.getMenuList()
    this.activePath = window.sessionStorage.getItem('activePath')
  },
  data() {
    return {
      menuList: [],
      iconsObj: {
        '125': 'iconfont icon-user',
        '103': 'iconfont icon-tijikongjian',
        '101': 'iconfont icon-shangpin',
        '102': 'iconfont icon-danju',
        '145': 'iconfont icon-baobiao'
      },
      collapse: false,
      activePath: ''
    }
  },
  methods: {
    saveNameState(path) {
      window.sessionStorage.setItem('activePath', path)
      this.activePath = path
    },
    toggleCollapse() {
      this.collapse = !this.collapse
    },
    async getMenuList() {
      const {data: res} = await this.$http.get('menus')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.menuList = res.data
    },
    logout() {
      window.sessionStorage.clear()
      this.$router.push('/login')
    }
  }
}
</script>

<style lang="less" scoped>
.home-container {
  height: 100%;
}
.el-header {
  background: #373d41;
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: white;
  font-size: 20px;
}
.el-aside {
  background: #333744;
  .el-menu {
    border-right: none;
  }
}
.el-main {
  background: #ededed;
}
.toggle-button {
  background: #4A5064;
  color: white;
  font-size: 10px;
  text-align: center;
  line-height: 24px;
  letter-spacing: 0.2em;
  cursor: pointer;
}
</style>
