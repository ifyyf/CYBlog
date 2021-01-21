<template>
  <el-card class="box-card">
    <div class="title">
      {{title}}
      <el-button
        style="float: right; position: relative; top:0px;"
        type="danger"
        icon="el-icon-back"
        circle
        plain
        @click="out"
      ></el-button>
    </div>
    <br />
    <div>
      <div>
        <div class="left">博客标题：</div>
        <el-input placeholder="请输入标题" v-model="blogtitle" clearable style="width: 1140px;"></el-input>
      </div>
      <br />
      <div class="left">博客分类：</div>
      <div style="text-align: left;">
        <el-select v-model="sort" filterable allow-create placeholder="请选择博客的分类">
          <el-option
            v-for="(item,index) in options"
            :key="index"
            :label="item.category"
            :value="item.category"
          ></el-option>
          <!--博客分类-->
        </el-select>
        <el-button
          type="primary"
          plain
          icon="el-icon-circle-plus-outline"
          style="position: relative; left: 20px;"
          @click="Visible = true"
        >增加分类</el-button>
        <el-button
          type="danger"
          plain
          icon="el-icon-delete"
          style="position: relative; left: 25px;"
          @click="delsort"
        >删除分类</el-button>
      </div>
      <br />
      <div class="left">博客内容：</div>
      <br />
      <div>
        <quill-editor v-model="editorHtml" ref="myQuillEditor" :options="editorOption"></quill-editor>
        <el-upload
          class="avatar-uploader"
          action="#"
          :http-request="Uploading"
          :show-file-list="false"
          :before-upload="beforeAvatarUpload"
          style="display: none"
        >
          <i class="el-icon-plus avatar-uploader-icon" id="imgInput"></i>
        </el-upload>
      </div>
    </div>
    <br />
    <br />
    <br />
    <div>
      <el-button type="primary" icon="el-icon-s-promotion" @click="addboke" v-show="btn">发布博客</el-button>
      <el-button type="primary" icon="el-icon-edit" @click="changeboke" v-show="!btn">修改并发布</el-button>
    </div>

    <el-dialog title="增加分类" :visible.sync="Visible" width="30%" :show-close="false">
      <el-form label-width="100px" class="demo-ruleForm">
        <el-form-item label="新分类">
          <el-input
            type="text"
            v-model="sortinput"
            style="width: 300px;"
            clearable
            @keydown.enter.native="addsort"
          ></el-input>
        </el-form-item>
        <el-button type="primary" @click="addsort" style="width: 400px;" plain size="small">确认增加</el-button>
        <br />
        <br />
        <el-button @click="Visible = false" style="width: 400px;" plain size="small">取消操作</el-button>
      </el-form>
    </el-dialog>
  </el-card>
</template>

<script>
import { quillEditor } from "vue-quill-editor";
import { quillRedefine } from "vue-quill-editor-upload";

