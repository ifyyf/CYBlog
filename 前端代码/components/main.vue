<template>
  <div>
    <el-container>
      <el-header>
        <span>
          <el-avatar :src="src" class="Headpht"></el-avatar>
         <div class="name">{{name}}</div>
        </span>
        <span class="title">博客园</span>
        <el-button
          type="danger"
          plain
          size="small"
          style="float: right; position: relative; top:12px;"
          @click="out"
        >退出登录</el-button>
      </el-header>
      <el-container>
        <el-aside width="200px">
          <el-menu
            router
            default-active="/newboke"
            class="el-menu-vertical-demo"
            background-color="#000000"
            text-color="#fff"
            active-text-color="#409EFF"
          >
            <el-menu-item index="/newboke">
              <i class="el-icon-s-home"></i>
              <span slot="title">博客页面</span>
            </el-menu-item>
            <el-menu-item index="/myboke">
              <i class="el-icon-s-custom"></i>
              <span slot="title">我的资料</span>
            </el-menu-item>
            <el-menu-item index="/addboke">
              <i class="el-icon-document-add"></i>
              <span slot="title">新增博客</span>
            </el-menu-item>
            <!-- <el-menu-item index="/othermess">
              <i class="el-icon-document-add"></i>
              <span slot="title">TA的详情</span>
            </el-menu-item> -->
            <el-menu-item></el-menu-item>
            <el-menu-item></el-menu-item>
            <el-menu-item></el-menu-item>
            <el-menu-item></el-menu-item>
            <el-menu-item></el-menu-item>
            <el-menu-item></el-menu-item>
            <el-menu-item></el-menu-item>
            <el-menu-item></el-menu-item>
            <el-menu-item></el-menu-item>
          </el-menu>
        </el-aside>
        <el-container>
          <el-main>
            <router-view></router-view>
          </el-main>
          <el-footer></el-footer>
        </el-container>
      </el-container>
    </el-container>
  </div>
</template>
 
<script>
export default {
  data() {
    return {
      src: "./img/freeBlog.png",
      name: ""
    };
  },
  mounted() {
    this.getmessage();
  },
  methods: {
    getmessage() {
      var that = this;
      const name = window.sessionStorage.getItem("name");
      this.name = name;
    },
    out() {
      //退出登录
      this.$confirm("此操作将退出登录, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          this.$router.push("/");
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消操作"
          });
        });
    }
  }
};
</script>

<style scoped>
.el-header {
  background-color: #000000;
  color: #fff;
  line-height: 60px;
}
.name{
  font-size: 18px;
  display: inline-block;
  position: relative;
  left: 10px;
  top: -10px;
}
.Headpht {
  position: relative;
  top: 3px;
}
.title {
  font-size: 35px;
  position: relative;
  left: 680px;
}
.el-footer {
  background-color: #000000;
  color: #fff;
  text-align: center;
  font-size: 20px;
  line-height: 60px;
}
.el-aside {
  background-color: #000000;
  color: #333;
  text-align: center;
}

.el-main {
  background-color: #ffffff;
  color: #333;
  text-align: center;
}
</style>