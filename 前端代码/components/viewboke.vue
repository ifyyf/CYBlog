<template>
  <div>
    <el-card class="box-card leftcard">
      <div style="font-size: 40px; text-align: left; margin-left: 10px;">
        {{blogtitle}}
        <el-button
          style="float: right; position: relative; top:8px;"
          type="danger"
          icon="el-icon-back"
          circle
          plain
          @click="out"
        ></el-button>
      </div><br>
      <div class="lessmess" @click="todeltile(identity1)">
          <el-avatar shape="square" :size="50" fit="cover" :src="imgurl"></el-avatar>
          <div style="margin-left: 10px;">
            <div style="color:#000000; font-weight: bold; font-size: 18px; margin-bottom: 10px;">{{identity1}}</div>
            <div style="color:#909399; display:inline-block;">{{time1}}</div>
            <div style="color:#909399; display:inline-block; margin-left: 10px;">人气：{{popularity1}}</div>
          </div>
      </div>
      <div style="text-align: left;">
          <div class="ql-snow">
            <div class="ql-editor content" v-html="blogbody"></div>
          </div>
      </div>
    </el-card>

    <el-card class="box-card rightcard">
      <div slot="header" class="clearfix">
        <span>博客评论</span>
      </div>
      <div>
        <div>
          <el-input
            ref="mark"
            placeholder="请输入您的评论"
            v-model="commentinput"
            @input="giverep"
            clearable
          ></el-input>
        </div>
        <br />
        <div v-show="btnshow">
          <el-button type="primary" plain style="width:100%;" @click="upcomment">添加评论</el-button>
        </div>
        <div v-show="!btnshow">
          <el-button type="primary" plain style="width:100%;" @click="upreply">确认回复</el-button>
        </div>
      </div>
      <br />
      <div v-show="isshowcom" v-for="(it,index) in Commentlist" :key="index" class="text item">
        <div class="avatar">
          <el-avatar :src=" './' + it[0].imgPath" :size="50"></el-avatar>
        </div>
        <div class="commentmess">
          <div>
            <div class="commentaname">{{it[0].identity1}}</div>
            <el-button
              size="mini"
              v-show="it[0].identity1 == name"
              @click="delcomment(it[0].commentbody)"
              icon="el-icon-delete"
              circle
              class="delbtn"
              type="danger"
            ></el-button>
          </div>
          <div style="text-align: left; word-wrap: break-word; color:#606266">{{it[0].commentbody}}</div>
          <br />
          <div>
            <div style="float: left; color: #C0C4CC;" @click="reply(it[0].id,it[0].identity1)">
              <i class="el-icon-chat-line-round">回复</i>
            </div>
            <div style="display: inline-block; text-align: right; color: #C0C4CC;">{{it[0].time1}}</div>

            <div>
              <el-collapse style="width:230px;">
                <el-collapse-item title="查看回复">
                  <div>
                    <div v-for="(item,index) in it[1]" :key="index" v-show="it[1].length != 0">
                      <div
                        style="display: inline-block; position: relative; height:40px ; top: -25px; left: -10px;"
                      >
                        <el-avatar :src=" './' + item.imgpath" :size="40"></el-avatar>
                      </div>
                      <div style="display: inline-block; width:170px;">
                        <div>
                          <div
                            style="display: inline-block; position: relative; left: -25px; color:#909399;"
                          >{{item.identity1}}&nbsp;回复&nbsp;{{item.reidentity1}}</div>
                          <el-button
                            size="mini"
                            v-show="item.identity1 == name"
                            @click="deleteReply(item.replybody,item.reidentity1)"
                            icon="el-icon-delete"
                            circle
                            style="display: inline-block; position: relative; right: -30px;"
                            type="danger"
                          ></el-button>
                        </div>
                        <div
                          style="text-align: left; word-wrap: break-word; color:#303133;"
                        >{{item.replybody}}</div>
                        <div>
                          <div style="text-align: right; color: #C0C4CC;">{{item.time1}}</div>
                        </div>
                      </div>
                    </div>
                    <div v-show="it[1].length == 0" style="font-size: 15px;">该评论没有回复，快抢沙发！！</div>
                  </div>
                </el-collapse-item>
              </el-collapse>
            </div>
          </div>
        </div>
      </div>
      <div v-show="!isshowcom" style="font-size: 15px;">该博客没有评论，快抢沙发！！</div>
      <el-pagination
        v-show="isshowcom"
        background
        @current-change="allcomment"
        layout="prev, pager, next,total"
        :page-size="size"
        :total="total"
      ></el-pagination>
    </el-card>
    <el-button type="primary" icon="el-icon-upload2" circle class="topbtn" @click="totop" v-show="topshow"></el-button>
  </div>
