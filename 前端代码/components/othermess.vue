<template>
  <div>
    <el-card>
      <div style="float:left;">
        <div style="display: inline-block;">
          <el-avatar shape="square" fit="cover" :src="imgurl"></el-avatar>
        </div>
        <div class="title">
          <div style="font-weight: bold; font-size: 20px;">{{usermessg.identity1}}</div>
          <br />
          <br />
          <div style="color:#909399;">{{usermessg.time1}}注册</div>
        </div>
        <ul>
          <li>
            <p style="color:#909399;">性别：</p>
            <p style="color:#606266;">{{usermessg.sex}}</p>
          </li>
          <li>
            <p style="color:#909399;">博客量：</p>
            <p style="color:#606266;">{{blognum}}</p>
          </li>
          <li>
            <p style="color:#909399;">人气值：</p>
            <p style="color:#606266;">{{bloghot}}</p>
          </li>
          <li>
            <p style="color:#909399;">收到评论：</p>
            <p style="color:#606266;">{{commentnum}}</p>
          </li>
          <li>
            <p style="color:#909399;">关注：</p>
            <p style="color:#606266;">{{concern}}</p>
          </li>
          <li>
            <p style="color:#909399;">粉丝：</p>
            <p style="color:#606266;">{{fans}}</p>
          </li>
        </ul>
        <div class="concernbtn" v-show="concernbtnshow">
           <el-button type="primary" round @click="concerta">关注博主</el-button>
        </div>
        <div class="concernbtn" v-show="!concernbtnshow">
           <el-button type="info" round @click="concerta">取消关注</el-button>
        </div>
      </div>
    </el-card>

    <div style="width: 800px; display: inline-block; float: left;">
      <el-card>
        <el-table :data="tableData" style="width: 100%;">
          <el-table-column label="序号" type="index"></el-table-column>
          <el-table-column label="博客标题" prop="blogtitle" width="200px"></el-table-column>
          <el-table-column label="博客内容" prop="blogbody" width="160px"></el-table-column>
          <el-table-column label="博客人气" prop="popularity1" width="80px"></el-table-column>
          <el-table-column label="创作时间" prop="time1" width="180px"></el-table-column>
          <el-table-column label="操作">
            <template slot-scope="scope">
              <el-button
                type="primary"
                icon="el-icon-view"
                @click="handleView(scope.$index, scope.row)"
                circle
              ></el-button>
            </template>
          </el-table-column>
        </el-table><br>
        <el-pagination
          background
          @current-change="bokechange"
          layout="prev, pager, next,total"
          :page-size="size"
          :total="total"
        ></el-pagination>
        <!--分页组件-->
      </el-card>
    </div>

    <div style="width: 460px; display: inline-block; float: right;">
      <el-card>
        <div style="text-align: left; color:#909399;">
          地址：
          <el-card class="box-card">
            <div>{{usermessg.address}}</div>
          </el-card>
        </div>
        <br />
        <div style="text-align: left; color:#909399;">
          个人简介：
          <el-card class="box-card">
            <div>{{usermessg.personShow}}</div>
          </el-card>
        </div>
      </el-card>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      size: 10, //每页5条
      total: 0, //共计数据条数
      tableData: [], //博客列表
      imgurl: "", //头像
      blognum: "", //博客量
      bloghot: "", //人气值
      usermessg: {}, //TA的信息表
      identity1: "", //TA的姓名
      commentnum: "", //总评论数
      fans:"",  //粉丝数
      bokename:'',    
      concern:"", //关注数
      concernbtnshow:true,  //关注按钮
    };
  },
  mounted() {
    this.loding();
    this.bokechange();
    this.searchIfFollow();
  },
  methods: {
    loding() {  //加载别人信息
      var name = this.$route.params.name;
      this.bokename = name;
      if (name != undefined) {
        window.sessionStorage.setItem("othername", name);
      }
      const othername = window.sessionStorage.getItem("othername");
      this.identity1 = othername;
      const user = window.sessionStorage.getItem("user");
      const pass = window.sessionStorage.getItem("pass");
      var that = this;
      this.$axios
        .get("./otherUserData", {
          params: {
            username: user,
            password: pass,
            identity1: othername
          }
        })
        .then(function(res) {
          // console.log(res.data);
          if (res.data.user.address == "" || res.data.user.personShow == "") {
            res.data.user.address = "用户没有填写";
            res.data.user.personShow = "用户没有填写";
          }
          that.usermessg = res.data.user;
          that.bloghot = res.data.popularity1;
          that.blognum = res.data.number;
          that.commentnum = res.data.commentCount[0].commentCount;
        });
      this.$axios
        .get("./searchImg?identity1=" + othername)
        .then(function(res) {
          //   console.log(res.data);
          that.imgurl =  './' + res.data;
        });
      this.$axios.get("./searchCount",{
        params:{
          username: user,
          password: pass,
          identity1: othername
        }
      }).then(function(res){
        // console.log(res.data);
        that.fans = res.data.fansCount[0].fansCount;
        that.concern = res.data.followCount[0].followCount;
      })
    },
    bokechange(page = 1) {  //加载博客、换页
      const user = window.sessionStorage.getItem("user");
      const pass = window.sessionStorage.getItem("pass");
      var that = this;
      this.$axios
        .get("./searchOtherUserBlog", {
          params: {
            username: user,
            password: pass,
            identity1: that.identity1,
            page: page
          }
        })
        .then(function(res) {
          //   console.log(res.data);
          that.total = res.data.count[0].count;
          for (var i = 0; i < res.data.AllBlogs.length; i++) {
            res.data.AllBlogs[i].blogbody = res.data.AllBlogs[
              i
            ].blogbody.replace(/[\x00-\xff]/gi, ""); //去掉非中文
            res.data.AllBlogs[i].blogbody = res.data.AllBlogs[
              i
            ].blogbody.substring(0, 25); //保留100个字符
            if (res.data.AllBlogs[i].blogbody == "") {
              res.data.AllBlogs[i].blogbody = "无中文预览，请看详情";
            }
          }
          that.tableData = res.data.AllBlogs;
          window.scrollTo(0,0);
        });
    },
    handleView(index, row) {  //观看博客
      // console.log(index,row)
      var title = row.blogtitle;
      this.$router.push({
        name: "viewboke", //路由跳转
        params: {
          title: title,
          bokename:this.bokename,
        }
      });
    },
    concerta(){     //关注TA
      // this.concernbtnshow = !this.concernbtnshow;
      const name = window.sessionStorage.getItem("name");
      const user = window.sessionStorage.getItem("user");
      const pass = window.sessionStorage.getItem("pass");
      var that = this;
      if(this.concernbtnshow == false){
        this.$confirm('将不再关注博主, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$axios.get("./addFollow",{
            params:{
              identity1:that.identity1,
              identity2:name,
              username: user,
              password: pass,
            }
          }).then(function(res){
            // console.log(res.data)
            if(res.data == "yes"){
              that.$message({
                type: 'success',
                message: '取关成功，博主快哭了'
              });
              that.searchIfFollow();
              that.loding()
            }
          })
        }).catch(() => {
          that.$message({
            type: 'info',
            message: '已取消操作'
          });          
        });
      }else{
        this.$axios.get("./addFollow",{
            params:{
              identity1:that.identity1,
              identity2:name,
              username: user,
              password: pass,
            }
          }).then(function(res){
            // console.log(res.data)
            if(res.data == "yes"){
              that.$message({
                type: 'success',
                message: '关注成功，博主可高兴了'
              });
              that.searchIfFollow();
              that.loding()
            }
          })
      }
    },
    searchIfFollow(){   //看关注状态
      const name = window.sessionStorage.getItem("name");
      const user = window.sessionStorage.getItem("user");
      const pass = window.sessionStorage.getItem("pass");
      var that = this;
      this.$axios.get("./searchIfFollow",{
        params:{
          identity1:that.identity1,
          identity2:name,
          username: user,
          password: pass,
        }
      }).then(function(res){
        // console.log(res.data);
        if(res.data == "yes"){
          that.concernbtnshow = false;
        }else{
          that.concernbtnshow = true;
        }
      })
    }
  }
};
</script>

<style scoped>
.title {
  /*个人主要信息样式*/
  width: 140px;
  display: inline-block;
  position: relative;
  left: 80px;
}
ul {
  list-style: none;
  display: inline-block;
  position: relative;
  top: -20px;
  left: 100px;
}
ul li {
  display: inline-block;
  padding: 0px 20px;
  border-right: 1px solid #c0c4cc;
}

.concernbtn {
  display: inline-block;
  position: relative;
  top:-30px;
  left: 180px;
} /*关注按钮样式 */

.el-avatar {
  width: 100px;
  height: 100px;
  display: inline-block;
}
</style>