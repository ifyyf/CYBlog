<template>
  <div>
    <el-card class="box-card">
      <div slot="header" class="clearfix">
        <el-input
          placeholder="关键字搜索"
          v-model="find"
          @keydown.enter.native="bloglist"
          @blur="bloglist"
          size="small"
          style="width: 150px; float:right;"
        ></el-input>
      </div>
      <div v-show="isshow">
        <div v-for="(it,index) in list" :key="index" class="text item">
          <el-link :underline="false">
            <div
              style="font-weight: bold; font-size:18px;"
              @click="detailedblog(it.blogtitle,it.identity1)"
            >{{it.blogtitle}}</div>
          </el-link>
          <div style="max-height: 65px; color:#606266; margin: auto; width: 500px;" v-html="it.blogbody"></div>
          <div style="float:right;">
            <el-link :underline="false">
              <div style="font-weight: bold;" @click="detailedmess(it.identity1)">{{it.identity1}}</div>
            </el-link>
            <div style="color:#C0C4CC;">人气值：{{it.popularity1}}</div>
            <div style="color:#C0C4CC;">{{it.time1}}</div>
          </div>
        </div>
        <br />
        <el-pagination
          background
          @current-change="bloglist"
          layout="prev, pager, next,total"
          :page-size="size"
          :total="total"
        ></el-pagination>
      </div>
      <!--分页组件-->
      <div v-show="!isshow">暂时没有相关博客,快写一个吧</div>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      find: "", //搜索内容
      list: [], //博客列表
      total: 0, //我的博客总条数
      size: 10, //每页10条
      isshow: true
    };
  },
  mounted() {
    this.bloglist();
  },
  methods: {
    bloglist(val = 1) {  //加载、搜索博客
      const user = window.sessionStorage.getItem("user");
      const pass = window.sessionStorage.getItem("pass");
      // console.log(val)
      var that = this;
      if (this.find == "") {      //没有进行关键字搜索
        if (!/^[0-9]+$/.test(val)) {
          val = 1;
        }
        that.isshow = true;
        this.$axios
          .get("./SearchAllBlog", {
            params: {
              page: val,
              username: user,
              password: pass
            }
          })
          .then(function(res) {
            // console.log(res.data);
            that.total = res.data.count[0].count;
            for (var i = 0; i < res.data.AllBlogs.length; i++) {
              res.data.AllBlogs[i].blogbody = res.data.AllBlogs[
                i
              ].blogbody.replace(/[\x00-\xff]/gi, ""); //去掉非中文
              if (res.data.AllBlogs[i].blogbody == "") {
                res.data.AllBlogs[i].blogbody = "无中文预览，请看详情";
              }
            }
            that.list = res.data.AllBlogs;
            window.scrollTo(0,0);
          });
      } else {        //进行关键字搜索博客
        if (!/^[0-9]+$/.test(val)) {
          val = 1;
        }
        // console.log("页码",val)
        this.$axios
          .get("./SearchAllBlog", {
            params: {
              page: val,
              username: user,
              password: pass,
              body: that.find
            }
          })
          .then(function(res) {
            // console.log(res.data);
            if (res.data.Blogs.length == 0) {
              that.isshow = false;
            } else {
              that.isshow = true;
              that.total = res.data.count[0].count;
              for (var i = 0; i < res.data.Blogs.length; i++) {
                res.data.Blogs[i].blogbody = res.data.Blogs[i].blogbody.replace(
                  /[\x00-\xff]/gi,
                  ""
                ); //去掉非中文
                if (res.data.Blogs[i].blogbody == "") {
                  res.data.Blogs[i].blogbody = "无中文预览，请看详情";
                }
              }
              that.list = res.data.Blogs;
              window.scrollTo(0,0);
            }
          });
      }
    },
    detailedblog(title,bokename) {  //博客详情
      this.$router.push({
        name: "viewboke", //路由跳转
        params: {
          title: title,
          bokename:bokename
        }
      });
    },
    detailedmess(name) {  //个人主页
      const name1 = window.sessionStorage.getItem("name");
      if (name == name1) {
        this.$message.error("请点击“我的资料”！");
      } else {
        this.$router.push({
          name: "othermess", //路由跳转
          params: {
            name: name
          }
        });
      }
    }
  }
};
</script>

<style scoped>
div {
  overflow: hidden;
}
</style>