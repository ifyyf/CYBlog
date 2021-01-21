<template>
  <div>
    <el-card>
      <div style="float:left;">
        <div style="display: inline-block;" @mouseenter="enter" @mouseleave="leave">
          <el-upload
            class="avatar-uploader"
            action="#"
            :show-file-list="false"
            :http-request="Uploading"
            :before-upload="beforeAvatarUpload"
          >
            <div class="img_container">
            <img v-if="imageUrl" :src="imageUrl" class="avatar" />
            <div class="mask" v-if="changeshow">更换头像</div>
            </div>
          </el-upload>
        </div>
        <div class="title">
          <div style="font-weight: bold; font-size: 20px;">{{usermessg.identity1}}</div>
          <br />
          <br />
          <div style="color:#909399;">{{usermessg.time1}}注册</div>
        </div>
        <ul>
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
      </div>
    </el-card>

    <div router style="width: 800px; display: inline-block; float: left;">
      <router-view></router-view>
    </div>

    <div style="width: 460px; display: inline-block; float: right;">
      <el-card>
        <el-form>
          <div>
            <el-input v-model="usermessg.sex" :disabled="disabled">
              <template slot="prepend">性别：</template>
            </el-input>
          </div>
          <br />
          <div>
            <el-input v-model="usermessg.uname" :disabled="disabled">
              <template slot="prepend">账号：</template>
            </el-input>
          </div>
          <br />
          <div>
            <el-input v-model="usermessg.pwd" :disabled="disabled">
              <template slot="prepend">密码：</template>
            </el-input>
          </div>
          <br />
          <div>
            <el-input v-model="usermessg.address" :disabled="disabled" placeholder="没有地址信息" maxlength="10"
              show-word-limit>
              <template slot="prepend">地址：</template>
            </el-input>
          </div>
          <br />
          <div>
            <el-input
              type="textarea"
              :autosize="{ minRows: 4, maxRows: 4}"
              :disabled="disabled"
              v-model="usermessg.personShow"
              maxlength="80"
              show-word-limit
              placeholder="这个人很懒，没写个人简介"
            ></el-input>
          </div>
          <br />
          <el-button type="primary" @click="change" plain v-show="btnshowing">修改信息</el-button>

          <el-button type="danger" @click="change" plain v-show="!btnshowing">放弃修改</el-button>
          <el-button type="primary" @click="upchange" plain v-show="!btnshowing">确认修改</el-button>
        </el-form>
      </el-card>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      imageUrl: "", //头像地址
      disabled: true,
      changeshow: false,  //换头像遮罩
      btnshowing: true,
      blognum: "", //博客量
      bloghot: "", //人气值
      usermessg: {}, //用户信息表
      identity1: "",    //用户姓名
      commentnum:"",  //总评论数
      fans:"",    //粉丝数
      concern:"",   //关注数
    };
  },
  mounted() {
    this.loading();
  },
  methods: {
    loading() { //加载我的信息
      var that = this;
      const name = window.sessionStorage.getItem("name");
      const user = window.sessionStorage.getItem("user");
      const pass = window.sessionStorage.getItem("pass");
      this.identity1 = name;
      this.$axios
        .get("./testUserData", {
          params: {
            username: user,
            password: pass,
            identity1: name
          }
        })
        .then(function(res) {
          // console.log(res.data);
          that.usermessg = res.data.user;
          that.bloghot = res.data.popularity1;
          that.blognum = res.data.number;
          that.commentnum = res.data.commentCount[0].commentCount;
        });
      this.$axios
        .get("./searchImg?identity1=" + name)
        .then(function(res) {
          // console.log(res.data);
          that.imageUrl = './' + res.data;
        });
      this.$axios.get("./searchCount",{
        params:{
          username: user,
          password: pass,
          identity1: name
        }
      }).then(function(res){
        // console.log(res.data);
        that.fans = res.data.fansCount[0].fansCount;
        that.concern = res.data.followCount[0].followCount;
      })
    },
    change() {  //预备修改信息
      this.btnshowing = !this.btnshowing;
      this.disabled = !this.disabled;
      this.loading();
    },
    upchange() {  //确认修改信息
      if (this.usermessg.sex == '' || this.usermessg.uname == '' || this.usermessg.pwd == '' || this.usermessg.address == '' ||
					this.usermessg.personShow == '') {
					  this.$message({
              message: "请填写完整",
              type: "warning"
            });
				} else if (!/^[\u4e00-\u9fa5]{1,3}$/.test(this.usermessg.sex)) {
        this.$message({
          message: "性别只能填中文",
          type: "warning"
        });
      } else if (!/^[0-9A-Za-z]{4,23}$/.test(this.usermessg.uname)) {
        this.$message({
          message: "账号不少于4位",
          type: "warning"
        });
      } else if (
        !/^(?![0-9]+$)(?![a-zA-Z]+$)[0-9A-Za-z]{4,23}$/.test(this.usermessg.pwd)
      ) {
        this.$message({
          message: "密码只能是字母加数字，且不少于4位",
          type: "warning"
        });
      } else {
        this.$confirm("此操作将修改您的信息, 是否继续?", "提示", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        })
          .then(() => {
            const user = window.sessionStorage.getItem("user");
            const pass = window.sessionStorage.getItem("pass");
            var that = this;
            this.$axios
              .get("./testChange", {
                params: {
                  identity1: that.usermessg.identity1,
                  sex: that.usermessg.sex,
                  username: that.usermessg.uname,
                  password: that.usermessg.pwd,
                  uname: user,
                  pwd: pass,
                  address:that.usermessg.address,
                  personShow:that.usermessg.personShow,
                }
              })
              .then(function(res) {
                // console.log(res);
                if (res.data == "yes") {
                  that.$message({
                    type: "success",
                    message: "修改信息成功"
                  });
                  window.sessionStorage.setItem("user", that.usermessg.uname); //存储修改状态
                  window.sessionStorage.setItem("pass", that.usermessg.pwd);
                  that.btnshowing = !that.btnshowing;
                  that.disabled = !that.disabled;
                  that.loading();
                } else {
                  that.$message.error("修改失败！！");
                }
              });
          })
          .catch(() => {
            that.$message({
              type: "info",
              message: "已取消修改"
            });
          });
      }
    },
    beforeAvatarUpload(file) {  //上传前的检查
      const isJPG = ["image/jpeg", "image/png", "image/jpg","image/gif"].includes(
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
    Uploading(data) {  // 上传头像
      var that = this;
      const name = window.sessionStorage.getItem("name");
      this.getBase64(data.file).then(resBase64 => {
        // this.fileBase64 = resBase64.split(",")[1]; //直接拿到base64信息
        // console.log(resBase64);
        this.$axios
          .post("./testBase", {
            base64Data: resBase64,
            identity1: name
          })
          .then(function(res) {
            // console.log(res);
            that.$message({
              message: res.data,
              type: "success"
            });
            that.loading();
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
    enter(){    //鼠标移入
      this.changeshow = true;
    },
    leave(){    //鼠标移出
      this.changeshow = false;
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
.avatar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}
/* .avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 100px;
  height: 100px;
  line-height: 100px;
  text-align: center;
} */
.img_container{   /*外部遮罩*/
  position: relative;
  padding: 0;
  width: 100px;
  height: 100px;
}
.mask{
  position: absolute;
  top: 0;
  left: 0;
  width: 100px;
  height: 100px;
  background: rgba(0,0,0,0.7);
  color: #f5f1e5;
  line-height: 100px;
  text-align: center;
}
.avatar {
  width: 100px;
  height: 100px;
  display: inline-block;
}
</style>