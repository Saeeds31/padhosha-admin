<template>
    <b-container fluid class="ticket-page py-4">
        <section v-if="mainData">

            <div class="card mb-3">
                <div class="card-header">
                    <h3 class=" p-2">
                        <i class="bi bi-person-workspace"></i>
                        <span>
                            ارجاع تیکت به کارشناس </span>
                    </h3>
                </div>
                <div class="card-body">

                    <form @submit.prevent="sendReferd()" class="row g-3">
                        <div class="col-md-12">
                            <label class="form-label">انتخاب کاربر</label>

                            <multiselect @search-change="loadUsers" v-model="selectedUser" placeholder="انتخاب کاربر"
                                open-direction="bottom" :options="options" label="label" track-by="id"
                                :searchable="true" :multiple="false" :close-on-select="true" :show-labels="false">
                                <template slot="noOptions">
                                    جستجو کنید
                                </template>
                                <template slot="noResult">
                                    <span v-if="isRequesting" v-text="'در حال جستجو...'" />
                                    <span v-else v-text="'موردی یافت نشد'"></span>
                                </template>
                            </multiselect>
                        </div>
                        <div class="col-12">
                            <button type="submit" class="btn btn-primary" :disabled="loading">
                                <i class="bi bi-save2"></i>
                                <span class="mx-2">

                                    {{ loading ? 'در حال ذخیره...' : 'ارجاع به کارشناس' }}
                                </span>
                            </button>
                        </div>
                    </form>
                </div>
            </div>
            <div class="card mb-3">
                <div class="card-header d-flex align-items-center justify-content-between">
                    <h5 class="fy">
                        <span>تیکت شماره: </span>
                        <b>
                            {{ mainData.ticket.id }}
                        </b>
                    </h5>

                    <h5 class="fy">
                        <span>آخرین بروز رسانی: </span>
                        <b>
                            {{ mainData.messages.length ? new
                                Date(mainData.messages[0].created_at).toLocaleDateString('fa') : "-" }}
                        </b>
                    </h5>

                </div>
                <div class="card-body">
                    <div class="row">
                        <b-col cols="12" md="4" class="mb-3">
                            <b-card class="stat-card" body-class="d-flex align-items-center">
                                <div class="stat-icon bg-success text-white">
                                    <i class="fas fa-exclamation-circle"></i>
                                </div>
                                <div class="stat-content">
                                    <div class="stat-title fy">وضعیت تیکت </div>
                                    <div class="stat-value text-success fy">
                                        {{ translater(mainData.ticket.status) }}
                                    </div>
                                </div>
                            </b-card>
                        </b-col>

                        <b-col cols="12" md="4" class="mb-3">
                            <b-card class="stat-card" body-class="d-flex align-items-center">
                                <div class="stat-icon bg-dark text-white">
                                    <i class="fas fa-exclamation-circle"></i>
                                </div>
                                <div class="stat-content">
                                    <div class="stat-title fy">در دست اقدام کارشناس </div>
                                    <div class="stat-value text-dark fy">
                                        {{ mainData.ticket.doer ? mainData.ticket.doer.full_name : '-' }}
                                    </div>
                                </div>
                            </b-card>
                        </b-col>
                        <b-col cols="12" md="4" class="mb-3">
                            <b-card class="stat-card" body-class="d-flex align-items-center">
                                <div class="stat-icon bg-primary text-white">
                                    <i class="fas fa-exclamation-circle"></i>
                                </div>
                                <div class="stat-content">
                                    <div class="stat-title fy">هزینه تیکت </div>
                                    <div class="stat-value text-primary fy">
                                        {{ mainData.cost ? Number(mainData.cost.amount).toLocaleString('fa') + ' تومان'
                                            :
                                            'بدون هزینه' }}
                                    </div>
                                </div>
                            </b-card>
                        </b-col>
                    </div>
                </div>
            </div>
            <b-card v-if="mainData.ticket.status != 'closed'" class="mb-3 p-3 bg-light">
                <div class="card-header">
                    <h5 class="mb-3">ارسال پاسخ جدید</h5>
                </div>
                <div class="card-body">
                    <b-form @submit.prevent="sendMessage()">
                        <b-col cols="12">
                            <b-form-group label="پاسخ کامل">
                                <Editor v-model="form.content" />

                            </b-form-group>
                        </b-col>
                        <b-col cols="12" md="12">
                            <b-form-group label="فایل پیوست">
                                <VueFileAgent @select="imageLoaded" :maxFiles="1" accept=".pdf,.jpg,.png,.webp"
                                    theme="grid" deletable sortable />
                            </b-form-group>
                        </b-col>
                        <b-button :disabled="loader" variant="primary" type="submit" class="mt-2">
                            ارسال پاسخ
                        </b-button>
                    </b-form>
                </div>

            </b-card>
            <b-card v-else class="mb-3 p-3 bg-light">
                <div class="card-header">
                    <h5 class="mb-3">ارسال پاسخ جدید</h5>
                </div>
                <div class="card-body">
                    <p class="alert alert-danger">
                        برای تیکت در وضعیت بسته شده امکان ارسال پاسخ وجود ندارد
                    </p>
                </div>

            </b-card>

            <div class="card">
                <div class="card-header d-flex align-items-center justify-content-between">
                    <h5 class="fy">
                        <span>موضوع تیکت: </span>
                        <b>
                            {{ mainData.ticket.title }}
                        </b>
                    </h5>
                </div>
                <div class="card-body">
                    <div class="chat-container">
                        <transition-group name="fade" tag="div">
                            <div v-for="(msg, idx) in mainData.messages" :key="msg.id"
                                :class="['chat-message', msg.sender_side == 'employer' ? 'right' : 'left']">
                                <b-card class="p-3"
                                    :class="msg.sender_side == 'employer' ? 'bg-primary text-white' : 'bg-white border shadow-sm'">
                                    <span :class="msg.sender_side == 'employer' ? ' text-white' : 'text-muted'"
                                        class="fy">
                                        {{ msg.sender ? msg.sender.full_name : '' }}
                                    </span>
                                    <hr>
                                    <div v-html="msg.message" class="message-content"></div>

                                    <div class="attachments mt-2">
                                        <template v-if="msg.file">
                                            <b-link :href="fileHandler(msg.file)" target="_blank">
                                                مشاهده فایل الصاق شده
                                            </b-link>
                                        </template>
                                        <template v-if="msg.voice">
                                            <audio controls :src="fileHandler(msg.voice)" class="mt-1"
                                                style="width: 100%;"></audio>
                                        </template>
                                    </div>

                                    <small :class="msg.sender_side == 'employer' ? ' text-white' : 'text-muted'"
                                        class=" d-block mt-2 text-right">
                                        تاریخ:
                                        {{ new
                                            Date(msg.created_at).toLocaleDateString('fa') }}</small>
                                </b-card>
                            </div>
                        </transition-group>
                    </div>
                </div>
            </div>

        </section>
    </b-container>