</template>

<script>
import { quillEditor } from "vue-quill-editor";
import "quill/dist/quill.snow.css";

export default {
  data() {
    return {
      size: 3,
      total: 0, //评论总数
      btnshow: true,
      topshow: false,
      imgurl:'',    //博主头像
      blogbody: "", //博客内容
      category:'',    //博客分类
      identity1:'',   //博主姓名
      time1:'',       //操作时间
      popularity1:'',   //博客人气
      blogtitle: "", //博客标题
      commentinput: "", //评论输入内容
      Commentlist: [], //评论列表
      comid: "", //评论id
      name: "", //登录者的姓名
      isshowcom: true //是否没有评论
    };
  },
  mounted() {
    this.viewboke();
    this.allcomment();
    window.addEventListener('scroll', this.scrollToTop)
  },
  destroyed () {
    window.removeEventListener('scroll', this.scrollToTop)
  },
  methods: {
    viewboke() {  //看博客
      var title = this.$route.params.title;
      var bokename = this.$route.params.bokename;
      window.scrollTo(0, 0);
      window.sessionStorage.setItem("oldtitle", title);
      const user = window.sessionStorage.getItem("user");
      const name = window.sessionStorage.getItem("name");
      const pass = window.sessionStorage.getItem("pass");
      this.name = name;
      // console.log(title);
      var that = this;
      this.$axios
        .get("./ReadBlog", {
          params: {
            blogtitle: title,
            username: user,
            password: pass
          }
        })
        .then(function(res) {
            // console.log(res);
          that.blogbody = res.data.blogbody;
          that.blogtitle = res.data.blogtitle;
          that.category = res.data.category;
          that.identity1 = res.data.identity1;
          that.time1 = res.data.time1;
          that.popularity1 = res.data.popularity1;    
        });
      this.$axios
        .get("./searchImg?identity1=" + bokename)
        .then(function(res) {
            // console.log(res.data);
          that.imgurl =  './' + res.data;
        });
    },
    todeltile(othermess){
      const name1 = window.sessionStorage.getItem("name");
      if (othermess == name1) {
        this.$message.error("请点击“我的资料”！");
      } else {
        this.$router.push({
          name: "othermess", //路由跳转
          params: {
            name: othermess
          }
        });
      }
    },
    allcomment(page = 1) {  //加载该博客所有评论
      var that = this;
      const user = window.sessionStorage.getItem("user");
      const blogtitle = window.sessionStorage.getItem("oldtitle");
      const pass = window.sessionStorage.getItem("pass");
      this.$axios
        .get("./searchAllComment", {
          params: {
            blogtitle: blogtitle,
            username: user,
            password: pass,
            page: page
          }
        })
        .then(function(res) {
          // console.log(res.data);
          that.total = res.data.BlogCount[0].count;
          if (res.data.AllCommentAndReply.length == 0) {
            that.isshowcom = false;
          } else {
            that.Commentlist = res.data.AllCommentAndReply;
          }
        });
    },
    upcomment() {  //添加评论
      if (this.commentinput != "") {
        var that = this;
        const user = window.sessionStorage.getItem("user");
        const blogtitle = window.sessionStorage.getItem("oldtitle");
        const pass = window.sessionStorage.getItem("pass");
        this.$axios
          .get("./addComment", {
            params: {
              identity1: that.name,
              blogtitle: blogtitle,
              username: user,
              password: pass,
              commentbody: that.commentinput
            }
          })
          .then(function(res) {
            // console.log(res.data);
            if (res.data == "yes") {
              that.$message({
                message: "评论博客成功",
                type: "success"
              });
              that.allcomment();
              that.isshowcom = true;
              that.commentinput = "";
            } else {
              that.$message.error("评论内容重复，评论失败");
            }
          });
      } else {
        this.$message({
          message: "评论不能为空",
          type: "warning"
        });
      }
    },
    delcomment(commentbody) {  //删除评论
      var that = this;
      this.$confirm("此操作将永久删除该评论, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          const user = window.sessionStorage.getItem("user");
          const blogtitle = window.sessionStorage.getItem("oldtitle");
          const pass = window.sessionStorage.getItem("pass");
          this.$axios
            .get("./deleteComment", {
              params: {
                identity1: that.name,
                blogtitle: blogtitle,
                username: user,
                password: pass,
                commentbody: commentbody
              }
            })
            .then(function(res) {
              // console.log(res.data);
              if (res.data == "yes") {
                that.$message({
                  type: "success",
                  message: "删除成功!"
                });
                that.allcomment();
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
    },
    reply(comid, name) {  //预备回复
      this.btnshow = false;
      this.commentinput = "@" + name + ":";
      this.$refs.mark.$el.querySelector("input").focus(); //获取焦点
      this.comid = comid;
      window.sessionStorage.setItem("replyname", name);
    },
    upreply() {  //确认回复
      var commentinput = this.commentinput.split(":")[1];
      if (commentinput != "") {
        const user = window.sessionStorage.getItem("user");
        const blogtitle = window.sessionStorage.getItem("oldtitle");
        const pass = window.sessionStorage.getItem("pass");
        const id = window.sessionStorage.getItem("comid");
        const replyname = window.sessionStorage.getItem("replyname");
        var that = this;
        this.$axios
          .get("./addReply", {
            params: {
              reidentity1: replyname,
              commentid: that.comid,
              blogtitle: blogtitle,
              replybody: that.commentinput,
              username: user,
              password: pass
            }
          })
          .then(function(res) {
            // console.log(res.data);
            if (res.data == "yes") {
              that.$message({
                message: "回复“" + replyname + "”成功！",
                type: "success",
                center: true
              });
              that.btnshow = true;
              that.allcomment();
              that.commentinput = "";
            } else {
              that.$message.error("回复内容重复，回复失败！！");
            }
          });
      } else {
        this.$message({
          message: "回复内容为空，回复失败！！",
          type: "warning"
        });
      }
    },
    giverep() {  //放弃回复
      if(this.btnshow == false){
        const name = window.sessionStorage.getItem("replyname");
        var rephead = "@" + name + ":";
        if (this.commentinput < rephead) {
          this.commentinput = "";
          this.btnshow = true;
        }
      }
    },
    deleteReply(replybody, reidentity1) {  //删除回复
      // console.log(replybody, reidentity1)
      this.$confirm("此操作将永久删除该回复, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          const repid = window.sessionStorage.getItem("repid");
          const user = window.sessionStorage.getItem("user");
          const blogtitle = window.sessionStorage.getItem("oldtitle");
          const pass = window.sessionStorage.getItem("pass");
          var that = this;
          this.$axios
            .get("./deleteReply", {
              params: {
                username: user,
                password: pass,
                replybody: replybody,
                reidentity1: reidentity1,
                blogtitle: blogtitle
              }
            })
            .then(function(res) {
              if (res.data == "yes") {
                that.$message({
                  type: "success",
                  message: "删除成功!"
                });
                that.allcomment();
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
    },
    scrollToTop () {
      const that = this
      let scrollTop = window.pageYOffset || document.documentElement.scrollTop || document.body.scrollTop
      that.scrollTop = scrollTop
      if (that.scrollTop > 60) {
        that.topshow = true
      } else {
        that.topshow = false
      }
    },
    totop(){
      window.scrollTo(0,0);
    },
    out() {
      //返回
      this.$router.go(-1);
    }
  }
};
</script>

<style scoped>
.lessmess{
  text-align: left;
  margin-left: 10px;
  display: flex;
}
.content {
  width: 100%;
  word-wrap: break-word; /*内容超过宽就自动换行*/
}
.leftcard {
  display: inline-block;
  width: 940px;
}
.rightcard {
  width: 300px;
  float: right;
  right: 1px;
}
.avatar {
  display: inline-block;
  width: 63px;
  position: relative;
  top: 5px;
  left: -120px;
}
.commentmess {
  display: inline-block;
  width: 250px;
  min-height: 30px;
  position: relative;
  top: -50px;
  right: 0px;
  left: 40px;
}
.commentaname {
  display: inline-block;
  color: #303133;
  position: relative;
  left: -90px;
  bottom: 6px;
}
.delbtn {
  display: inline-block;
  position: relative;
  right: -70px;
}
.topbtn{
  float: left;
  position: fixed;
  top: 600px;
  left: 1150px;
}
</style>