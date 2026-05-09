<template>
    <div id="ticket-page" class="container mt-4">
        <b-card title="ثبت رسید جدید" class="mt-3  shadow-sm">

            <b-form @submit.prevent="handleSubmit">
                <div class="col-md-12">
                    <label class="form-label">انتخاب مشتری</label>

                    <multiselect @search-change="loadEmployer" v-model="selectedEmployer" placeholder="جستجوی کسب و کار"
                        open-direction="bottom" :options="options" label="label" track-by="id" :searchable="true"
                        :multiple="false" :close-on-select="true" :show-labels="false">
                        <template slot="noOptions">
                            جستجو کنید
                        </template>
                        <template slot="noResult">
                            <span v-if="isRequesting" v-text="'در حال جستجو...'" />
                            <span v-else v-text="'موردی یافت نشد'"></span>
                        </template>
                    </multiselect>
                    <small class="text-danger" v-if="errors.employer_id">{{ errors.employer_id[0] }}</small>

                </div>
                <b-form-group label="عنوان رسید" label-for="subject">
                    <b-form-input id="subject" v-model="newReceipt.title" placeholder="عنوان را وارد کنید" />
                    <small class="text-danger" v-if="errors.title">{{ errors.title[0] }}</small>
                </b-form-group>

                <div class="mb-3">
                    <label for="receipt-amount" class="form-label">
                        تومان ({{ Number(newReceipt.amount).toLocaleString('fa') }})
                    </label>
                    <input placeholder="مبلغ واریزی" type="number" class="form-control" id="receipt-amount"
                        v-model="newReceipt.amount" required>
                    <small class="text-danger" v-if="errors.amount">{{ errors.amount[0] }}</small>
                </div>
                <!-- توضیحات -->
                <b-form-group label="توضیحات">
                    <textarea class="form-control" v-model="newReceipt.description" rows="5"
                        placeholder="توضیحات خود را بنویسید...">
    </textarea>
                    <small class="text-danger" v-if="errors.description">{{ errors.description[0] }}</small>
                </b-form-group>

                <!-- فایل پیوست -->
                <b-form-group label="فایل پیوست">
                    <VueFileAgent @select="imageLoaded" :maxFiles="1" accept=".pdf,.jpg,.png,.webp" theme="grid"
                        deletable sortable />
                    <small class="text-danger" v-if="errors.image">{{ errors.image[0] }}</small>
                </b-form-group>

                <!-- دکمه ارسال -->
                <b-button type="submit" :disabled="loader" variant="success">ارسال رسید</b-button>
            </b-form>
        </b-card>
    </div>
</template>

<script setup>
import { ref, reactive } from 'vue'
import Editor from '@/components/shared/editor.vue';
import axios from "axios"
import Swal from "sweetalert2";
import { useRouter } from 'vue-router';
import { toast } from 'vue3-toastify'
import 'vue3-toastify/dist/index.css'
const router = useRouter();
let loader = ref(false);
const newReceipt = ref({
    title: '',
    description: '',
    amount: '',
    image: ''
})
const selectedEmployer = ref(null)
const options = ref([]);
let abortController = null;
const errors = reactive({});
let mediaRecorder = null
let audioChunks = []
const isRecording = ref(false)
const audioUrl = ref(null)
function imageLoaded(files) {
    newReceipt.value.image = files[0].file
}
// شروع یا توقف ضبط صوت
const toggleRecording = async () => {
    if (!isRecording.value) {
        try {
            const stream = await navigator.mediaDevices.getUserMedia({ audio: true })
            mediaRecorder = new MediaRecorder(stream)
            audioChunks = []

            mediaRecorder.ondataavailable = e => {
                audioChunks.push(e.data)
            }
            mediaRecorder.onstop = () => {
                const blob = new Blob(audioChunks, { type: 'audio/webm' })
                newReceipt.value.audio = blob
                audioUrl.value = URL.createObjectURL(blob)
            }

            mediaRecorder.start()
            isRecording.value = true
        } catch (err) {
            alert('دسترسی به میکروفون برقرار نشد: ' + err.message)
        }
    } else {
        mediaRecorder.stop()
        isRecording.value = false
    }
}
const loadEmployer = async (searchQuery) => {
    if (abortController) {
        abortController.abort();
    }
    abortController = new AbortController();
    try {
        const { data } = await axios.get('/employers?search=' + searchQuery ?? '', {
            signal: abortController.signal,
        })
        const ops = data.data.data.map(u => ({ id: u.id, label: `${u.bussines_label}` }))
        options.value = ops;

    } catch (error) {
        if (axios.isCancel(error)) {
            console.log('درخواست قبلی کنسل شد:', error.message);
        } else {
            toast.error('خطا در جستجوی کاربران')
        }
    }
}
const handleSubmit = async () => {
    // اینجا می‌تونی دیتا رو به API ارسال کنی (FormData مناسب برای فایل‌ها)
    const formData = new FormData()
    formData.append('title', newReceipt.value?.title);
    formData.append('amount', newReceipt.value?.amount);
    formData.append('description', newReceipt.value?.description);
    formData.append('image', newReceipt.value?.image);
    formData.append('employer_id', selectedEmployer.value?.id)

    try {
        loader.value = true;
        let { data } = await axios.post('employers-receipt', formData);
        Swal.fire("موفق", "پیغام شما با موفقیت ثبت شد", "success");
        router.push('/employers/reciepts')
    } catch (err) {
        if (err.response && err.response.status === 422) {
            Object.assign(errors, err.response.data.errors)
        }
    } finally {
        loader.value = false;

    }
}
</script>

<style scoped>
.card {
    margin: 20px auto;
}
</style>