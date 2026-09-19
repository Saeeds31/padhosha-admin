<template>
    <b-container fluid class="ticket-page py-4">
        <section v-if="mainData">

            <!-- ================= ارجاع و تغییر وضعیت ================= -->
            <div class="row g-3 mb-3">
                <div class="col-lg-6">
                    <div class="card action-card h-100">
                        <div class="card-header">
                            <h6 class="mb-0">
                                <i class="bi bi-person-workspace me-1"></i>
                                ارجاع تیکت به کارشناس
                            </h6>
                        </div>
                        <div class="card-body">
                            <form @submit.prevent="sendReferd" class="row g-3">
                                <div class="col-12">
                                    <label class="form-label">انتخاب کاربر</label>
                                    <multiselect @search-change="loadUsers" v-model="selectedUser"
                                        placeholder="انتخاب کاربر" open-direction="bottom" :options="options"
                                        label="label" track-by="id" :searchable="true" :multiple="false"
                                        :close-on-select="true" :show-labels="false">
                                        <template slot="noOptions">جستجو کنید</template>
                                        <template slot="noResult">
                                            <span v-if="isRequesting">در حال جستجو...</span>
                                            <span v-else>موردی یافت نشد</span>
                                        </template>
                                    </multiselect>
                                </div>
                                <div class="col-12">
                                    <button type="submit" class="btn btn-primary w-100" :disabled="loading">
                                        <i class="bi bi-save2 me-1"></i>
                                        {{ loading ? 'در حال ذخیره...' : 'ارجاع به کارشناس' }}
                                    </button>
                                </div>
                            </form>
                        </div>
                    </div>
                </div>

                <div class="col-lg-6">
                    <div class="card action-card h-100">
                        <div class="card-header">
                            <h6 class="mb-0">
                                <i class="bi bi-arrow-repeat me-1"></i>
                                تغییر وضعیت تیکت
                            </h6>
                        </div>
                        <div class="card-body">
                            <form @submit.prevent="changeStatus" class="row g-3">
                                <div class="col-12">
                                    <label class="form-label">انتخاب وضعیت</label>
                                    <select class="form-select" v-model="status">
                                        <option value="closed">بسته شده</option>
                                        <option value="awaiting_payment">در انتظار پرداخت</option>
                                        <option value="awaiting_response">در انتظار پاسخ کارفرما</option>
                                    </select>
                                </div>
                                <div class="col-12">
                                    <button type="submit" class="btn btn-primary w-100" :disabled="loading">
                                        <i class="bi bi-save2 me-1"></i>
                                        {{ loading ? 'در حال ذخیره...' : 'تغییر وضعیت' }}
                                    </button>
                                </div>
                            </form>
                        </div>
                    </div>
                </div>
            </div>

            <!-- ================= هدر تیکت ================= -->
            <div class="card mb-3">
                <div class="card-header d-flex flex-wrap align-items-center justify-content-between gap-2">
                    <span class="fw-bold">تیکت شماره: {{ mainData.ticket.id }}</span>
                    <span class="text-muted small">
                        آخرین بروزرسانی:
                        {{ mainData.messages.length ? new Date(mainData.messages[0].created_at).toLocaleDateString('fa')
                            : '-' }}
                    </span>
                </div>
                <div class="card-body">
                    <div class="row g-3">
                        <div class="col-md-4">
                            <div class="stat-card">
                                <div class="stat-icon bg-success text-white"><i class="fas fa-exclamation-circle"></i>
                                </div>
                                <div>
                                    <div class="stat-title">وضعیت تیکت</div>
                                    <div class="stat-value text-success">{{ translater(mainData.ticket.status) }}</div>
                                </div>
                            </div>
                        </div>
                        <div class="col-md-4">
                            <div class="stat-card">
                                <div class="stat-icon bg-dark text-white"><i class="fas fa-user-cog"></i></div>
                                <div>
                                    <div class="stat-title">در دست اقدام کارشناس</div>
                                    <div class="stat-value text-dark">
                                        {{ mainData.ticket.doer ? mainData.ticket.doer.full_name : '-' }}
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div class="col-md-4">
                            <div class="stat-card">
                                <div class="stat-icon bg-primary text-white"><i class="fas fa-coins"></i></div>
                                <div>
                                    <div class="stat-title">هزینه تیکت</div>
                                    <div class="stat-value text-primary">
                                        {{ mainData.cost
                                            ? Number(mainData.cost.amount).toLocaleString('fa') + ' تومان'
                                            : 'بدون هزینه' }}
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>

            <!-- ================= ارسال پاسخ ================= -->
            <div class="card mb-3">
                <div class="card-header">
                    <h6 class="mb-0">
                        <i class="bi bi-chat-dots me-1"></i>
                        ارسال پاسخ جدید
                    </h6>
                </div>
                <div class="card-body">
                    <template v-if="mainData.ticket.status !== 'closed'">
                        <b-form @submit.prevent="sendMessage">

                            <!-- پاسخ‌های پیشنهادی -->
                            <div class="mb-3">
                                <label class="form-label">پاسخ‌های پیشنهادی</label>
                                <div class="quick-replies">
                                    <span v-for="(q, i) in quickReplies" :key="i" class="quick-reply-chip"
                                        :title="q.text" @click="applyQuickReply(q)">
                                        {{ q.title }}
                                    </span>
                                </div>
                            </div>

                            <div class="mb-3">
                                <label class="form-label">پاسخ کامل</label>
                                <Editor v-model="form.content" />
                            </div>

                            <div class="mb-3">
                                <label class="form-label">فایل پیوست</label>
                                <VueFileAgent @select="imageLoaded" :maxFiles="1"
                                    accept=".pdf,.jpg,.png,.webp,.docx,.xlsx" theme="grid" deletable sortable />
                            </div>

                            <b-button :disabled="loader" variant="primary" type="submit">
                                <i class="bi bi-send me-1"></i>
                                ارسال پاسخ
                            </b-button>
                        </b-form>
                    </template>

                    <p v-else class="alert alert-danger mb-0">
                        برای تیکت در وضعیت بسته شده امکان ارسال پاسخ وجود ندارد.
                    </p>
                </div>
            </div>

            <!-- ================= گفتگو ================= -->
            <div class="card">
                <div class="card-header">
                    <h6 class="mb-0">
                        <i class="bi bi-chat-square-text me-1"></i>
                        موضوع تیکت: {{ mainData.ticket.title }}
                    </h6>
                </div>
                <div class="card-body">
                    <div class="chat-container">
                        <transition-group name="fade" tag="div">
                            <div v-for="msg in mainData.messages" :key="msg.id"
                                :class="['chat-message', msg.sender_side == 'employer' ? 'right' : 'left']">
                                <b-card class="p-3" :class="msg.sender_side == 'employer'
                                    ? 'bg-primary text-white'
                                    : 'bg-white border shadow-sm'">
                                    <span :class="msg.sender_side == 'employer' ? 'text-white' : 'text-muted'"
                                        class="small">
                                        {{ msg.sender ? msg.sender.full_name : '' }}
                                    </span>
                                    <hr class="my-2" />
                                    <div v-html="msg.message" class="message-content"></div>

                                    <div class="attachments mt-2">
                                        <template v-if="msg.attachment">
                                            <b-link :href="fileHandler(msg.attachment)"
                                                :class="msg.sender_side == 'employer' ? 'text-white' : 'text-muted'"
                                                target="_blank">
                                                <i class="bi bi-paperclip me-1"></i>
                                                مشاهده فایل الصاق شده
                                            </b-link>
                                        </template>
                                        <template v-if="msg.voice">
                                            <audio controls :src="fileHandler(msg.voice)" class="mt-1 w-100"></audio>
                                        </template>
                                    </div>

                                    <small :class="msg.sender_side == 'employer' ? 'text-white' : 'text-muted'"
                                        class="d-block mt-2 text-end">
                                        تاریخ: {{ new Date(msg.created_at).toLocaleDateString('fa') }}
                                    </small>
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
import Swal from 'sweetalert2';
import { useRoute, useRouter } from 'vue-router';
import { toast } from 'vue3-toastify';
import 'vue3-toastify/dist/index.css';

