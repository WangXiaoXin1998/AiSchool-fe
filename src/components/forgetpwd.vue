<template>
  <div id="control">
    <vue-particles
      color="#ccc"
      :particleOpacity="0.7"
      :particlesNumber="200"
      shapeType="circle"
      :particleSize="3"
      linesColor="#dedede"
      :linesWidth="2"
      :lineLinked="true"
      :lineOpacity="0.4"
      :linesDistance="150"
      :moveSpeed="2"
      :hoverEffect="true"
      hoverMode="grab"
      :clickEffect="true"
      clickMode="push"
      class="lizi"
    ></vue-particles>
    <div class="pwdform">
      <el-card class="box-card" shadow="always">
        <div slot="header" class="clearfix">
          <span>
            <b>AiSchool - 忘记密码</b>
          </span>
          <el-button style="float: right; padding: 3px 0" @click="relogin" type="text">返回登录</el-button>
        </div>
        <el-steps :active="step" align-center finish-status="success">
          <el-step title="填写信息"></el-step>
          <el-step title="验证密保"></el-step>
          <el-step title="设置密码"></el-step>
        </el-steps>
        <div class="form0" v-if="step==0">
          <el-form
            hide-required-asterisk
            :model="pwdform0"
            status-icon
            :rules="rules"
            ref="pwdform0"
            label-width="140px"
            class="demo-ruleForm"
          >
            <br />
            <br />
            <el-form-item label="用户名" size="medium" prop="username">
              <el-input v-model="pwdform0.username" placeholder="请输入用户名" class="input"></el-input>
            </el-form-item>
          </el-form>
        </div>

        <div class="form1" v-show="step==1">
          <el-form
            hide-required-asterisk
            :model="pwdform1"
            status-icon
            :rules="rules"
            ref="pwdform1"
            label-width="140px"
            class="demo-ruleForm"
          >
            <el-form-item label="密保一" size="medium" prop="answer1">
              <el-input v-model="pwdform1.answer1" :placeholder="pwdform1.question1" class="input"></el-input>
            </el-form-item>
            <el-form-item label="密保二" size="medium" prop="answer2">
              <el-input v-model="pwdform1.answer2" :placeholder="pwdform1.question2" class="input"></el-input>
            </el-form-item>
            <el-form-item label="密保三" size="medium" prop="answer3">
              <el-input v-model="pwdform1.answer3" :placeholder="pwdform1.question3" class="input"></el-input>
            </el-form-item>
          </el-form>
        </div>

        <div class="form2" v-show="step==2">
          <el-form
            hide-required-asterisk
            :model="pwdform2"
            status-icon
            :rules="rules"
            ref="pwdform2"
            label-width="140px"
            class="demo-ruleForm"
          >
            <el-form-item label="新密码" size="medium" prop="newpwd" autocomplete="off">
              <el-input
                type="password"
                v-model="pwdform2.newpwd"
                placeholder="请输入密码"
                class="input"
                show-password
              ></el-input>
            </el-form-item>
            <el-form-item label="再输一次" size="medium" prop="agapwd" autocomplete="off">
              <el-input
                type="password"
                v-model="pwdform2.agapwd"
                placeholder="请再次输入密码"
                class="input"
                show-password
              ></el-input>
            </el-form-item>
          </el-form>
        </div>

        <div class="form3" v-show="step==3">
          <br />密码重置成功
          <div class="des">
            3秒后跳转至
            <el-button type="text" @click="relogin()">登录</el-button>界面
          </div>
        </div>

        <div class="button" v-if="step<3">
          <el-button @click="resetForm('pwdform'+step)">重置</el-button>
          <el-button type="primary" @click="submit('pwdform'+step)">{{!(step>1)?'下一步':'确定'}}</el-button>
        </div>
      </el-card>
    </div>
  </div>
</template>

<script>
import Vue from "vue";
import qs from "qs";
import VueParticles from "vue-particles";
Vue.use(VueParticles);
Vue.use(qs);