export default {
  components: { quillEditor },
  data() {
    return {
      sortinput: "",
      Visible: false, //增加分类对话框
      editorHtml: "", //博客内容
      editorOption: {
        placeholder: "您想写点什么？"
      }, //富文本编辑器配置
      title: "新建博客",
      btn: true, //按钮是否展示
      blogtitle: "",
      sort: "", //博客的分类
      options: [] //分类的列表
    };
  },
  mounted() {
    var title = this.$route.params.title;
    if (title != undefined) {
      this.change();
    }
    this.loadingcategory();
    this.$refs.myQuillEditor.quill
      .getModule("toolbar")
      .addHandler("image", this.imgHandler);
  },
  methods: {
    beforeAvatarUpload(file) {  //上传前的检查
      const isJPG = ["image/jpeg", "image/png", "image/jpg"].includes(
        file.type
      );
      const isLt2M = file.size / 1024 / 1024 < 1;

      if (!isJPG) {
        this.$message.error("上传图片格式不符！!");
      }
      if (!isLt2M) {
        this.$message.error("上传头像图片大小不能超过 1MB!");
      }
      return isJPG && isLt2M;
    },
    imgHandler(state) {  //更换编辑器原来的上传图片方法
      this.addRange = this.$refs.myQuillEditor.quill.getSelection();
      if (state) {
        let fileInput = document.getElementById("imgInput");
        fileInput.click(); // 加一个触发事件
      }
    },
    Uploading(data) {  // 上传博客图片
      // console.log(data);
      var that = this;
      const name = window.sessionStorage.getItem("name");
      this.getBase64(data.file).then(resBase64 => {
        // this.fileBase64 = resBase64.split(",")[1]; //直接拿到base64信息
        // console.log(resBase64);
        this.$axios
          .post("./addBlogImgPost", {
            base64Data: resBase64
          })
          .then(function(res) {
            let value = res.data.src;
            // console.log(value);
            let quill = that.$refs.myQuillEditor.quill;
            let length = quill.getSelection().index;
            // 插入图片 response.data.url为服务器返回的图片地址
            quill.insertEmbed(length, "image", value);
            // 调整光标到最后
            quill.setSelection(length + 1);
          });
      });
    },
    getBase64(file) {  // 转base64编码
      return new Promise((resolve, reject) => {
        let reader = new FileReader();
        let fileResult = "";
        reader.readAsDataURL(file); //开始转
        reader.onload = function() {
          fileResult = reader.result;
        }; //转失败
        reader.onerror = function(error) {
          reject(error);
        }; //转结束咱就 resolve 出去
        reader.onloadend = function() {
          // console.log(fileResult);
          resolve(fileResult);
        };
      });
    },
    change() {  //预备改博客
      this.btn = false;
      var title = this.$route.params.title;
      var body = this.$route.params.body;
      var category = this.$route.params.category;
      // console.log(title,body);
      this.title = "修改博客";
      this.blogtitle = title;
      this.editorHtml = body;
      this.sort = category;
      window.sessionStorage.setItem("oldtitle", title);
    },
    changeboke() {  //确认改博客
      const user = window.sessionStorage.getItem("user");
      const pass = window.sessionStorage.getItem("pass");
      const oldtitle = window.sessionStorage.getItem("oldtitle");
      // console.log(oldtitle)
      var that = this;
      this.$axios
        .get("./ChangeBlog", {
          params: {
            blogtitle: that.blogtitle,
            blogbody: that.editorHtml,
            username: user,
            password: pass,
            oldtitle: oldtitle,
            category: that.sort
          }
        })
        .then(function(res) {
          if (res.data == "yes") {
            that.$message({
              message: "博客修改成功",
              type: "success"
            });
            that.$router.push("/myboke");
          } else {
            that.$message.error("博客修改失败");
          }
        });
    },
    loadingcategory() {  //加载所有分类
      const user = window.sessionStorage.getItem("user");
      const pass = window.sessionStorage.getItem("pass");
      var that = this;
      this.$axios
        .get("./searchAllCategory", {
          params: {
            username: user,
            password: pass
          }
        })
        .then(function(res) {
          // console.log("res",res.data);
          that.options = res.data;
        });
    },
    addsort() {  //增加分类
      if (this.sortinput != "") {
        const user = window.sessionStorage.getItem("user");
        const pass = window.sessionStorage.getItem("pass");
        var that = this;
        this.$axios
          .get("./addCategory", {
            params: {
              username: user,
              password: pass,
              category: that.sortinput
            }
          })
          .then(function(res) {
            that.$message({
              message: "添加新分类成功！！",
              type: "success"
            });
            that.Visible = false;
            that.loadingcategory();
            that.sortinput = "";
          });
      } else {
        this.$message.error("分类为空，添加失败！！");
      }
    },
    delsort() {  //删除分类
      if (this.sort != "") {
        this.$confirm("此操作将永久删除该分类, 是否继续?", "提示", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        })
          .then(() => {
            const user = window.sessionStorage.getItem("user");
            const pass = window.sessionStorage.getItem("pass");
            var that = this;
            this.$axios
              .get("./deleteCategory", {
                params: {
                  category: that.sort,
                  username: user,
                  password: pass
                }
              })
              .then(function(res) {
                // console.log(res)
                that.$message({
                  message: "删除新分类成功！！",
                  type: "success"
                });
                that.sort = "";
                that.loadingcategory();
              });
          })
          .catch(() => {
            this.$message({
              type: "info",
              message: "已取消删除"
            });
          });
      } else {
        this.$message.error("分类为空，删除失败！！");
      }
    },
    addboke() {  //增加博客
      if (this.blogtitle == "" || this.editorHtml == "" || this.sort == "") {
        this.$message({
          center: true,
          showClose: true,
          message: "内容没有填写完整",
          type: "warning"
        });
      } else {
        const name = window.sessionStorage.getItem("name");
        const user = window.sessionStorage.getItem("user");
        const pass = window.sessionStorage.getItem("pass");
        var that = this;
        // console.log(that.sort)
        this.$axios
          .get("./AddBlog", {
            params: {
              identity1: name,
              blogbody: that.editorHtml,
              blogtitle: that.blogtitle,
              username: user,
              password: pass,
              category: that.sort //博客分类
            }
          })
          .then(function(res) {
            // console.log(res);
            if (res.data == "yes") {
              that.$message({
                message: "博客发布成功",
                type: "success"
              });
              that.$router.push("/myboke");
            } else {
              that.$message.error("标题重复，发布失败");
            }
          });
      }
    },
    out() {
      //返回提示
      this.$confirm("内容不会保存, 是否返回?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          this.$router.go(-1);
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消"
          });
        });
    }
  }
};
</script>

<style scoped>
.title {
  font-size: 25px;
  color: #000;
}
.left {
  font-weight: bold;
  font-size: 15px;
  float: left;
}

.quill-editor {
  height: 300px;
}
</style>