<template>
  <el-container class="home-container">
    <el-header >
      <div>
        <img src="../assets/heima.png" alt="" >
        <span>电商后台管理系统</span>
      </div>
      <el-button class="login_out_btn" type="info" @click="logout">退出</el-button></el-header>
    <el-container>
      <el-aside :width="isCollapse ? '64px' : '200px' ">
<!--        侧边栏-->
        <div class="toggle" @click="toggleCollapse">
          |||
        </div>
        <el-menu
          :default-active="activePath"
          router
          :collapse-transition="false"
          :collapse="isCollapse"
          unique-opened
          background-color="#333744"
          text-color="#fff"
          active-text-color="#409BFF">
<!--          一级菜单-->
          <el-submenu :index="item.id + ''" v-for="item in menuList" :key="item.id">
<!--            一级菜单模板区-->
            <template slot="title">
<!--              图标-->
              <i :class="iconsObj[item.id]"></i>
<!--              文本-->
              <span>{{item.authName}}</span>
            </template>
<!--            二级菜单-->
              <el-menu-item @click="saveNavState('/' + c.path)" :index="'/' + c.path" v-for="c in item.children" :key="c.id">
                <template slot="title">
                  <!--              图标-->
                  <i class="el-icon-menu"></i>
                  <!--              文本-->
                  <span>{{c.authName}}</span>
                </template>
              </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <el-main>
        <router-view>
        </router-view></el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
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
      isCollapse: false,
      activePath: ''
    }
  },
  created() {
    this.getMenuList()
    this.activePath = window.sessionStorage.getItem('acticvepath')
  },
  methods: {
    logout() {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    async getMenuList() {
      const { data: res } = await this.$http.get('menus')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.menuList = res.data
      console.log(this.menuList[0].authName)
    },
    toggleCollapse() {
      this.isCollapse = !this.isCollapse
    },
    saveNavState(activepath) {
      window.sessionStorage.setItem('acticvepath', activepath)
      this.activePath = window.sessionStorage.getItem('acticvepath')
    }
  }
}
</script>

<style lang="less" scoped>
  .el-header
  {
    background-color: #373d41;
    display:flex;
    flex-direction:row;
    justify-content: space-between;
    padding-left: 0;
    align-items: center;
    color: #ffffff;
    font-size: 20px;
    > div
    {
      display:flex;
      flex-direction:row;
      align-items: center;
      > span
      {
        margin-left: 10px;
      }
    }
  }
  .el-aside
  {
    background-color: #333744;
    .el-menu{
      border-right: 0;
    }
  }
  .el-main
  {
    background-color: #eaedf1;
  }
  .home-container
  {
    width: 100%;
    height: 100%;
  }
  .iconfont
  {
    margin-right: 10px;
  }
  .toggle
  {
    background-color: #4A5064;
    font-size: 10px;
    line-height: 24px;
    color: #ffffff;
    text-align: center;
    letter-spacing: 0.2em;
  }
</style>