export default {
  name: "forgetpwd",
  data() {
    var validatePass1 = (rule, value, callback) => {
      var reg = /^[0-9A-Za-z]{6,20}$/;
      if (!reg.test(value)) {
        callback(new Error("密码必须为6-20位的数字或字母"));
      } else {
        callback();
      }
    };
    var validatePass2 = (rule, value, callback) => {
      if (value === "") {
        callback(new Error("请再次输入密码"));
      } else if (value !== this.pwdform2.newpwd) {
        callback(new Error("两次输入密码不一致"));
      } else {
        callback();
      }
    };
    return {
      step: 0,
      pwdform0: {
        username: "",
        character: "用户"
      },
      pwdform1: {
        question1: "",
        question2: "",
        question3: "",
        answer1: "",
        answer2: "",
        answer3: ""
      },
      pwdform2: {
        token: "",
        newpwd: "",
        agapwd: ""
      },
      rules: {
        username: [
          { required: true, message: "请输入用户名", trigger: "blur" }
        ],
        character: [{ required: true, message: "请选择角色", trigger: "blur" }],
        answer1: [
          { required: true, message: "请输入问题答案", trigger: "blur" }
        ],
        answer2: [
          { required: true, message: "请输入问题答案", trigger: "blur" }
        ],
        answer3: [
          { required: true, message: "请输入问题答案", trigger: "blur" }
        ],
        newpwd: [{ required: true, validator: validatePass1, trigger: "blur" }],
        agapwd: [{ required: true, validator: validatePass2, trigger: "blur" }]
      }
    };
  },
  methods: {
    submit(formName) {
      this.$refs[formName].validate(valid => {
        if (valid) {
          if (this.step == 0) {
            var pwdform = {
              username: this.pwdform0.username
            };
            this.$axios
              .post(
                "/apife/api/forget_question_getQue",
                qs.stringify(pwdform),
                {}
              )
              .then(res => {
                if (res.data.error_num == 1) {
                  this.$message.error("验证失败：" + res.data.msg);
                  return;
                }
                this.pwdform1.question1 = res.data.question1;
                this.pwdform1.question2 = res.data.question2;
                this.pwdform1.question3 = res.data.question3;
                this.step = this.step + 1;
              })
              .catch(error => {
                this.$message.error("验证失败：服务器连接超时");
                return;
              });
          } else if (this.step == 1) {
            var pwdform = {
              username: this.pwdform0.username,
              answer1: this.pwdform1.answer1,
              answer2: this.pwdform1.answer2,
              answer3: this.pwdform1.answer3
            };
            this.$axios
              .post(
                "/apife/api/forget_question_checkAns",
                qs.stringify(pwdform),
                {}
              )
              .then(res => {
                if (res.data.error_num == 1) {
                  this.$message.error("验证失败：" + res.data.msg);
                  return;
                }
                this.pwdform2.token = res.data.token;
                this.step = this.step + 1;
              })
              .catch(error => {
                this.$message.error("验证失败：服务器连接超时");
                return;
              });
          } else if (this.step == 2) {
            var pwdform = {
              token: this.pwdform2.token,
              password: this.pwdform2.agapwd
            };
            this.$axios
              .post(
                "/apife/api/forget_question_resetPwd",
                qs.stringify(pwdform),
                {}
              )
              .then(res => {
                if (res.data.error_num == 1) {
                  this.$message.error("重置失败：" + res.data.msg);
                  return;
                }
                this.step = this.step + 1;
                setTimeout(() => {
                  this.relogin();
                }, 3000);
              })
              .catch(error => {
                this.$message.error("重置失败：服务器连接超时");
                return;
              });
          }
        }
      });
    },
    resetForm(formName) {
      this.$refs[formName].resetFields();
    },
    relogin() {
      this.$router.push("login");
    }
  }
};
</script>

<style scoped>
.background {
  z-index: -1;
}

.input {
  width: 230px;
}

.pwdform {
  position: fixed;
  top: 10%;
  left: 50%;
  width: 500px;
  margin-left: -300px;
}

.box-card {
  width: 500px;
  height: 400px;
}

.form0 {
  margin-top: 50px;
}

.form1 {
  margin-top: 25px;
}

.form2 {
  margin-top: 50px;
}

.form3 {
  margin-top: 30px;
  text-align: center;
  font-size: 20px;
}

.des {
  font-size: 14px;
}

.button {
  left: 175px;
  position: absolute;
  bottom: 20px;
}
</style>