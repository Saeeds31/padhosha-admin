<template>
    <div class="container mt-4 bg-gray" v-if="checkPermission(['employer_store'])">
        <h3 class=" p-2">
            <i class="bi bi-book-half"></i>
            <span>
                ایجاد کارفرما جدید
            </span>
        </h3>
        <b-form @submit.prevent="submitForm">
            <b-row>
                <!-- Title -->
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
                    <b-form-group label="تصویر (URL)">
                        <VueFileAgent @select="imageLoaded" :maxFiles="1" accept=".pdf,.jpg,.png,.webp" theme="grid"
                            deletable sortable />
                        <small class="text-danger" v-if="errors.bussines_logo">{{ errors.bussines_logo[0] }}</small>
                    </b-form-group>
                </b-col>
            </b-row>

            <div class="mt-3">
                <b-button type="submit" :disabled="loading" variant="primary">
                    <i class="bi bi-save2">
                        ذخیره
                    </i>
                </b-button>
            </div>
        </b-form>
    </div>
</template>

<script setup>
import { reactive, ref } from 'vue'
import axios from 'axios'
import { toast } from 'vue3-toastify'
import 'vue3-toastify/dist/index.css'
import { BForm, BFormGroup, BFormInput, BButton, BRow, BCol } from 'bootstrap-vue-3'
import { displayError } from '@/composable/useError'
import { useAdmin } from '@/stores/modules/admin';
const store = useAdmin();
const checkPermission = store.checkPermission;
let loading = ref(false);
const form = reactive({
    full_name: '',
    bussines_logo: '',
    bussines_label: '',
    mobile: '',
    password: ''

})
const errors = reactive({})
function imageLoaded(files) {
    form.bussines_logo = files[0].file
}


const submitForm = async () => {

    Object.keys(errors).forEach(key => delete errors[key])
    loading.value = true;
    try {
        const formData = new FormData()
        for (const key in form) {
            formData.append(key, form[key] ?? '')
        }
        await axios.post('/employers', formData)
        toast.success('کارفرما با موفقیت ذخیره شد ✅')
        Object.keys(form).forEach(key => (form[key] = key))
    } catch (err) {
        if (err.response && err.response.status === 422) {
            Object.assign(errors, err.response.data.errors)
            displayError(err.response.data.errors)
        } else {
            toast.error('خطا در ارسال اطلاعات ❌')
        }
    } finally {
        loading.value = false;
    }
}
</script>