<template>
  <div v-if="checkPermission(['employer_update'])" class="container py-4">
    <b-card>
      <h5 class="mb-3">ویرایش کارفرما</h5>
      <b-form @submit.prevent="handleSubmit">
        <b-row>
          <b-col cols="12" md="6">
            <b-form-group label="نام کامل" label-for="full_name">
              <b-form-input id="full_name" v-model="form.full_name" />
              <small class="text-danger" v-if="errors.full_name">{{ errors.full_name[0] }}</small>
            </b-form-group>
          </b-col>
          <b-col cols="12" md="6">
            <b-form-group label="نام کسب و کار" label-for="bussines_label">
              <b-form-input id="bussines_label" v-model="form.bussines_label" />
              <small class="text-danger" v-if="errors.bussines_label">{{ errors.bussines_label[0] }}</small>
            </b-form-group>
          </b-col>
          <b-col cols="12" md="6">
            <b-form-group label="شماره تماس" label-for="mobile">
              <b-form-input id="mobile" v-model="form.mobile" />
              <small class="text-danger" v-if="errors.mobile">{{ errors.mobile[0] }}</small>
            </b-form-group>
          </b-col>
          <b-col cols="12" md="6">
            <b-form-group label="رمز عبور" label-for="password">
              <b-form-input id="password" v-model="form.password" />
              <small class="text-danger" v-if="errors.password">{{ errors.password[0] }}</small>
            </b-form-group>
          </b-col>
          <b-col cols="12" md="12">
            <b-form-group label="تصویر" label-for="image">
              <VueFileAgent @update:raw-model-value="imageLoaded" :raw-model-value="oldImage" :maxFiles="1"
                accept=".pdf,.jpg,.png,.webp" theme="grid" deletable sortable>
              </VueFileAgent>

              <small v-if="errors.path" class="text-danger">{{ errors.path[0] }}</small>
            </b-form-group>
          </b-col>



        </b-row>

        <div class="mt-3">
          <b-button type="submit" :disabled="loading" variant="primary">
            <i class="bi bi-save2"></i>
            <span class="mx-2">
              ویرایش
              کارفرما
            </span>
          </b-button>
        </div>
      </b-form>
    </b-card>
  </div>
</template>

<script setup>
import { reactive, ref, onMounted } from 'vue'
import axios from 'axios'
import { toast } from 'vue3-toastify'
import 'vue3-toastify/dist/index.css'
import { BForm, BFormGroup, BFormInput, BButton, BCard, BRow, BCol } from 'bootstrap-vue-3'
import { useRoute } from 'vue-router'
import { useAdmin } from '@/stores/modules/admin';
const store = useAdmin();
const checkPermission = store.checkPermission;
let loading = ref(false);
const route = useRoute();
const oldImage = ref([]);
const form = reactive({
  full_name: '',
  bussines_logo: [],
  bussines_label: '',
  mobile: '',
  password: ''
})
const errors = reactive({})

onMounted(async () => {
  loading.value = true;
  try {
    // GET اطلاعات مقاله
    const res = await axios.get(`/employers/${route.params.id}`)
    if (res.data.data.bussines_logo)
      oldImage.value =
        [{
          name: res.data.data.bussines_logo.split('/').pop(),
          size: 0,
          type: 'image/jpeg',
          ext: res.data.data.bussines_logo.split('.').pop(),
          url: `${baseImageAddress}${res.data.data.bussines_logo}`,
        }];
    Object.assign(form, res.data.data);
    form.mobile = res.data.data.user.mobile
    form.full_name = res.data.data.user.full_name
  } catch (err) {
    console.log(err);
    toast.error('خطا در دریافت اطلاعات لوگو ❌')
  } finally {
    loading.value = false;
  }
})
function imageLoaded(files) {
  if (files.length) {
    form.bussines_logo = files[0].file
  } else {
    form.bussines_logo = '';
  }
}
const handleSubmit = async () => {
  loading.value = true;

  Object.keys(errors).forEach(k => delete errors[k])
  try {
    const formData = new FormData()
    for (const key in form) {
      if (key != 'bussines_logo' && form[key]) formData.append(key, form[key])
    }
    formData.append("_method", "PUT");
    if (form.bussines_logo) {
      formData.append("bussines_logo", form.bussines_logo);
    }
    await axios.post(`/employers/${route.params.id}`, formData, {
      headers: { 'Content-Type': 'multipart/form-data' }
    })
    toast.success('کارفرما با موفقیت ویرایش شد ✅');
  } catch (err) {
    if (err.response?.status === 422) {
      Object.assign(errors, err.response.data.errors)
      toast.error('خطاهای فرم را بررسی کنید ❌')
    } else {
      toast.error('خطا در ارسال اطلاعات ❌')
    }
  } finally {
    loading.value = false;
  }
}
</script>