const route = useRoute();
const router = useRouter();
const mainData = ref(null);
const status = ref('');
const loading = ref(false);
const loader = ref(false);

/* ================= ارجاع ================= */
async function sendReferd() {
    const fd = new FormData();
    fd.append('user_id', selectedUser.value ? selectedUser.value.id : '');
    loading.value = true;
    try {
        await axios.post(`/tickets/${route.params.id}/referd`, fd);
        Swal.fire('موفق', 'ارجاع به کارشناس موفقیت آمیز بود', 'success');
        router.push('/tickets');
    } catch (error) {
        toast.error(error.response?.data?.message);
    } finally {
        loading.value = false;
    }
}

/* ================= تغییر وضعیت ================= */
async function changeStatus() {
    const fd = new FormData();
    fd.append('status', status.value);
    loading.value = true;
    try {
        await axios.post(`/tickets/${route.params.id}/status`, fd);
        toast.success('وضعیت تیکت با موفقیت تغییر کرد');
        getTicket();
    } catch (error) {
        toast.error(error.response?.data?.message);
    } finally {
        loading.value = false;
    }
}

/* ================= دریافت تیکت ================= */
async function getTicket() {
    const { data } = await axios.get(`/tickets/${route.params.id}`);
    mainData.value = data.data;
}

