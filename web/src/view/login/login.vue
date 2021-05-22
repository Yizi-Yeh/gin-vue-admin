<template>
  <div id="userLayout" class="user-layout-wrapper">
    <div class="container">
      <div class="top">
        <div class="desc">
          <img class="logo_login" src="https://upload.wikimedia.org/wikipedia/commons/thumb/4/49/Weiwuying_logo.svg/1280px-Weiwuying_logo.svg.png" alt="" />
        </div>
        <div class="header">
          <a href="/">
            <!-- <img src="~@/assets/logo.png" class="logo" alt="logo" /> -->
          
          </a>
        </div>
      </div>
      <div class="main">
        <el-form
          :model="form"
          :rules="rules"
          ref="loginForm"
          @keyup.enter="submitForm"
        >
          <el-form-item prop="username">
            <el-input placeholder="請輸入帳號" v-model="form.username">
              <template #suffix>
                <i class="el-input__icon el-icon-user"></i>
              </template>
            </el-input>
          </el-form-item>
          <el-form-item prop="password">
            <el-input
              :type="lock === 'lock' ? 'password' : 'text'"
              placeholder="請輸入密碼"
              v-model="form.password"
            >
              <template #suffix>
                <i :class="'el-input__icon el-icon-' + lock" @click="changeLock"></i>
              </template>
            </el-input>
          </el-form-item>
          <el-form-item style="position: relative">
            <el-input
              v-model="form.captcha"
              placeholder="請輸入驗證碼"
              style="width: 60%"
            />
            <div class="vPic">
              <img
                v-if="picPath"
                :src="picPath"
                alt="請輸入驗證碼"
                @click="loginVefify()"
              />
            </div>
          </el-form-item>
          
          <div class="input-container">
          <el-form-item>
            <el-button type="primary" @click="submitForm" style="width: 80%"
              >登 入</el-button
            >
          </el-form-item>

          <el-form-item>
            <el-button type="primary" @click="submitForm" style="width:  80%"
              >訪客QRcode</el-button
            >
          </el-form-item>

          <el-form-item>
            <el-button type="primary" @click="submitForm" style="width:  80%"
              >訪客預約註冊</el-button
            >
          </el-form-item>
        </div>
        </el-form>
      </div>

      <!-- <div class="footer">
        <div class="links">
          <a href="http://doc.henrongyi.top/"
            ><img src="@/assets/docs.png" class="link-icon"
          /></a>
          <a href="https://www.yuque.com/flipped-aurora/"
            ><img src="@/assets/yuque.png" class="link-icon"
          /></a>
          <a href="https://github.com/flipped-aurora/gin-vue-admin"
            ><img src="@/assets/github.png" class="link-icon"
          /></a>
          <a href="https://space.bilibili.com/322210472"
            ><img src="@/assets/video.png" class="link-icon"
          /></a>
        </div>
        <div class="copyright">Copyright &copy; {{ curYear }} 💖flipped-aurora</div>
      </div> -->
    </div>
  </div>
</template>

<script>
import { captcha } from "@/api/user";
import { getCurrentInstance, ref, reactive } from "vue";
import { useStore } from 'vuex';
const checkUsername = (rule, value, callback) => {
  if (value.length < 5 || value.length > 12) {
    return callback(new Error("請輸入正確的帳號"));
  } else {
    callback();
  }
};
const checkPassword = (rule, value, callback) => {
  if (value.length < 6 || value.length > 12) {
    return callback(new Error("請輸入正確的密碼"));
  } else {
    callback();
  }
};
export default {
  name: "Login",
  setup() {
    const { ctx } = getCurrentInstance();
    const store = useStore();
    const curYear = ref(0);
    const lock = ref("lock");
    const picPath = ref("");
    const form = reactive({
      username: "",
      password: "",
      captcha: "",
      captchaId: "",
    });
    const rules = reactive({
      username: [{ validator: checkUsername, trigger: "blur" }],
      password: [{ validator: checkPassword, trigger: "blur" }],
    });
    const LoginIn = (form) => store.dispatch("user/LoginIn",form);
    const login = async () => {
      return await LoginIn(form)
    };
    const submitForm = async () => {
      ctx.$refs.loginForm.validate(async (v) => {
        if (v) {
          const flag = await login();
          if(!flag){
            loginVefify();
          }
        } else {
          ctx.$message({
            type: "error",
            message: "請填入正確的登入資訊",
            showClose: true,
          });
          loginVefify();
          return false;
        }
      });
    };
    const changeLock = () => {
      lock.value === "lock" ? (lock.value = "unlock") : (lock.value = "lock");
    };
    const loginVefify = () => {
      captcha({}).then((ele) => {
        picPath.value = ele.data.picPath;
        form.captchaId = ele.data.captchaId;
      });
    };
    loginVefify();
    curYear.value = new Date().getFullYear();
    return {
      lock,
      form,
      rules,
      changeLock,
      picPath,
      loginVefify,
      submitForm,
      curYear,
    };
  }
};
</script>

<style scoped lang="scss">
@import "@/style/login.scss";
</style>