</template>
<script setup>
import axios from 'axios';
import { onMounted, reactive, ref } from 'vue';
import Editor from '@/components/shared/editor.vue';
import Swal from "sweetalert2";
import { useRoute } from 'vue-router';
const route = useRoute();
const mainData = ref(null);
async function getTicket() {
    let { data } = await axios.get(`/tickets/${route.params.id}`);
    mainData.value = data.data;
}
let abortController = null;
const options = ref([]);
const selectedUser = ref(null)
const loadUsers = async (searchQuery) => {
    if (abortController) {
        abortController.abort();
    }
    abortController = new AbortController();
    try {
        const { data } = await axios.get('/users?search=' + searchQuery ?? '', {
            signal: abortController.signal,
        })
        const ops = data.data.map(u => ({ id: u.id, label: `${u.full_name} (${u.mobile})` }))
        options.value = ops;

    } catch (error) {
        if (axios.isCancel(error)) {
            console.log('درخواست قبلی کنسل شد:', error.message);
        } else {
            toast.error('خطا در جستجوی کاربران')
        }
    }
}
let loader = ref(false);
const form = reactive({
    file: '',
    content: ''
})
function imageLoaded(files) {
    form.file = files[0].file
}
onMounted(() => {
    getTicket();
})
function fileHandler(path) {
    return window.baseImageAddress + path

}
const translater = (status) => {
    switch (status) {
        case 'answered': return 'پاسخ داده شده';
        case 'closed': return 'بسته شده';
        case 'pending': return 'در انتظار بررسی';
        case 'awaiting_payment': return 'در انتظار پرداخت';
        case 'referred': return 'ارجاع شده';
    }
};
async function sendMessage() {
    loader.value = true;
    let fd = new FormData();
    if (!form.content || (form.content && form.content.length < 3)) {
        return Swal.fire("خطا", "پیام خود را به صورت کامل وارد کنید", "success");
    }
    fd.append('ticket_id', mainData.value.ticket.id)
    fd.append('file', form.file ?? '')
    fd.append('voice', '')
    fd.append('message', form.content)
    try {
        let { data } = await axios.post('/messages', fd);
        Swal.fire("موفق", "پیام شما با موفقیت ارسال شد", "success");
        getTicket();

    } catch (error) {
        Swal.fire("خطا", "خطایی در ارسال پیش آمده است", "danger");

    } finally {
        loader.value = false;
    }
}
</script>
<style scoped>
.ticket-page {
    margin: auto;
}

