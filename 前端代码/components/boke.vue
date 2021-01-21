<template>
  <div>
    <el-card class="box-card left">
      <div slot="header" class="clearfix">
        <i class="el-icon-s-data">博客量排行</i>
      </div>
      <el-table :data="blogList" style="width: 100%;" @row-click="viewother">
        <el-table-column label="排名" type="index"></el-table-column>
        <el-table-column label="姓名" prop="identity1"></el-table-column>
        <el-table-column label="博客总数" prop="count"></el-table-column>
      </el-table>
    </el-card>

    <div class="center">
      <el-menu router default-active="/newboke" class="el-menu-demo" mode="horizontal">
        <el-menu-item index="/newboke">最新博客</el-menu-item>
        <el-menu-item index="/hot">人气博客</el-menu-item>
      </el-menu>
      <div>
        <router-view></router-view>
      </div>
    </div>

    <el-card class="box-card right">
      <div slot="header" class="clearfix">
        <i class="el-icon-star-on">人气值排行</i>
      </div>
      <el-table :data="popularityList" style="width: 100%;" @row-click="viewother">
        <el-table-column label="排名" type="index"></el-table-column>
        <el-table-column label="姓名" prop="identity1"></el-table-column>
        <el-table-column label="人气总值" prop="sum"></el-table-column>
      </el-table>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      blogList: [], //博客量排行单
      popularityList: [] //人气值排行单
    };
  },
  mounted() {
    this.ranking();
  },
  methods: {
    ranking() {   //加载排行榜列表
      var that = this;
      const user = window.sessionStorage.getItem("user");
      const pass = window.sessionStorage.getItem("pass");
      this.$axios.get("./searchBlogList", {
        params: {
          username: user,
          password: pass,
        }
      }).then(function(res){
        // console.log(res);
        that.blogList = res.data.blogList
      })
      this.$axios.get("./searchPopularityList", {
        params: {
          username: user,
          password: pass,
        }
      }).then(function(res){
        // console.log(res);
        that.popularityList = res.data.popularityList
      })
    },
    viewother(row){
      // console.log(row.identity1);
      const name1 = window.sessionStorage.getItem("name");
      if (row.identity1 == name1) {
        this.$message.error("请点击“我的资料”！");
      } else {
        this.$router.push({
          name: "othermess", //路由跳转
          params: {
            name: row.identity1
          }
        });
      }
    }
  }
};
</script>

<style scoped>
.left {
  width: 280px;
  float: left;
  left: 1px;
}

.center {
  width: 690px;
  display: inline-block;
  right: 1px;
  left: 1px;
}

.right {
  width: 280px;
  float: right;
  right: 1px;
}
</style>