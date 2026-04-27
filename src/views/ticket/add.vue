<template>
    <div id="ticket-page" class="container mt-4">
        <b-card title="ثبت تیکت جدید" class="mt-3  shadow-sm">
            <b-form @submit.prevent="handleSubmit">

                <!-- عنوان تیکت -->
                <b-form-group label="موضوع تیکت" label-for="subject">
                    <b-form-input id="subject" v-model="ticket.title"  placeholder="موضوع را وارد کنید" />
                    <small class="text-danger" v-if="errors.title">{{ errors.title[0] }}</small>
                </b-form-group>

                <!-- توضیحات -->
                <b-form-group label="توضیحات">
                    <Editor v-model="ticket.description" rows="5" placeholder="توضیحات خود را بنویسید..." />
                    <small class="text-danger" v-if="errors.description">{{ errors.description[0] }}</small>
                </b-form-group>

                <!-- فایل پیوست -->
                <b-form-group label="فایل پیوست">
                    <VueFileAgent @select="imageLoaded" :maxFiles="1" accept=".pdf,.jpg,.png,.webp" theme="grid"
                        deletable sortable />
                    <small class="text-danger" v-if="errors.file">{{ errors.file[0] }}</small>
                </b-form-group>

                <!-- ضبط صوت -->
                <b-form-group label="ضبط وویس (اختیاری)">
                    <div class="d-flex align-items-center gap-2">
                        <b-button variant="outline-primary" @click="toggleRecording">
                            <span v-if="!isRecording">🎙️ شروع ضبط</span>
                            <span v-else>⏹️ توقف ضبط</span>
                        </b-button>

                        <audio v-if="audioUrl" :src="audioUrl" controls class="ms-2"></audio>
                    </div>
                </b-form-group>

                <!-- دکمه ارسال -->
                <b-button type="submit" :disabled="loader" variant="success">ارسال تیکت</b-button>
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
const router = useRouter();
let loader = ref(false);
const ticket = ref({
    title: '',
    description: '',
    file: null,
    audio: null
})
const errors = reactive({});
let mediaRecorder = null
let audioChunks = []
const isRecording = ref(false)
const audioUrl = ref(null)
function imageLoaded(files) {
    form.file = files[0].file
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
                ticket.value.audio = blob
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

const handleSubmit = async () => {
    // اینجا می‌تونی دیتا رو به API ارسال کنی (FormData مناسب برای فایل‌ها)
    const formData = new FormData()
    formData.append('title', ticket.value.title)
    formData.append('description', ticket.value.description)
    formData.append('file', ticket.value.file??'')
     formData.append('audio', ticket.value.audio??'')
    try {
        loader.value = true;
        let { data } = await axios.post('tickets', formData);
        Swal.fire("موفق", "پیغام شما با موفقیت ثبت شد", "success");
        router.push('/ticket')
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
    max-width: 600px;
    margin: 20px auto;
}
</style>