<template>
  <div class="login">
    <div class="filter">
      <div class="login-left">
        <img src="./../assets/gather/login-img.png" alt="加载异常" class="login-img" />
        <div class="text">让你的劳动更有获得感</div>
      </div>
      <div class="login-right">
        <div class="header flex-c-c" v-if="loginType==1||loginType==2">
          <div
            class="header-left header-text"
            :style="loginType==2?'color:rgba(255,220,0,1);':''"
            @click="changeLoginType(2)"
          >短信登录</div>
          <div
            class="header-text"
            :style="loginType==1?'color:rgba(255,220,0,1);':''"
            @click="changeLoginType(1)"
          >账户登录</div>
        </div>
        <div v-if="loginType==3" class="zuce">商家注册账号</div>
        <div v-if="loginType==4" class="zuce">重置密码</div>

        <div class="main">
          <!--  扫码登录-->

          <div v-show="loginType!=0" style="padding:50px 60px 45px">
            <!-- 手机号 -->
            <div class="margin">
              <div class="flex-c-c input-box" :style="phoneTip?'border-color: red;':''">
                <label for="phone">
                  <img src="./../assets/gather/phone.png" alt="加载异常" class="img" />
                </label>

                <input
                  type="text"
                  id="phone"
                  placeholder="请输入手机号"
                  v-model="phone"
                  @input="inputPhone"
                  class="input"
                  maxlength="11"
                  @blur="blurPhone"
                  style="  background: transparent;"
                />
              </div>
              <p v-show="phoneTip" class="red">{{phoneTip}}</p>
            </div>
            <!-- 图片验证 -->
            <div class="margin" v-show="loginType>=2">
              <div class="flex-c-c img-box" :style="phoneTip?'border-color: red;':''">
                <input
                  type="text"
                  placeholder="验证码"
                  v-model="imgCode"
                  @blur="blurImgCode"
                  class="img-input"
                  @input="inputImgCode"
                />

                <img :src="imgSrc" alt="加载异常" @click="getImgCode" class="img-code" />
              </div>
              <p v-show="imgCodeTip" class="red">{{imgCodeTip}}</p>
            </div>
            <!-- 短信登录 -->
            <div class="margin" v-show="loginType>=2">
              <div class="note-code-box flex-c-c">
                <input
                  type="text"
                  :style="noteTip?'border-color: red;':''"
                  v-model="code"
                  class="code-input"
                  @input="inputCode"
                  @blur="blurCode"
                  placeholder="请输入短信验证码"
                />
                <button class="note-code" @click="getCode" ref="codeBtn" :disabled="limitCode">{{countDown}}</button>
              </div>
              <p v-show="noteTip" class="red">{{noteTip}}</p>
            </div>
            <!-- 账户密码 -->
            <div class="margin" v-show="loginType==1||loginType==3||loginType==4">
              <div class="flex-c-c input-box" :style="passwordTip?'border-color: red;':''">
                <label for="password">
                  <img src="./../assets/gather/password.png" alt="加载异常" class="img" />
                </label>
                <input
                  type="password"
                  id="password"
                  :placeholder="loginType==1?'请输入密码':'密码由数字或字母组合'"
                  v-model="password"
                  @input="inputPassword"
                  class="input"
                  @blur="blurPassword"
                  maxlength="20"
                  style="  background: transparent;"
                />
              </div>
              <p v-show="passwordTip" class="red">{{passwordTip}}</p>
            </div>
            <!--再次确认密码 -->
            <div class="margin" v-show="loginType>=3">
              <div class="flex-c-c input-box" :style="confirmTip?'border-color: red;':''">
                <label for="password">
                  <img src="./../assets/gather/password.png" alt="加载异常" class="img" />
                </label>
                <input
                  type="password"
                  id="confirmPassword"
                  placeholder="再次确认密码"
                  v-model="confirmPassword"
                  @input="inputConfirm"
                  class="input"
                  @blur="blurConfirm"
                  maxlength="20"
                />
              </div>
              <p v-show="confirmTip" class="red">{{confirmTip}}</p>
            </div>

            <div class="useTips">
              <button class="note-login" v-show="loginType==1" @click="changeLoginType(4)">忘记密码</button>
            </div>
            <div
              class="login-btn"
              v-show="loginType==1||loginType==2"
              @click="loginUser"
              ref="loginBtn"
            >登录</div>
            <div
              class="register-btn"
              v-show="loginType==3"
              @click="createUser"
              ref="registerBtn"
            >立即注册</div>
            <div class="register-btn" v-show="loginType==4" @click="resetPassword" ref="resetBtn">确定</div>
            <div class="foot flex-c-c">
              <button
                class="foot-text"
                v-show="loginType==1||loginType==2"
                @click="changeLoginType(3)"
              >创建新账户</button>
              <button class="foot-text" v-show="loginType==1||loginType==2">商家入驻</button>
              <!-- <button class="foot-text" v-show="loginType==1||loginType==2" @click="goHome">商家入驻</button> -->
              <button class="foot-login-text" v-show="loginType==3">
                已注册
                <span style="color:rgba(255,220,0,1);" @click="changeLoginType(1)">直接登录</span>
              </button>
              <button class="foot-login-text" v-show="loginType==4">
                已找回
                <span style="color:rgba(255,220,0,1);" @click="changeLoginType(1)">直接登录</span>
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import { mapActions, mapMutations } from "vuex";