/* ================= جستجوی کاربران ================= */
let abortController = null;
const options = ref([]);
const selectedUser = ref(null);
const isRequesting = ref(false);

const loadUsers = async (searchQuery = '') => {
    if (abortController) abortController.abort();
    abortController = new AbortController();
    isRequesting.value = true;
    try {
        const { data } = await axios.get('/supporter?search=' + searchQuery, {
            signal: abortController.signal,
        });
        options.value = data.data.map(u => ({
            id: u.id,
            label: `${u.full_name} (${u.mobile})`,
        }));
    } catch (error) {
        if (!axios.isCancel(error)) toast.error('خطا در جستجوی کاربران');
    } finally {
        isRequesting.value = false;
    }
};

/* ================= ارسال پیام ================= */
const form = reactive({ file: '', content: '' });

function imageLoaded(files) {
    form.file = files[0]?.file || '';
}

async function sendMessage() {
    if (!form.content || form.content.length < 3) {
        return Swal.fire('خطا', 'پیام خود را به صورت کامل وارد کنید', 'warning');
    }
    loader.value = true;
    const fd = new FormData();
    fd.append('ticket_id', mainData.value.ticket.id);
    fd.append('file', form.file ?? '');
    fd.append('voice', '');
    fd.append('message', form.content);

    try {
        await axios.post('/messages', fd);
        Swal.fire('موفق', 'پیام شما با موفقیت ارسال شد', 'success');
        form.content = '';
        form.file = '';
        getTicket();
    } catch (error) {
        Swal.fire('خطا', 'خطایی در ارسال پیش آمده است', 'error');
    } finally {
        loader.value = false;
    }
}

/* ================= پاسخ‌های پیشنهادی ================= */
/* عنوان کوتاه در چیپ نمایش داده می‌شود و متن کامل در ادیتور درج می‌شود */
const quickReplies = ref([
    {
        title: 'درخواست اطلاعات بیشتر',
        text: 'با سلام و احترام , لطفا برای بررسی دقیق‌تر درخواست شما،  اطلاعات بیشتری در خصوص موضوع تیکت ارسال کنید\n واحد پشتیبانی پدیده هوشمند شاهکوه - پدهوشا.',
    },
    {
        title: 'در حال بررسی',
        text: 'با سلام، درخواست شما در حال بررسی است و به‌زودی نتیجه به شما اطلاع داده خواهد شد\n واحد پشتیبانی پدیده هوشمند شاهکوه - پدهوشا.',
    },
    {
        title: 'برطرف شد',
        text: 'با سلام و عرض احترام مورد مطرح‌شده برطرف شد، لطفاً بررسی کنید و در صورت وجود مشکل اطلاع دهید\n واحد پشتیبانی پدیده هوشمند شاهکوه - پدهوشا.',
    },
    {
        title: 'انجام شد',
        text: 'با سلام و عرض احترام مورد مطرح‌شده به سایت اضافه شد، لطفاً بررسی کنید و در صورت وجود مشکل نتیجه را از طریق همین تیکت اطلاع دهید\n واحد پشتیبانی پدیده هوشمند شاهکوه - پدهوشا.',
    },
    {
        title: 'نیاز به پرداخت',
        text: 'با سلام و احترام درخواست شما نیاز به پرداخت هزینه مجزا از مبلغ پشتیبانی دارد و در زمره توسعه سایت محسوب میشود.در صورت تایید شما مبلغ برآورد و از طریق تیکت اطلاع رسانی خواهد شد\n واحد پشتیبانی پدیده هوشمند شاهکوه - پدهوشا.',
    },
]);

