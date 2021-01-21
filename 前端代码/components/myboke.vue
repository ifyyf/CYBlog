<template>
  <div>
    <el-card class="box-card">
      <el-table :data="tableData" style="width: 100%;">
        <el-table-column label="序号" type="index"></el-table-column>
        <el-table-column label="博客标题" prop="blogtitle"></el-table-column>
        <el-table-column label="博客内容" prop="blogbody"></el-table-column>
        <el-table-column label="博客人气" prop="popularity1" width="80px"></el-table-column>
        <el-table-column label="创作时间" prop="time1" width="180px"></el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-edit"
              @click="handleEdit(scope.$index, scope.row)"
              circle
            ></el-button>
            <el-button
              type="danger"
              icon="el-icon-delete"
              @click="delopen(scope.$index, scope.row)"
              circle
            ></el-button>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
        v-show="tableData.length != 0"
        background
        @current-change="listmy"
        layout="prev, pager, next,total"
        :page-size="size"
        :total="total"
      ></el-pagination>    <!--分页组件--> 
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      dialogVisible: false,
      total:0,     //我的博客总条数
      size: 10,      //页面的条数
      tableData: [],
      page:"",
    };
  },
  mounted() {
    this.listmy();
  },
  methods: { 
    listmy(page = 1) {      //加载我的博客
      const user = window.sessionStorage.getItem("user");
      const pass = window.sessionStorage.getItem("pass");
      this.page = page;
      var that = this;
      this.$axios
        .get("./SearchUserBlog", {
          params: {
            username: user,
            password: pass,
            page:page,
          }
        })
        .then(function(res) {
          // console.log(res.data);
          that.total = res.data.count[0].count;
          for (var i = 0; i < res.data.AllBlogs.length; i++) {
            res.data.AllBlogs[i].blogbody = res.data.AllBlogs[i].blogbody.replace(/[\x00-\xff]/ig,"")   //去掉非中文
            res.data.AllBlogs[i].blogbody = res.data.AllBlogs[i].blogbody.substring(0, 25);      //保留25个字符
            if(res.data.AllBlogs[i].blogbody == ""){
              res.data.AllBlogs[i].blogbody = "无中文预览，请看详情"
            }
          }
          that.tableData = res.data.AllBlogs;
          window.scrollTo(0,0);
        });
    },
    handleEdit(index, row) {      //改博客
      // console.log(index, row);
      const user = window.sessionStorage.getItem("user");
      const pass = window.sessionStorage.getItem("pass");
      var that = this;
      this.$axios
        .get("./SearchUserBlog", {
          params: {
            username: user,
            password: pass,
            page:that.page,
          }
        })
        .then(function(res) {
          // console.log(res);
          that.$router.push({
            name: "addboke",
            params: {
              category:res.data.AllBlogs[index].category,
              title: row.blogtitle,
              body: res.data.AllBlogs[index].blogbody,
            }
          });
        });
    },
    delopen(index, row) {   //删除博客
      this.$confirm("此操作将永久删除该博客, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          // console.log(index,row.blogtitle);
          const user = window.sessionStorage.getItem("user");
          const pass = window.sessionStorage.getItem("pass");
          var that = this;
          this.$axios
            .get("./DeleteBlog", {
              params: {
                blogtitle: row.blogtitle,
                username: user,
                password: pass
              }
            })
            .then(function(res) {
              // console.log(res.data);
              if (res.data == "yes") {
                that.$message({
                  type: "success",
                  message: "删除成功!"
                });
                that.listmy();
              } else {
                that.$message.error("删除失败！！");
              }
            });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除"
          });
        });
    }
  },
};
</script>

<style scoped>
</style>