import md5 from "js-md5";
import Cookies from "js-cookie";
import Vue from "vue";
import MyLoading from "./../utlis/loading.js";
let loadingInstance = new MyLoading();

export default {
  name: "login",
  data() {
    return {
      phone: "",
      password: "",
      phoneTip: "",
      passwordTip: "",
      loginType: 1, //logintype 1账号登录 2短信登录 3注册 4忘记密码
      confirmTip: "",
      confirmPassword: "",
      code: "",
      countDown: "获取验证码",
      times: 60,
      noteTip: "",
      is_query: false,
      timer: null,
      imgCode: "",
      imgCodeTip: "",
      imgSrc: "",
      imgSign: "",
      lock: true,
      limitCode:null
    };
  },

  beforeDestroy() {
    clearInterval(this.timer);
  },
  methods: {
    ...mapMutations(["changeToken", "changePhone"]),
    ...mapActions([
      "zuce",
      "forCode",
      "login",
      "queryImgCode",
      "shopDetail",
      "reset",
      "resetCode"
    ]),
    // goHome() {
    //   this.$router.push("/");
    // },
    changeLoginType(index) {
      let self = this;
      this.loginType = index;
      self.is_query = false;
      self.password = "";
      self.phoneTip = "";
      self.passwordTip = "";
      self.is_query = false;
      (this.confirmTip = ""),
        (this.confirmPassword = ""),
        clearInterval(this.timer);
      this.countDown = "获取验证码";
      this.code = "";
      this.noteTip = "";
      this.imgCode = "";
      this.imgCodeTip = "";
      this.limitCode = null;
      this.times = 60;
      this.lock = true
      if (this.loginType > 1) {
        this.getImgCode();
      }
    },
    inputImgCode() {},
    blurImgCode() {},
    getImgCode() {
      let self = this;
      let type = "";
      if (self.loginType == 3) {
        type = "register";
      } else if (self.loginType == 2) {
        type = "login";
      } else {
        type = "reset";
      }
      this.queryImgCode({
        type: type
      }).then(res => {
        self.imgSrc = res.data.data.url;
        self.imgSign = res.data.data.sign;
      });
    },
    getCode() {
      let self = this;
      if (!self.phone) {
        Vue.prototype.$message({
          message: "手机号不能为空！",
          type: "warning"
        });
        return false;
      }
      if (!/^1[3456789]\d{9}/.test(self.phone)) {
        Vue.prototype.$message({
          message: "手机号码格式不正确！",
          type: "warning"
        });
        return false;
      }

      if (!self.imgCode) {
        Vue.prototype.$message({
          message: "图片验证码",
          type: "warning"
        });
        return false;
      }

      if (self.is_query) {
        return;
      }
      self.is_query = true;
      loadingInstance.open(self.$refs.codeBtn);
      let type = "";
      if (self.loginType == 3) {
        type = "register";
      } else if (self.loginType == 2) {
        type = "login";
      }
      // else {
      //   type = "reset";
      // }

      if (self.loginType != 4) {
        self
          .forCode({
            phone: self.phone,
            type: type,
            code: self.imgCode,
            sign: self.imgSign
          })
          .then(res => {
            let d = res.data;
            loadingInstance.close();
            self.is_query = false;
            if (d.errcode == 0) {
              if (this.lock) {
                self.makeTimer();
              }
              Vue.prototype.$message({ message: d.errmsg, type: "success" });
            } else {
              self.is_query = false;
              Vue.prototype.$message({ message: d.errmsg, type: "warning" });
            }
          })
          .catch(err => {
            loadingInstance.close();
            self.is_query = false;
            Vue.prototype.$message({ message: err.errmsg, type: "warning" });
          });
      } else {
        self
          .resetCode({
            phone: self.phone,
            type: "reset",
            code: self.imgCode,
            sign: self.imgSign
          })
          .then(res => {
            let d = res.data;
            loadingInstance.close();
            self.is_query = false;
            if (d.errcode == 0) {
              if (this.lock) {
                self.makeTimer();
              }
              Vue.prototype.$message({ message: d.errmsg, type: "success" });
            } else {
              self.is_query = false;
              Vue.prototype.$message({ message: d.errmsg, type: "warning" });
            }
          })
          .catch(err => {
            loadingInstance.close();
            self.is_query = false;
            Vue.prototype.$message({ message: err.errmsg, type: "warning" });
          });
      }
    },
    loginUser() {
      let self = this;
      if (!self.phone) {
        Vue.prototype.$message({
          message: "手机号不能为空！",
          type: "warning"
        });
        return false;
      }

      if (!/^1[3456789]\d{9}/.test(self.phone)) {
        Vue.prototype.$message({
          message: "手机号码格式不正确！",
          type: "warning"
        });
        return false;
      }
      if (self.loginType == 1) {
        if (!self.password) {
          Vue.prototype.$message({
            message: "密码不能为空！",
            type: "warning"
          });
          return false;
        }
        if (self.passwordTip != 0) {
          Vue.prototype.$message({
            message: self.passwordTip,
            type: "warning"
          });
          return false;
        }
      } else if (self.loginType == 2) {
        if (!self.imgCode) {
          Vue.prototype.$message({
            message: "图片验证码",
            type: "warning"
          });
          return false;
        }
        if (!self.code) {
          Vue.prototype.$message({
            message: "请输入手机验证码",
            type: "warning"
          });
          return false;
        }
      }
      if (self.is_query) {
        return;
      }

      self.is_query = true;
      loadingInstance.open(self.$refs.loginBtn);
      Cookies.set("moiToken", "", { expires: 60 });
      Cookies.set("phone", self.phone, { expires: 60 });

      self
        .login({
          phone: self.phone,
          password: md5(self.password),
          scenario: self.loginType == 1 ? "login_phone" : "login_code",
          code: self.code
        })
        .then(res => {
          let d = res.data;

          loadingInstance.close();
          self.is_query = false;
          if (d.errcode == 0) {
            Cookies.set("moiToken", d.data.token.token, { expires: 60 });
            Cookies.set("phone", d.data.phone, { expires: 60 });
            self.password = "";
            Vue.prototype.$message({ message: "登录成功~", type: "success" });

            self.changeToken(d.data.token.token);
            /********************判断到哪一步************ */

            this.changePhone(d.data.phone);

            this.shopDetail();

            /********************************************/
          } else {
            Vue.prototype.$message({ message: d.errmsg, type: "warning" });
          }
        })
        .catch(err => {
          loadingInstance.close();
          self.is_query = false;
          Vue.prototype.$message({ message: err.errmsg, type: "warning" });
        });
    },
    makeTimer() {
      let self = this;
      self.times = 60;
      self.countDown = "已发送(" + 60 + "s)";
      if (this.lock) {
        this.lock = false;
        this.limitCode = "disabled"
        self.timer = setInterval(() => {
          if (self.times > 0) {
            self.countDown = "已发送(" + (self.times - 1) + "s)";
            self.times = self.times - 1;
          } else {
            clearInterval(self.timer);
            self.countDown = "获取验证码";
            self.times = 60;
            this.lock = true;
            this.limitCode = null
          }
        }, 1000);
      }
    },
    resetPassword() {
      let self = this;
      if (!self.phone) {
        Vue.prototype.$message({
          message: "手机号不能为空！",
          type: "warning"
        });
        return false;
      }
      if (!/^1[3456789]\d{9}/.test(self.phone)) {
        Vue.prototype.$message({
          message: "手机号码格式不正确",
          type: "warning"
        });
        return false;
      }
      if (!self.password) {
        Vue.prototype.$message({
          message: "密码不能为空！",
          type: "warning"
        });
        return false;
      }
      if (self.passwordTip != 0) {
        Vue.prototype.$message({ message: self.passwordTip, type: "warning" });
        return false;
      }
      if (!self.confirmPassword) {
        Vue.prototype.$message({ message: "请再次输入密码", type: "warning" });
        return false;
      }
      if (self.password != self.confirmPassword) {
        Vue.prototype.$message({ message: "密码输入不一致", type: "warning" });
        return false;
      }
      if (!self.code) {
        Vue.prototype.$message({
          message: "请输入手机验证码",
          type: "warning"
        });
        return false;
      }
      if (self.is_query) {
        return;
      }
      self.is_query = true;
      loadingInstance.open(self.$refs.resetBtn);
      this.reset({
        phone: self.phone,
        code: self.code,
        password: md5(self.password),
        scenario: "reset",
        graphCode: self.imgCode
      })
        .then(res => {
          loadingInstance.close();

          if (res.data.errcode == 0) {
            Vue.prototype.$message({
              message: "修改成功，请登录",
              type: "success"
            });
            clearInterval(self.timer);

            self.code = "";
            self.is_query = false;

            self.countDown = "获取验证码";
            self.password = "";
            self.confirmPassword = "";
            self.times = 60;
          } else {
            self.is_query = false;
            Vue.prototype.$message({
              message: res.data.errmsg,
              type: "warning"
            });
          }
        })
        .catch(err => {
          loadingInstance.close();
          self.is_query = false;
          Vue.prototype.$message({ message: err.errmsg, type: "warning" });
        });
    },
    createUser() {
      let self = this;
      if (!self.phone) {
        Vue.prototype.$message({
          message: "手机号不能为空！",
          type: "warning"
        });
        return false;
      }
      if (!/^1[3456789]\d{9}/.test(self.phone)) {
        Vue.prototype.$message({
          message: "手机号码格式不正确",
          type: "warning"
        });
        return false;
      }
      if (!self.password) {
        Vue.prototype.$message({
          message: "密码不能为空！",
          type: "warning"
        });
        return false;
      }
      if (self.passwordTip != 0) {
        Vue.prototype.$message({ message: self.passwordTip, type: "warning" });
        return false;
      }
      if (!self.confirmPassword) {
        Vue.prototype.$message({ message: "请再次输入密码", type: "warning" });
        return false;
      }
      if (self.password != self.confirmPassword) {
        Vue.prototype.$message({ message: "密码输入不一致", type: "warning" });
        return false;
      }
      if (!self.code) {
        Vue.prototype.$message({
          message: "请输入手机验证码",
          type: "warning"
        });
        return false;
      }
      if (self.is_query) {
        return;
      }
      self.is_query = true;
      loadingInstance.open(self.$refs.registerBtn);

      this.zuce({
        phone: self.phone,
        code: self.code,
        password: md5(self.password),
        scenario: "register_phone"
      })
        .then(res => {
          loadingInstance.close();

          if (res.data.errcode == 0) {
            Vue.prototype.$message({
              message: "注册成功，请登录",
              type: "success"
            });
            clearInterval(self.timer);

            self.code = "";
            self.is_query = false;

            self.countDown = "获取验证码";
            self.password = "";
            self.confirmPassword = "";
            self.times = 60;
          } else {
            self.is_query = false;
            Vue.prototype.$message({
              message: res.data.errmsg,
              type: "warning"
            });
          }
        })
        .catch(err => {
          loadingInstance.close();

          self.is_query = false;
          Vue.prototype.$message({ message: err.errmsg, type: "warning" });
        });
    },
    inputPhone() {
      if (this.phone.trim()) {
        let strArr = /^1\d*/i.exec(this.phone.trim());

        if (!strArr) {
          this.phone = "";
        } else {
          if (this.phoneTip != "") {
            this.phoneTip = "";
          }

          this.phone = strArr[0];
        }
      } else {
        this.phone = "";
      }
    },
    inputPassword() {
      if (this.password.trim()) {
        let reg = /^[A-Za-z0-9]+/i;

        let bool = reg.exec(this.password.trim());
        if (!bool) {
          this.password = "";
        } else {
          this.password = bool[0];
        }

        if (this.passwordTip != "") {
          this.passwordTip = "";
        }
      } else {
        this.password = "";
      }
    },
    inputConfirm() {
      if (this.confirmTip) {
        this.confirmTip = "";
      }
    },
    inputCode() {
      if (this.noteTip) {
        this.noteTip = "";
      }
    },
    blurCode() {
      if (!this.code.trim()) {
        this.noteTip = "验证码不能为空";
      }
    },
    blurConfirm() {
      if (!this.confirmPassword.trim()) {
        this.confirmTip = "确认密码不能为空！";
        return;
      }
      if (this.confirmPassword != this.password) {
        this.confirmTip = "两次密码输入不一致";
        return;
      }
    },
    blurPhone() {
      if (!this.phone.trim()) {
        this.phoneTip = "手机号码不能为空";
      } else {
        let bool = /^1[3456789]\d{9}/.test(this.phone.trim());

        if (!bool) {
          this.phoneTip = "手机号码格式不正确！";
        }
      }
    },
    blurPassword() {
      if (!this.password.trim()) {
        this.passwordTip = "密码不能为空！";
      } else if (
        this.password.trim().length < 6 ||
        this.password.trim().length > 20
      ) {
        this.passwordTip = "密码长度应该在6-20之间";
      } else {
        let reg = /^[A-Za-z0-9]{6,20}$/i;
        if (!reg.test(this.password.trim())) {
          this.passwordTip = "密码格式不对！";
        }
      }
    }
  }
};
</script>
<style>
.login input:focus {
  outline: none;
}
.login .el-loading-spinner i {
  color: rgb(255, 255, 255);
}
</style>
<style scoped>
.login-img {
  width: 342px;
  margin: 0 auto;
}
.filter {
  background: rgba(0, 0, 0, 0.2);
  min-width: 1260px;
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-grow: 0;
}
.zuce {
  height: 70px;
  font-size: 26px;
  /* padding-top: 18px; */
  border-bottom: 1px solid rgba(0, 0, 0, 0.7);
  padding-left: 60px;
  font-weight: 500;
  line-height: 70px;
  color: rgba(0, 0, 0, 0.7);
}
.img-input {
  flex-grow: 1;
  height: 45px;
  border-radius: 8px;
  border: 1px solid rgba(0, 0, 0, 0.15);
  padding-left: 20px;
}
.img-code {
  height: 45px;
  border: 2px solid rgba(0, 0, 0, 0.15);
  border-radius: 8px;
  margin: 0 0 0 20px;
}
.register-btn {
  height: 50px;
  background: rgba(255, 220, 0, 1);
  border-radius: 6px;
  margin: 0 auto;
  font-size: 26px;
  font-weight: 500;
  line-height: 50px;
  text-align: center;
  color: rgba(255, 255, 255, 1);
  overflow: hidden;
}
.scan-tip-img {
  width: 18px;
  margin-right: 2px;
}
.scan-tip {
  text-align: center;
  font-size: 16px;
  font-weight: 400;
  line-height: 24px;
  color: rgba(0, 0, 0, 1);
  margin: 15px 0 20px;
}
.scan-img {
  width: 250px;
  height: 250px;
}

