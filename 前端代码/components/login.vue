<template>
  <div style="text-align: center; margin: auto; font-size: 20px; height: 300px;">
    <div class="reg">
      <div style="font-size: 18; position: relative; top: 10px; color:#fff;">用户登录</div>
      <br />
      <el-form label-width="100px" class="demo-ruleForm">
        <el-form-item label="账号" class="fc">
          <el-input type="text" v-model="user" placeholder="请输入账号" style="width: 300px;" clearable></el-input>
        </el-form-item>
        <el-form-item label="密码" class="fc">
          <el-input
            type="password"
            v-model="pass"
            placeholder="请输入密码"
            style="width: 300px;"
            show-password
          ></el-input>
        </el-form-item>
        <el-button @click="visible = true" style="width: 150px;" size="small">注册</el-button>
        <el-button type="primary" @click="reg" style="width: 150px;" plain size="small">登录</el-button>
        <br />
        <el-link
          style="color:#fff; float:right; position: relative; right: 30px; top:6px;"
          @click="forgetVisible = true"
        >忘记密码</el-link>
      </el-form>
    </div>

    <el-dialog title="忘记密码" :visible.sync="forgetVisible" :show-close="false" width="33%">
      <el-form label-width="100px" class="demo-ruleForm">
        <el-form-item label="原账号">
          <el-input type="text" v-model="user1" placeholder="不少于4位" style="width: 300px;" clearable></el-input>
        </el-form-item>
        <el-button type="primary" @click="findpass" style="width: 400px;" plain size="small">找回密码</el-button>
        <br />
        <br />
        <el-button @click="forgetVisible = false" style="width: 400px;" plain size="small">取消操作</el-button>
      </el-form>
    </el-dialog>

    <el-dialog
      :visible.sync="visible"
      top="50px"
      :show-close="false"
      style="width: 1000px; margin: auto; font-size: 15px;"
    >
      <div style="font-size: 18px;">用户注册</div>
      <br />
      <el-form label-width="100px" class="demo-ruleForm">
        <el-form-item label="姓名">
          <el-input
            type="text"
            v-model="nameadd"
            placeholder="只能是中文"
            style="width: 300px;"
            size="small"
            clearable
          ></el-input>
        </el-form-item>
        <el-form-item label="性别">
          <el-input
            type="text"
            v-model="sexadd"
            placeholder="只能是中文"
            style="width: 300px;"
            size="small"
            clearable
          ></el-input>
        </el-form-item>
        <el-form-item label="账号">
          <el-input
            type="text"
            v-model="useradd"
            placeholder="不少于4位"
            style="width: 300px;"
            size="small"
            clearable
          ></el-input>
        </el-form-item>
        <el-form-item label="密码">
          <el-input
            type="password"
            v-model="passadd"
            placeholder="只能字母加数字，不少于4位"
            style="width: 300px;"
            size="small"
            show-password
          ></el-input>
        </el-form-item>
        <el-form-item label="邮箱">
          <el-input
            type="text"
            v-model="emailadd"
            placeholder="请正确填写"
            style="width: 300px;"
            size="small"
            clearable
          ></el-input>
        </el-form-item>
        <el-button @click="visible = false" style="width: 150px;" size="small">取消注册</el-button>
        <el-button type="success" @click="add" style="width: 150px;" plain size="small">确认注册</el-button>
        <br />
        <br />
        <div class="order">
          <span class="line"></span>
          <span class="txt">第三方登录</span>
          <span class="line"></span>
        </div>
        <div style="font-size: 15px;">暂不支持</div>
      </el-form>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      user1: "", //忘记密码是输入账号
      user: "", //登录账号
      pass: "", //登录密码
      nameadd: "", //注册姓名
      sexadd: "", //注册性别
      useradd: "", //注册账号
      passadd: "", //注册密码
      emailadd: "", //注册邮箱
      forgetVisible: false, //忘记密码
      visible: false //注册模态框默认关闭
    };
  },
  methods: {
    reg() {
      //登录
      if (!/^[0-9A-Za-z]{4,23}$/.test(this.user)) {
        this.$message({
          message: "账号不少于4位",
          type: "warning"
        });
      } else if (
        !/^(?![0-9]+$)(?![a-zA-Z]+$)[0-9A-Za-z]{4,23}$/.test(this.pass)
      ) {
        this.$message({
          message: "密码只能是字母加数字，且不少于4位",
          type: "warning"
        });
      } else {
        var that = this;
        this.$axios
          .get("./testLogin", {
            params: {
              username: that.user,
              password: that.pass
            }
          })
          .then(function(res) {
            // console.log(res);
            if (res.data != "") {
              that.$message({
                message: res.data.identity1 + "，登录成功，欢迎您",
                type: "success",
                center: true
              }); //成功
              window.sessionStorage.setItem("name", res.data.identity1); //存储登录状态
              window.sessionStorage.setItem("user", res.data.uname);
              window.sessionStorage.setItem("pass", res.data.pwd);
              that.$router.push("/newboke"); //使用路由跳转到主页面
            } else {
              that.$message.error("登录失败，请检查账号、密码,并确认激活");
            }
          });
      }
    },
    add() {
      //注册
      if (
        !this.nameadd ||
        !this.sexadd ||
        !this.useradd ||
        !this.passadd ||
        !this.emailadd
      ) {
        this.$message({
          message: "请填写完整",
          type: "warning"
        });
      } else if (!/^[\u4e00-\u9fa5]{1,4}$/.test(this.nameadd)) {
        this.$message({
          message: "姓名只能填中文，且不多于4个字",
          type: "warning"
        });
      } else if (!/^[\u4e00-\u9fa5]{1,3}$/.test(this.sexadd)) {
        this.$message({
          message: "性别只能填中文",
          type: "warning"
        });
      } else if (!/^[0-9A-Za-z]{4,23}$/.test(this.useradd)) {
        this.$message({
          message: "账号不少于4位",
          type: "warning"
        });
      } else if (
        !/^(?![0-9]+$)(?![a-zA-Z]+$)[0-9A-Za-z]{4,23}$/.test(this.passadd)
      ) {
        this.$message({
          message: "密码只能是字母加数字，且不少于4位",
          type: "warning"
        });
      } else if (
        !/^[a-zA-Z0-9_-]+@[a-zA-Z0-9_-]+(\.[a-zA-Z0-9_-]+)+$/.test(
          this.emailadd
        )
      ) {
        this.$message({
          message: "请正确填写邮箱",
          type: "warning"
        });
      } else {
        var that = this;
        this.$axios
          .get("./testRequestParam", {
            params: {
              identity1: that.nameadd,
              sex: that.sexadd,
              uname: that.useradd,
              pwd: that.passadd,
              email: that.emailadd
            }
          })
          .then(function(res) {
            if (res.data == "yes") {
              // console.log(res);
              that.$message({
                message: "注册成功,请到邮箱激活账号",
                type: "success",
                center: true
              }); //成功
              that.visible = false; //关闭模态框
            } else {
              that.$message.error("姓名、账号或邮箱重复，注册失败！！");
            }
          });
      }
    },
    findpass() {
      if (!/^[0-9A-Za-z]{4,23}$/.test(this.user1)) {
        this.$message({
          message: "原账号不少于4位",
          type: "warning"
        });
      } else {
        var that = this;
        this.$axios
          .get(
            "./forgetPassWord?username=" +
              that.user1
          )
          .then(function(res) {
            // console.log(res);
            if(res.data == "yes"){
              that.$message({
                message: "新密码已经发送至邮箱，登录后请修改密码",
                type: "success"
              });
              that.forgetVisible = false;
            } else {
              that.$message({
                message: "请核对原账号是否正确",
                type: "warning"
              });
            }
          });
      }
    },
  }
};
</script>

<style scoped>
.reg {
  width: 470px;
  text-align: center;
  margin: auto;
  position: relative;
  top: 100px;
  height: 250px;
  background-color: #545c64;
  border-radius: 10px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.12), 0 0 6px rgba(0, 0, 0, 0.04);
}

.fc > :first-child {
  color: #fff;
}
.order {
  line-height: 40px;
  text-align: center;
}

.order .line {
  display: inline-block;
  width: 150px;
  border-top: 1px solid black;
}

.order .txt {
  color: black;
  font-size: 15px;
  vertical-align: -4px;
}
</style>
