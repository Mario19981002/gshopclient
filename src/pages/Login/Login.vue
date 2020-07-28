<template>
  <section class="loginContainer">
    <div class="loginInner">
      <div class="login_header">
        <h2 class="login_logo">饿了吗</h2>
        <div class="login_header_title">
          <a href="javascript:;" :class="{on: loginWay}" @click="loginWay=true">注册</a>
          <a href="javascript:;" :class="{on: !loginWay}" @click="loginWay=false">登录</a>
        </div>
      </div>
      <div class="login_content">
        <form>
          <div :class="{on: loginWay}">
            <section class="login_message">
              <input type="text" placeholder="用户名" v-model="username" />
            </section>
            <section class="login_verification">
              <input type="text" placeholder="密码" v-model="password" v-if="showPwd" />
              <input type="password" placeholder="密码" v-else v-model="password" />
              <div class="switch_button" :class="showPwd?'on':'off'" @click="showPwd=!showPwd">
                <div class="switch_circle" :class="{right: showPwd}"></div>
                <span class="switch_text">{{showPwd ? 'abc' : '...'}}</span>
              </div>
            </section>
            <section class="login_verification">
              <input type="text" placeholder="确认密码" v-model="repassword" v-if="showPwd" />
              <input type="password" placeholder="确认密码" v-else v-model="repassword" />
              <div class="switch_button" :class="showPwd?'on':'off'" @click="showPwd=!showPwd">
                <div class="switch_circle" :class="{right: showPwd}"></div>
                <span class="switch_text">{{showPwd ? 'abc' : '...'}}</span>
              </div>
            </section>
          </div>
          <div :class="{on: !loginWay}">
            <section>
              <section class="login_message">
                <input type="text" maxlength="11" placeholder="用户名" v-model="name" />
              </section>
              <section class="login_verification">
                <input type="text" maxlength="8" placeholder="密码" v-if="showPwd" v-model="pwd" />
                <input type="password" maxlength="8" placeholder="密码" v-else v-model="pwd" />
                <div class="switch_button" :class="showPwd?'on':'off'" @click="showPwd=!showPwd">
                  <div class="switch_circle" :class="{right: showPwd}"></div>
                  <span class="switch_text">{{showPwd ? 'abc' : '...'}}</span>
                </div>
              </section>
            </section>
          </div>
          <button class="login_submit" v-show="loginWay" @click="logon">注册</button>
          <button class="login_submit" v-show="!loginWay" @click="login">登录</button>
        </form>
        <a href="javascript:;" class="about_us">关于我们</a>
      </div>
      <a href="javascript:" class="go_back" @click="$router.back()">
        <i class="iconfont icon-jiantou2"></i>
      </a>
    </div>
    <AlertTip :alertText="alertText" v-show="alertShow" @closeTip="closeTip" />
  </section>
</template>

<script>
import AlertTip from "../../components/AlertTip/AlertTip.vue";
import { MessageBox, Toast } from "mint-ui";
import { reqSendCode, reqSmsLogin, reqPwdLogin } from "../../api";
import NProgress from "nprogress";
import "nprogress/nprogress.css";

export default {
  data() {
    return {
      loginWay: false, // true代表短信登陆, false代表密码
      computeTime: 0, // 计时的时间
      showPwd: false, // 是否显示密码
      username: "", // 注册用户名
      password: "", // 注册密码
      repassword: "", // 注册确认密码
      name: "", // 登录用户名
      pwd: "", // 登录密码
      alertText: "", // 提示文本
      alertShow: false // 是否显示警告框
    };
  },

  methods: {
    showAlert(alertText) {
      this.alertShow = true;
      this.alertText = alertText;
    },
    // 注册
    logon() {
      const { username, password, repassword } = this;
      if (!username) {
        this.showAlert("用户名不能为空");
        return;
      }
      if (!password) {
        this.showAlert("密码不能为空");
        return;
      }
      if (password != repassword) {
        this.showAlert("两次密码不一致");
        return;
      }
      NProgress.start();
      let params = {
        username: this.username,
        password: this.password,
        balance: 1000
      };
      Bmob.User.register(params)
        .then(res => {
          this.showAlert("注册成功");
          NProgress.done();
        })
        .catch(err => {
          if (err.code == "202") {
            this.showAlert("用户名已存在");
          } else {
            this.showAlert("注册失败");
          }
          NProgress.done();
        });
    },
    login() {
      const { name, pwd } = this;
      if (!this.name) {
        this.showAlert("用户名不能为空");
        return;
      }
      if (!this.pwd) {
        this.showAlert("密码不能为空");
        return;
      }
      NProgress.start();
      Bmob.User.login(this.name, this.pwd)
        .then(res => {
          const username = res.username;
          const objectId = res.objectId;
          const balance = res.balance;
          window.sessionStorage.setItem("username", username);
          window.sessionStorage.setItem("objectId", objectId);
          window.sessionStorage.setItem("balance", balance);
          this.$router.replace("/profile");
          NProgress.done();
        })
        .catch(err => {
          if (err.code == "101") {
            this.showAlert("用户名或者密码错误");
          }
          NProgress.done();
        });
    },
    // 异步登陆
    // async login() {
    //   let result;
    //   // 前台表单验证
    //   // 密码登陆
    //   const { name, pwd, captcha } = this;
    //   if (!this.name) {
    //     // 用户名必须指定
    //     this.showAlert("用户名必须指定");
    //     return;
    //   } else if (!this.pwd) {
    //     // 密码必须指定
    //     this.showAlert("密码必须指定");
    //     return;
    //   } else if (!this.captcha) {
    //     // 验证码必须指定
    //     this.showAlert("验证码必须指定");
    //     return;
    //   }
    //   // 发送ajax请求密码登陆
    //   result = await reqPwdLogin({ name, pwd, captcha });

    //   // 停止计时
    //   if (this.computeTime) {
    //     this.computeTime = 0;
    //     clearInterval(this.intervalId);
    //     this.intervalId = undefined;
    //   }

    //   // 根据结果数据处理
    //   if (result.code === 0) {
    //     const user = result.data;
    //     // 将user保存到vuex的state
    //     this.$store.dispatch("recordUser", user);
    //     // 去个人中心界面
    //     this.$router.replace("/profile");
    //   } else {
    //     // 显示新的图片验证码
    //     this.getCaptcha();
    //     // 显示警告提示
    //     const msg = result.msg;
    //     this.showAlert(msg);
    //   }
    // },
    // 关闭警告框
    closeTip() {
      this.alertShow = false;
      this.alertText = "";
    }
  },

  components: {
    AlertTip
  }
};
</script>