function applyQuickReply(q) {
    const current = form.content || '';
    const html = `<p>${q.text}</p>`;
    form.content = html;
}

/* ================= متفرقه ================= */
onMounted(() => {
    getTicket();
});

function fileHandler(path) {
    return window.baseImageAddress + path;
}

const translater = (status) => {
    switch (status) {
        case 'answered': return 'پاسخ داده شده';
        case 'closed': return 'بسته شده';
        case 'pending': return 'در انتظار بررسی';
        case 'awaiting_payment': return 'در انتظار پرداخت';
        case 'awaiting_response': return 'در انتظار پاسخ کارفرما';
        case 'referred': return 'ارجاع شده';
        default: return status;
    }
};
</script>

<style scoped>
.ticket-page {
    direction: rtl;
    max-width: 1200px;
    margin: auto;
}

/* کارت‌های بالایی */
.action-card .card-header {
    background: #f8f9fa;
    border-bottom: 1px solid #eef0f2;
}

/* کارت آماری */
.stat-card {
    display: flex;
    align-items: center;
    gap: 12px;
    padding: 14px;
    border-radius: 12px;
    background: #fff;
    border: 1px solid #eef0f2;
    box-shadow: 0 6px 14px rgba(0, 0, 0, 0.04);
    transition: transform 0.15s ease, box-shadow 0.15s ease;
    height: 100%;
}

.stat-card:hover {
    transform: translateY(-3px);
    box-shadow: 0 10px 22px rgba(0, 0, 0, 0.07);
}

.stat-icon {
    width: 42px;
    height: 42px;
    border-radius: 50%;
    display: inline-flex;
    align-items: center;
    justify-content: center;
    font-size: 18px;
    flex-shrink: 0;
}

.stat-title {
    font-size: 0.8rem;
    color: #6c757d;
}

.stat-value {
    font-size: 1.05rem;
    font-weight: 600;
    margin-top: 2px;
}

/* پاسخ‌های پیشنهادی */
.quick-replies {
    display: flex;
    flex-wrap: wrap;
    gap: 8px;
}

.quick-reply-chip {
    display: inline-flex;
    align-items: center;
    background: #eef4ff;
    color: #2b4d9c;
    border: 1px solid #d6e2ff;
    padding: 6px 12px;
    border-radius: 20px;
    font-size: 0.85rem;
    cursor: pointer;
    transition: background 0.15s ease;
}

.quick-reply-chip:hover {
    background: #dde9ff;
}

/* چت */
.chat-container {
    display: flex;
    flex-direction: column-reverse;
    gap: 8px;
}

.chat-message {
    display: flex;
}

.chat-message.left {
    justify-content: flex-start;
}

.chat-message.right {
    justify-content: flex-end;
}

.chat-message .card {
    max-width: 70%;
    border-radius: 14px;
    overflow-x: auto;
}

.message-content {
    padding: 8px 10px;
    border-radius: 8px;
    background: #f1f3f5;
    color: #111;
    min-height: 40px;
    word-break: break-word;
}

.chat-message.right .message-content {
    background: rgba(255, 255, 255, 0.15);
    color: #fff;
}

/* انیمیشن */
.fade-enter-active,
.fade-leave-active {
    transition: all 0.3s ease;
}

.fade-enter-from {
    opacity: 0;
    transform: translateY(10px);
}

/* موبایل */
@media (max-width: 768px) {
    .chat-message .card {
        max-width: 90%;
    }
}
</style>