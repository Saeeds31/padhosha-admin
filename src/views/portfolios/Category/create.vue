<template>
  <div class="container py-4 mt-4 bg-gray" v-if="checkPermission(['portfoliocategory_store'])">
    <h3 class=" p-2">
      <i class="bi bi-list-nested"></i>
      <span>
        ایجاد دسته بندی جدید
      </span>
    </h3>
    <b-form @submit.prevent="handleSubmit">
      <b-row>
        <!-- Title -->
        <b-col cols="12" md="6">
          <b-form-group label="عنوان" label-for="title">
            <b-form-input id="title" v-model="form.title" />
            <small v-if="errors.title" class="text-danger">{{ errors.title[0] }}</small>
          </b-form-group>
        </b-col>


        <!-- Icon -->
        <b-col cols="12" md="6">
          <b-form-group label="آیکن" label-for="icon">
            <VueFileAgent @select="imageLoaded1" :maxFiles="1" accept=".pdf,.jpg,.png,.webp" theme="grid" deletable
              sortable />
            <small v-if="errors.icon" class="text-danger">{{ errors.icon[0] }}</small>
          </b-form-group>
        </b-col>
        <!-- Description (Editor) -->
        <b-col cols="12">
          <b-form-group label="توضیح" label-for="description">
            <Editor v-model="form.description" />
            <small v-if="errors.description" class="text-danger">{{ errors.description[0] }}</small>
          </b-form-group>
        </b-col>

        <!-- Meta Title -->
        <b-col cols="12" md="6">
          <b-form-group label="Meta Title" label-for="meta_title">
            <b-form-input id="meta_title" v-model="form.meta_title" />
            <small v-if="errors.meta_title" class="text-danger">{{ errors.meta_title[0] }}</small>
          </b-form-group>
        </b-col>

        <!-- Meta Description -->
        <b-col cols="12" md="6">
          <b-form-group label="Meta Description" label-for="meta_description">
            <b-form-input id="meta_description" v-model="form.meta_description" />
            <small v-if="errors.meta_description" class="text-danger">{{ errors.meta_description[0] }}</small>
          </b-form-group>
        </b-col>
      </b-row>

      <div class="mt-3">
        <b-button type="submit" :disabled="loading" variant="success">
          <i class="bi bi-save2"></i>
          <span class="mx-2">
            ایجاد دسته‌بندی

          </span>
        </b-button>
      </div>
    </b-form>
  </div>
</template>

<script setup>
import { reactive, ref, onMounted } from 'vue'
import axios from 'axios'
import { toast } from 'vue3-toastify'
import 'vue3-toastify/dist/index.css'
import { BForm, BFormGroup, BFormInput, BButton, BCard, BRow, BCol } from 'bootstrap-vue-3'
import Editor from '@/components/shared/Editor.vue'
import { useAdmin } from '@/stores/modules/admin';
const store = useAdmin();
const checkPermission = store.checkPermission;
let loading = ref(false);

const form = reactive({
  title: '',
  icon: null,
  meta_title: '',
  meta_description: '',
  description: '',
})

const errors = reactive({})



function imageLoaded1(files) {
  form.icon = files[0].file
}

const handleSubmit = async () => {
  Object.keys(errors).forEach(k => delete errors[k])
  loading.value = true;
  try {
    const formData = new FormData()
    for (const key in form) {
      formData.append(key, form[key] ?? "")
    }
    await axios.post('/portfolio-categories', formData)
    toast.success('دسته‌بندی با موفقیت ایجاد شد ✅')

    Object.keys(form).forEach(k => form[k] = null)

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