<style lang="stylus" rel="stylesheet/stylus">
@import '../../common/stylus/mixins.styl';

.loginContainer {
  width: 100%;
  height: 100%;
  background: #fff;

  .loginInner {
    padding-top: 60px;
    width: 80%;
    margin: 0 auto;

    .login_header {
      .login_logo {
        font-size: 40px;
        font-weight: bold;
        color: rgb(0, 141, 225);
        text-align: center;
      }

      .login_header_title {
        padding-top: 40px;
        text-align: center;

        >a {
          color: #333;
          font-size: 14px;
          padding-bottom: 4px;

          &:first-child {
            margin-right: 40px;
          }

          &.on {
            color: rgb(0, 141, 225);
            font-weight: 700;
            border-bottom: 2px solid rgb(0, 141, 225);
          }
        }
      }
    }

    .login_content {
      >form {
        >div {
          display: none;

          &.on {
            display: block;
          }

          input {
            width: 100%;
            height: 100%;
            padding-left: 10px;
            box-sizing: border-box;
            border: 1px solid #ddd;
            border-radius: 4px;
            outline: 0;
            font: 400 14px Arial;

            &:focus {
              border: 1px solid #02a774;
            }
          }

          .login_message {
            position: relative;
            margin-top: 16px;
            height: 48px;
            font-size: 14px;
            background: #fff;

            .get_verification {
              position: absolute;
              top: 50%;
              right: 10px;
              transform: translateY(-50%);
              border: 0;
              color: #ccc;
              font-size: 14px;
              background: transparent;

              &.right_phone {
                color: black;
              }
            }
          }

          .login_verification {
            position: relative;
            margin-top: 16px;
            height: 48px;
            font-size: 14px;
            background: #fff;

            .switch_button {
              font-size: 12px;
              border: 1px solid #ddd;
              border-radius: 8px;
              transition: background-color 0.3s, border-color 0.3s;
              padding: 0 6px;
              width: 30px;
              height: 16px;
              line-height: 16px;
              color: #fff;
              position: absolute;
              top: 50%;
              right: 10px;
              transform: translateY(-50%);

              &.off {
                background: #fff;

                .switch_text {
                  float: right;
                  color: #ddd;
                }
              }

              &.on {
                background: rgb(0, 141, 225);
              }

              >.switch_circle {
                position: absolute;
                top: -1px;
                left: -1px;
                width: 16px;
                height: 16px;
                border: 1px solid #ddd;
                border-radius: 50%;
                background: #fff;
                box-shadow: 0 2px 4px 0 rgba(0, 0, 0, 0.1);
                transition: transform 0.3s;

                &.right {
                  transform: translateX(30px);
                }
              }
            }
          }

          .login_hint {
            margin-top: 12px;
            color: #999;
            font-size: 14px;
            line-height: 20px;

            >a {
              color: #02a774;
            }
          }
        }

        .login_submit {
          display: block;
          width: 100%;
          height: 42px;
          margin-top: 30px;
          border-radius: 4px;
          background: rgb(0, 141, 225);
          color: #fff;
          text-align: center;
          font-size: 16px;
          line-height: 42px;
          border: 0;
        }
      }

      .about_us {
        display: block;
        font-size: 12px;
        margin-top: 20px;
        text-align: center;
        color: #999;
      }
    }

    .go_back {
      position: absolute;
      top: 5px;
      left: 5px;
      width: 30px;
      height: 30px;

      >.iconfont {
        font-size: 20px;
        color: #999;
      }
    }
  }
}
</style>