.chat-container {
    display: flex;
    flex-direction: column-reverse;
}

.chat-message {
    display: flex;
    margin: 10px 0;
}

.chat-message.left {
    justify-content: flex-start;
}

.chat-message.right {
    justify-content: flex-end;
}

.chat-message .b-card {
    max-width: 65%;
    overflow-x: auto;
}

.message-content p {
    margin-bottom: 0;
    white-space: pre-wrap;
}

.fade-enter-active,
.fade-leave-active {
    transition: all 0.3s ease;
}

.fade-enter-from {
    opacity: 0;
    transform: translateY(10px);
}
</style>

<style scoped>
.employer-dashboard {
    direction: rtl;
}

/* کارت‌ها */
.stat-card {
    border-radius: 12px;
    box-shadow: 0 8px 18px rgba(0, 0, 0, 0.04);
    border: 1px solid #f0f0f0;
    transition: transform 0.15s ease, box-shadow 0.15s ease;
}

.stat-card:hover {
    transform: translateY(-4px);
    box-shadow: 0 12px 25px rgba(0, 0, 0, 0.07);
}

/* آیکون‌ها */
.stat-icon {
    width: 46px;
    height: 46px;
    border-radius: 50%;
    display: inline-flex;
    align-items: center;
    justify-content: center;
    margin-left: 12px;
    font-size: 20px;
    flex-shrink: 0;
}

/* متن داخل کارت */
.stat-content {
    flex: 1;
}

.stat-title {
    font-size: 0.85rem;
    color: #6c757d;
}

.stat-value {
    font-size: 1.2rem;
    font-weight: 600;
    margin-top: 4px;
    margin-bottom: 4px;
}

h5 {
    font-size: 1rem;
    font-weight: 600;
}

.small {
    font-size: 0.85rem;
    border-bottom: 1px solid #1213;
    padding-bottom: 8px;
}

.dashboardItem strong {
    width: 100%;
    min-height: 40px;
}

.dashboardItem .small {
    background-color: var(--theme-color);
    color: white !important;
    width: 100%;
    padding: 8px 0;
}

.dashboardItem {
    border: 1px solid #1213;
    border-radius: 8px;
    min-height: 100px;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: space-between;
    box-shadow: 0 0 5px #1213;
    padding: 0;
    overflow: hidden;
}

.message-content {
    padding: 8px;
    border-radius: 8px;
    min-height: 80px;
    background: #cecece;
    color: black !important;
}

.chat-message .card {
    width: 100%;
}

.chat-message.left {
    margin-right: 5%;
}
</style>