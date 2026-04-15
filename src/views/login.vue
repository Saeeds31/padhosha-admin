<template>
  <div class="login-page d-flex justify-content-center align-items-center vh-100">
    <b-card class="login-card p-4 shadow-lg">
      <h2 class="text-center mb-4 text-primary">خوش آمدید</h2>

      <b-form @submit.prevent="handleLogin" id="loginForm">
        <!-- Username -->
        <b-form-group label-for="username">
          <div class="input-group mb-3">
            <b-form-input id="username" name="mobile" v-model="username" placeholder="Username" required></b-form-input>
            <span @click="sendOtp()" :class="{ 'disableButton': counter != 0 }"
              class="input-group-text bg-primary text-white">
              <i v-if="counter <= 0" class="bi-send"></i>
              <span v-else>{{ counter }}</span>
            </span>
          </div>
        </b-form-group>

        <b-form-group label="کد یکبار مصرف:" label-for="password">
          <v-otp-input ref="otpInput" input-classes="otp-input" inputmode="tel" separator="-" inputType="letter-numeric"
            :num-inputs="6" v-model:value="otp" />
        </b-form-group>

        <!-- Submit -->
        <b-button type="submit" variant="primary" class="w-100 btn-login">
          ورود به پنل

        </b-button>
      </b-form>

      <div v-if="message" class="mt-3 text-center text-danger">
        {{ message }}
      </div>
    </b-card>
  </div>
</template>

<script setup>
import {
  BCard,
  BForm,
  BFormGroup,
  BFormInput,
  BButton
} from 'bootstrap-vue-3';
import axios from 'axios';
import { ref } from "vue"
import { toast } from 'vue3-toastify'
import { useRouter } from "vue-router";
import VOtpInput from "vue3-otp-input";
import { setCookie } from "../tools/methods.js"
import { useAdmin } from '@/stores/modules/admin';
const store = useAdmin();

const router = useRouter();
let username = ref('');
let password = ref('');
let message = ref('');
let otp = ref();
let counter = ref(0);

function handleLogin() {
  if (!username.value || !otp) {
    toast.error("لطفا شماره تماس و کد یکبار مصرف را وارد کنید")
    return
  }
  let fd = new FormData();
  fd.append("mobile", username.value)
  fd.append("token", otp.value)
  axios.post("/login-verify", fd).then(res => {
    toast.success("ورود موفقیت  آمیز بود! 🎉")
    axios.defaults.headers.common.Authorization = `Bearer ${res.data.token}`;
    message.value = '';
    setCookie("token", res.data.token)
    store.getAdminDetail();

    router.push("/dashboard")
  }).catch(err => {
    console.log(err);
    message.value = 'نام کاربری و رمز عبور اشتباه است.'

  })
}
function sendOtp() {
  counter.value = -1;
  let fd = new FormData();
  fd.append("mobile", username.value);
  axios.post("/send-token", fd).then((res) => {
    toast.success("کد یکبار مصرف برای شما ارسال شد! ")
    showTimer();
  }).catch((err) => {
    counter.value = 0;
    toast.error(err.response?.data?.message)
  })
}
let timer = null;
function showTimer() {
  counter.value = 60;
  clearInterval(timer);
  timer = setInterval(() => {
    counter.value--;
    if (counter.value == 0) {
      clearInterval(timer);
    }
  }, 1000);
}
</script>

<style scoped>
.login-card {
  width: 380px;
  border-radius: 15px;
  padding: 2rem;
  background: #ffffffee;
}

.input-group-text {
  border-radius: 8px 0 0 8px;
}

.b-form-input {
  border-radius: 0 8px 8px 0;
}

#username,
#password {
  border-radius: unset;
}

.btn-login {
  border-radius: 8px;
  transition: transform 0.2s, box-shadow 0.2s;
}

.btn-login:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 12px rgba(0, 0, 0, 0.2);
}
</style>
<style>
input.otp-input {
  width: 36px;
  background: white;
  border: 1px solid #1213;
  border-radius: 4px;
  text-align: center;
  color: black;
}

.otp-input-container {
  display: flex;
  justify-content: center;
  direction: ltr;
}
</style>