.scan-code {
  margin: 40px auto;
  width: 252px;
  height: 252px;
  border: 1px solid rgba(0, 0, 0, 0.15);
}
.scan-box {
  border-bottom: 1px dashed rgba(0, 0, 0, 1);
}
.note-login {
  cursor: pointer;
  border: none;
  background-color: transparent;
  font-size: 20px;
}
.foot-login-text {
  border: none;
  margin: 0 10px;
  padding-bottom: 3px;
  border-left: none;
  border-right: none;
  border-top: none;
  background-color: transparent;
  font-size: 20px;
  cursor: pointer;
  font-weight: 400;
  line-height: 24px;
}
.foot-login-text span {
  border-bottom: 1px solid rgba(0, 0, 0, 0.3);
}
.foot-text {
  border-bottom: 1px solid rgba(0, 0, 0, 0.3);
  margin: 0 10px;
  padding-bottom: 3px;
  border-left: none;
  border-right: none;
  border-top: none;
  background-color: transparent;
  font-size: 20px;
  cursor: pointer;
  font-weight: 400;
  line-height: 24px;
}
.scan-foot {
  margin: 20px 0 20px;
  font-size: 20px;
  font-weight: 400;
  line-height: 24px;
  color: rgba(0, 0, 0, 1);
  min-width: 1260px;
}
.foot {
  margin-top: 50px;
  font-size: 20px;
  font-weight: 400;
  line-height: 24px;
  color: rgba(0, 0, 0, 1);
}
.login-btn {
  height: 50px;
  background: rgba(255, 220, 0, 1);
  border-radius: 6px;
  margin: 30px auto;
  font-size: 26px;
  font-weight: 500;
  line-height: 50px;
  text-align: center;
  color: rgba(255, 255, 255, 1);
  overflow: hidden;
}
.note-code {
  width: 180px;
  height: 45px;
  border: none;
  overflow: hidden;
  background: rgba(0, 0, 0, 0.2);
  border-radius: 6px;
  font-size: 14px;
  font-weight: 400;
  line-height: 45px;
  color: rgba(0, 0, 0, 0.7);
  margin-left: 20px;
}
.code-input {
  width: 280px;
  height: 45px;
  border: 1px solid rgba(0, 0, 0, 0.15);
  opacity: 0.9;
  border-radius: 6px;
  flex-shrink: 1;
  text-align: center;
}
.note-code-box {
  height: 45px;
  opacity: 0.9;
}
.input-box {
  height: 45px;
  border: 1px solid rgba(0, 0, 0, 0.15);
  opacity: 0.9;
  border-radius: 6px;
}
.useTips {
  font-size: 20px;
  font-weight: 400;
  line-height: 24px;
  color: rgba(0, 0, 0, 0.9);
}
.margin {
  height: 85px;
}
.red {
  font-size: 16px;
  font-weight: 400;
  line-height: 24px;
  color: rgba(236, 44, 44, 1);
  margin-top: 5px;
}
.img {
  height: 24px;
  margin: 0 3px 0 9px;
}
.input {
  flex-grow: 1;
  font-size: 18px;
  font-weight: 400;
  line-height: 24px;
  color: rgba(0, 0, 0, 0.7);
  border: none;
}
.header {
  padding: 18px 0 17px;
  border-bottom: 1px solid rgba(0, 0, 0, 0.7);
  flex-grow: 1;
  text-align: center;
  font-size: 26px;
  font-weight: 500;
  line-height: 35px;
  color: rgba(0, 0, 0, 0.7);
}

.header-left {
  border-right: 1px solid rgba(0, 0, 0, 0.5);
}
.header-text {
  flex-grow: 1;
}
.login {
  min-height: 100vh;
  background: url("./../assets/gather/login-bg.png");
  background-size: cover;
  background-origin: 50% 50%;
  overflow-y: auto;
}
.login-left {
  display: inline-block;
  vertical-align: middle;
  text-align: center;
  color: rgba(255, 255, 255, 1);
  flex-grow: 0;
  margin-right: 15.31%;
}
.text {
  font-size: 50px;
  margin-top: 40px;
  font-weight: 500;
}
.title {
  font-size: 120px;
  font-weight: 400;
}
.login-right {
  vertical-align: middle;
  display: inline-block;
  background: rgba(255, 255, 255, 1);
  border-radius: 20px;
  width: 500px;
}
</style>