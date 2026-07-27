<template>
    <div class="tasks-page container mt-4" v-if="checkPermission(['task_view'])">
        <!-- Header -->
        <div class="card mb-3">
            <div class="card-header d-flex justify-content-between align-items-center">
                <div class="d-flex align-items-center gap-3">
                    <h3 class="mb-0">
                        <i class="bi bi-list-check"></i>
                        <span>مدیریت تسک‌ها</span>
                    </h3>
                    <span class="badge bg-primary">{{ statistics.total_active || 0 }}</span>
                    <span class="badge bg-warning text-dark">{{ statistics.overdue || 0 }}</span>
                </div>
                <button class="btn btn-primary" @click="openCreateModal">
                    <i class="bi bi-plus-lg"></i>
                    <span>تسک جدید</span>
                </button>
            </div>
        </div>

        <!-- Stats Cards -->
        <div class="row mb-3">
            <div class="col-md-2 col-6" v-for="stat in statsCards" :key="stat.key">
                <div class="card stat-card" :class="stat.class" @click="applyStatusFilter(stat.key)">
                    <div class="card-body text-center">
                        <h5 class="stat-number">{{ stat.count }}</h5>
                        <small class="stat-label">{{ stat.label }}</small>
                    </div>
                </div>
            </div>
        </div>

        <!-- Filters -->
        <div class="card mb-3">
            <div class="card-body">
                <div class="row g-2">
                    <div class="col-md-3">
                        <input v-model="filters.search" @input="debounceSearch" type="text" class="form-control"
                            placeholder="جستجوی تسک..." />
                    </div>
                    <div class="col-md-2">
                        <select v-model="filters.status" @change="applyFilters" class="form-select">
                            <option value="">همه وضعیت‌ها</option>
                            <option value="pending">ارجاع شده</option>
                            <option value="in_progress">در حال انجام</option>
                            <option value="completed">انجام شده</option>
                            <option value="waiting_for_another">در انتظار تسک دیگر</option>
                        </select>
                    </div>
                    <div class="col-md-2">
                        <select v-model="filters.priority" @change="applyFilters" class="form-select">
                            <option value="">همه اولویت‌ها</option>
                            <option value="low">کم</option>
                            <option value="medium">متوسط</option>
                            <option value="high">بالا</option>
                            <option value="critical">بحرانی</option>
                        </select>
                    </div>
                    <div class="col-md-2">
                        <select v-model="filters.schedule_type" @change="applyFilters" class="form-select">
                            <option value="">همه زمان‌بندی‌ها</option>
                            <option value="specific_date">تاریخ مشخص</option>
                            <option value="deadline">ددلاین</option>
                            <option value="weekly">هفتگی</option>
                            <option value="monthly">ماهانه</option>
                        </select>
                    </div>
                    <div class="col-md-2">
                        <select v-model="filters.show_closed" @change="applyFilters" class="form-select">
                            <option value="false">فقط باز</option>
                            <option value="true">فقط بسته شده</option>
                            <option value="all">همه تسک‌ها</option>
                        </select>
                    </div>
                    <div class="col-md-1">
                        <button class="btn btn-outline-secondary w-100" @click="resetFilters">
                            <i class="bi bi-arrow-counterclockwise"></i>
                        </button>
                    </div>
                </div>
            </div>
        </div>

        <!-- Tasks Table -->
        <div class="card">
            <div class="card-body">
                <div v-if="loading" class="text-center my-5">
                    <div class="spinner-border" role="status"></div>
                    <p class="mt-2">در حال بارگذاری...</p>
                </div>

                <div v-else-if="tasks.data && tasks.data.length === 0" class="text-center my-5">
                    <i class="bi bi-inbox display-1 text-muted"></i>
                    <p class="mt-3 text-muted">هیچ تسکی یافت نشد</p>
                </div>

                <div v-else>
                    <div class="table-responsive">
                        <table class="table table-hover table-bordered">
                            <thead class="table-light">
                                <tr>
                                    <th style="width: 5%">#</th>
                                    <th style="width: 20%">عنوان</th>
                                    <th style="width: 15%">گیرنده</th>
                                    <th style="width: 15%">پروژه</th>
                                    <th style="width: 12%">وضعیت</th>
                                    <th style="width: 10%">اولویت</th>
                                    <th style="width: 10%">زمان انجام</th>
                                    <th style="width: 13%">عملیات</th>
                                </tr>
                            </thead>
                            <tbody>
                                <tr v-for="task in tasks.data" :key="task.id"
                                    :class="{ 'table-danger': task.is_overdue && !task.is_closed }">
                                    <td>{{ task.id }}</td>
                                    <td>
                                        <div class="task-title-wrapper">
                                            <span class="task-title">{{ task.title }}</span>
                                            <span v-if="task.is_closed" class="badge bg-secondary ms-1">بسته</span>
                                            <span v-if="task.is_overdue && !task.is_closed"
                                                class="badge bg-danger ms-1">دیرکرد</span>
                                        </div>
                                        <small class="text-muted d-block">{{ task.description?.slice(0, 50) || '' }}</small>
                                    </td>
                                    <td>{{ task.assignee?.full_name || '-' }}</td>
                                    <td>{{ task.portfolio?.title || '-' }}</td>
                                    <td>
                                        <span class="badge" :class="getStatusBadge(task.status, task.is_closed)">
                                            {{ task.formatted_status }}
                                        </span>
                                    </td>
                                    <td>
                                        <span class="badge" :class="getPriorityBadge(task.priority)">
                                            {{ getPriorityLabel(task.priority) }}
                                        </span>
                                    </td>
                                    <td>
                                        <small>{{ formatScheduleDate(task) }}</small>
                                        <span v-if="task.schedule_type === 'weekly'" class="badge bg-info ms-1">هفتگی</span>
                                        <span v-if="task.schedule_type === 'monthly'" class="badge bg-info ms-1">ماهانه</span>
                                    </td>
                                    <td>
                                        <div class="btn-group btn-group-sm">
                                            <button class="btn btn-outline-info" @click="openViewModal(task)"
                                                title="جزئیات">
                                                <i class="bi bi-eye"></i>
                                            </button>
                                            <button class="btn btn-outline-warning" @click="openEditModal(task)"
                                                title="ویرایش" v-if="!task.is_closed">
                                                <i class="bi bi-pencil"></i>
                                            </button>
                                            <button class="btn btn-outline-success" @click="openStatusModal(task)"
                                                title="تغییر وضعیت" v-if="!task.is_closed">
                                                <i class="bi bi-arrow-repeat"></i>
                                            </button>
                                            <button class="btn btn-outline-danger" @click="openCloseModal(task)"
                                                title="بستن تسک" v-if="!task.is_closed">
                                                <i class="bi bi-x-circle"></i>
                                            </button>
                                        </div>
                                    </td>
                                </tr>
                            </tbody>
                        </table>
                    </div>

                    <b-pagination v-model="currentPage" :total-rows="tasks.total" v-if="tasks.last_page != 1"
                        :per-page="tasks.per_page" @Update:modelValue="changePage" align="center"
                        class="mt-3"></b-pagination>
                </div>
            </div>
        </div>

        <!-- Modal Create/Edit Task -->
        <ModalComponent :id="'taskModal'" :title="modalTitle" @closeModal="closeModal" v-if="showTaskModal">
            <form @submit.prevent="saveTask">
                <div class="row">
                    <div class="col-md-6">
                        <div class="mb-3">
                            <label class="form-label">عنوان تسک <span class="text-danger">*</span></label>
                            <input v-model="form.title" type="text" class="form-control"
                                :class="{ 'is-invalid': errors.title }" required />
                            <div v-if="errors.title" class="invalid-feedback">{{ errors.title[0] }}</div>
                        </div>
                    </div>
                    <div class="col-md-6">
                        <div class="mb-3">
                            <label class="form-label">گیرنده <span class="text-danger">*</span></label>
                            <select v-model="form.assignee_id" class="form-select"
                                :class="{ 'is-invalid': errors.assignee_id }" required>
                                <option value="">انتخاب گیرنده</option>
                                <option v-for="user in usersList" :key="user.id" :value="user.id">
                                    {{ user.full_name }}
                                </option>
                            </select>
                            <div v-if="errors.assignee_id" class="invalid-feedback">{{ errors.assignee_id[0] }}</div>
                        </div>
                    </div>
                </div>

                <div class="mb-3">
                    <label class="form-label">توضیحات</label>
                    <textarea v-model="form.description" rows="3" class="form-control"
                        :class="{ 'is-invalid': errors.description }"></textarea>
                    <div v-if="errors.description" class="invalid-feedback">{{ errors.description[0] }}</div>
                </div>

                <div class="row">
                    <div class="col-md-6">
                        <div class="mb-3">
                            <label class="form-label">پروژه <span class="text-danger">*</span></label>
                            <select v-model="form.portfolio_id" class="form-select"
                                :class="{ 'is-invalid': errors.portfolio_id }" required>
                                <option value="">انتخاب پروژه</option>
                                <option v-for="portfolio in portfolios" :key="portfolio.id" :value="portfolio.id">
                                    {{ portfolio.title }}
                                </option>
                            </select>
                            <div v-if="errors.portfolio_id" class="invalid-feedback">{{ errors.portfolio_id[0] }}</div>
                        </div>
                    </div>
                    <div class="col-md-6">
                        <div class="mb-3">
                            <label class="form-label">اولویت <span class="text-danger">*</span></label>
                            <select v-model="form.priority" class="form-select"
                                :class="{ 'is-invalid': errors.priority }" required>
                                <option value="low">کم</option>
                                <option value="medium">متوسط</option>
                                <option value="high">بالا</option>
                                <option value="critical">بحرانی</option>
                            </select>
                            <div v-if="errors.priority" class="invalid-feedback">{{ errors.priority[0] }}</div>
                        </div>
                    </div>
                </div>

                <div class="row">
                    <div class="col-md-6">
                        <div class="mb-3">
                            <label class="form-label">نوع زمان‌بندی <span class="text-danger">*</span></label>
                            <select v-model="form.schedule_type" @change="onScheduleTypeChange" class="form-select"
                                :class="{ 'is-invalid': errors.schedule_type }" required>
                                <option value="deadline">ددلاین</option>
                                <option value="specific_date">تاریخ مشخص</option>
                                <option value="weekly">هفتگی (شنبه)</option>
                                <option value="monthly">ماهانه</option>
                            </select>
                            <div v-if="errors.schedule_type" class="invalid-feedback">{{ errors.schedule_type[0] }}</div>
                        </div>
                    </div>
                    <div class="col-md-6">
                        <div class="mb-3" v-if="form.schedule_type === 'deadline'">
                            <label class="form-label">تاریخ ددلاین <span class="text-danger">*</span></label>
                            <input v-model="form.due_date" type="date" class="form-control"
                                :class="{ 'is-invalid': errors.due_date }" />
                            <div v-if="errors.due_date" class="invalid-feedback">{{ errors.due_date[0] }}</div>
                        </div>
                        <div class="mb-3" v-if="form.schedule_type === 'specific_date'">
                            <label class="form-label">تاریخ مشخص <span class="text-danger">*</span></label>
                            <input v-model="form.specific_date" type="date" class="form-control"
                                :class="{ 'is-invalid': errors.specific_date }" />
                            <div v-if="errors.specific_date" class="invalid-feedback">{{ errors.specific_date[0] }}</div>
                        </div>
                        <div class="mb-3" v-if="form.schedule_type === 'monthly'">
                            <label class="form-label">روز ماه <span class="text-danger">*</span></label>
                            <input v-model="form.month_day" type="number" min="1" max="31" class="form-control"
                                :class="{ 'is-invalid': errors.month_day }" placeholder="مثلاً 5" />
                            <div v-if="errors.month_day" class="invalid-feedback">{{ errors.month_day[0] }}</div>
                        </div>
                    </div>
                </div>

                <div class="d-flex justify-content-end gap-2 mt-3">
                    <button type="button" class="btn btn-secondary" @click="closeModal">انصراف</button>
                    <button type="submit" class="btn btn-primary" :disabled="saving">
                        <span v-if="saving" class="spinner-border spinner-border-sm me-1"></span>
                        {{ saving ? 'در حال ذخیره...' : 'ذخیره' }}
                    </button>
                </div>
            </form>
        </ModalComponent>

        <!-- Modal Status Change -->
        <ModalComponent :id="'statusModal'" title="تغییر وضعیت تسک" @closeModal="closeStatusModal" v-if="showStatusModal">
            <form @submit.prevent="updateTaskStatus">
                <div class="mb-3">
                    <label class="form-label">وضعیت جدید <span class="text-danger">*</span></label>
                    <select v-model="statusForm.status" class="form-select" required>
                        <option value="pending">ارجاع شده</option>
                        <option value="in_progress">در حال انجام</option>
                        <option value="completed">انجام شده</option>
                        <option value="waiting_for_another">در انتظار تسک دیگر</option>
                    </select>
                </div>

                <div class="mb-3" v-if="statusForm.status === 'waiting_for_another'">
                    <label class="form-label">تسک مورد انتظار <span class="text-danger">*</span></label>
                    <select v-model="statusForm.waiting_for_task_id" class="form-select" required>
                        <option value="">انتخاب تسک</option>
                        <option v-for="task in pendingTasks" :key="task.id" :value="task.id">
                            {{ task.title }} ({{ task.assignee?.full_name }})
                        </option>
                    </select>
                </div>

                <div class="d-flex justify-content-end gap-2">
                    <button type="button" class="btn btn-secondary" @click="closeStatusModal">انصراف</button>
                    <button type="submit" class="btn btn-success" :disabled="saving">
                        <span v-if="saving" class="spinner-border spinner-border-sm me-1"></span>
                        {{ saving ? 'در حال تغییر...' : 'تغییر وضعیت' }}
                    </button>
                </div>
            </form>
        </ModalComponent>

        <!-- Modal Close Task -->
        <ModalComponent :id="'closeModal'" title="بستن تسک" @closeModal="closeCloseModal" v-if="showCloseModal">
            <form @submit.prevent="closeTask">
                <div class="mb-3">
                    <label class="form-label">دلیل بستن <span class="text-danger">*</span></label>
                    <select v-model="closeForm.reason" class="form-select" required>
                        <option value="">انتخاب دلیل</option>
                        <option value="completed_permanently">تکمیل دائمی</option>
                        <option value="no_longer_needed">نیاز نیست</option>
                        <option value="project_cancelled">لغو پروژه</option>
                        <option value="replaced_by_another">جایگزین با تسک دیگر</option>
                        <option value="client_request">درخواست مشتری</option>
                        <option value="other">سایر</option>
                    </select>
                </div>

                <div class="mb-3">
                    <label class="form-label">توضیحات</label>
                    <textarea v-model="closeForm.description" rows="3" class="form-control"
                        placeholder="توضیحات تکمیلی در مورد بستن تسک..."></textarea>
                </div>

                <div class="alert alert-warning">
                    <i class="bi bi-exclamation-triangle"></i>
                    <span>با بستن این تسک، تسک‌های وابسته به آن نیز غیرفعال خواهند شد.</span>
                </div>

                <div class="d-flex justify-content-end gap-2">
                    <button type="button" class="btn btn-secondary" @click="closeCloseModal">انصراف</button>
                    <button type="submit" class="btn btn-danger" :disabled="saving">
                        <span v-if="saving" class="spinner-border spinner-border-sm me-1"></span>
                        {{ saving ? 'در حال بستن...' : 'بستن تسک' }}
                    </button>
                </div>
            </form>
        </ModalComponent>

        <!-- Modal View Task -->
        <ModalComponent :id="'viewModal'" :title="`جزئیات تسک #${viewingTask?.id || ''}`" @closeModal="closeViewModal" v-if="showViewModal">
            <div v-if="viewingTask">
                <div class="row">
                    <div class="col-md-6">
                        <div class="mb-3">
                            <label class="fw-bold">عنوان:</label>
                            <p>{{ viewingTask.title }}</p>
                        </div>
                        <div class="mb-3">
                            <label class="fw-bold">توضیحات:</label>
                            <p>{{ viewingTask.description || '-' }}</p>
                        </div>
                        <div class="mb-3">
                            <label class="fw-bold">وضعیت:</label>
                            <span class="badge ms-2" :class="getStatusBadge(viewingTask.status, viewingTask.is_closed)">
                                {{ viewingTask.formatted_status }}
                            </span>
                        </div>
                    </div>
                    <div class="col-md-6">
                        <div class="mb-3">
                            <label class="fw-bold">سازنده:</label>
                            <p>{{ viewingTask.creator?.full_name || '-' }}</p>
                        </div>
                        <div class="mb-3">
                            <label class="fw-bold">گیرنده:</label>
                            <p>{{ viewingTask.assignee?.full_name || '-' }}</p>
                        </div>
                        <div class="mb-3">
                            <label class="fw-bold">پروژه:</label>
                            <p>{{ viewingTask.portfolio?.title || '-' }}</p>
                        </div>
                        <div class="mb-3">
                            <label class="fw-bold">اولویت:</label>
                            <span class="badge ms-2" :class="getPriorityBadge(viewingTask.priority)">
                                {{ getPriorityLabel(viewingTask.priority) }}
                            </span>
                        </div>
                    </div>
                </div>

                <div class="row">
                    <div class="col-md-6">
                        <div class="mb-3">
                            <label class="fw-bold">نوع زمان‌بندی:</label>
                            <p>{{ getScheduleTypeLabel(viewingTask.schedule_type) }}</p>
                        </div>
                        <div class="mb-3" v-if="viewingTask.due_date">
                            <label class="fw-bold">ددلاین:</label>
                            <p>{{ formatDate(viewingTask.due_date) }}</p>
                        </div>
                        <div class="mb-3" v-if="viewingTask.specific_date">
                            <label class="fw-bold">تاریخ مشخص:</label>
                            <p>{{ formatDate(viewingTask.specific_date) }}</p>
                        </div>
                    </div>
                    <div class="col-md-6">
                        <div class="mb-3" v-if="viewingTask.is_closed">
                            <label class="fw-bold text-danger">بسته شده:</label>
                            <p>توسط: {{ viewingTask.closed_by?.full_name || '-' }}</p>
                            <p>دلیل: {{ viewingTask.close_reason_label || '-' }}</p>
                            <p>توضیحات: {{ viewingTask.close_description || '-' }}</p>
                        </div>
                        <div class="mb-3">
                            <label class="fw-bold">تاریخ ایجاد:</label>
                            <p>{{ formatDate(viewingTask.created_at) }}</p>
                        </div>
                        <div class="mb-3" v-if="viewingTask.completed_at">
                            <label class="fw-bold">تاریخ تکمیل:</label>
                            <p>{{ formatDate(viewingTask.completed_at) }}</p>
                        </div>
                    </div>
                </div>
            </div>

            <div class="d-flex justify-content-end">
                <button type="button" class="btn btn-secondary" @click="closeViewModal">بستن</button>
            </div>
        </ModalComponent>
    </div>
    <div v-else class="container mt-4">
        <div class="alert alert-danger">
            <i class="bi bi-lock"></i>
            <span>شما دسترسی لازم برای مشاهده این صفحه را ندارید.</span>
        </div>
    </div>
</template>

<script setup>
import { ref, reactive, onMounted, computed } from "vue";
import axios from "axios";
import Swal from "sweetalert2";
import { useRoute, useRouter } from "vue-router";
import { useAdmin } from '@/stores/modules/admin';
import { debounce } from 'lodash';
import ModalComponent from '@/components/shared/modal.vue';

const store = useAdmin();
const checkPermission = store.checkPermission;
const router = useRouter();
const route = useRoute();

// Refs
const tasks = ref({ data: [] });
const loading = ref(false);
const saving = ref(false);
const currentPage = ref(1);
const usersList = ref([]);
const portfolios = ref([]);
const pendingTasks = ref([]);
const viewingTask = ref(null);

// Filters
const filters = reactive({
    search: '',
    status: '',
    priority: '',
    schedule_type: '',
    show_closed: 'false'
});

// Statistics
const statistics = ref({
    total: 0,
    total_active: 0,
    total_closed: 0,
    pending: 0,
    in_progress: 0,
    completed: 0,
    waiting: 0,
    overdue: 0
});

// Stats Cards
const statsCards = computed(() => [
    { key: 'all', label: 'همه', count: statistics.value.total, class: 'bg-light' },
    { key: 'pending', label: 'ارجاع شده', count: statistics.value.pending, class: 'bg-warning text-dark' },
    { key: 'in_progress', label: 'در حال انجام', count: statistics.value.in_progress, class: 'bg-info text-white' },
    { key: 'completed', label: 'انجام شده', count: statistics.value.completed, class: 'bg-success text-white' },
    { key: 'waiting', label: 'در انتظار', count: statistics.value.waiting, class: 'bg-secondary text-white' },
    { key: 'overdue', label: 'دیرکرد', count: statistics.value.overdue, class: 'bg-danger text-white' },
]);

// Modals
const showTaskModal = ref(false);
const showStatusModal = ref(false);
const showCloseModal = ref(false);
const showViewModal = ref(false);
const modalTitle = ref('تسک جدید');
const editingTask = ref(null);

// Forms
const form = reactive({
    title: '',
    description: '',
    creator_id: null,
    assignee_id: '',
    portfolio_id: '',
    priority: 'medium',
    schedule_type: 'deadline',
    due_date: '',
    specific_date: '',
    month_day: '',
    week_day: 'saturday'
});

const statusForm = reactive({
    task_id: null,
    status: '',
    waiting_for_task_id: ''
});

const closeForm = reactive({
    task_id: null,
    reason: '',
    description: ''
});

const errors = reactive({});

// Methods
const getTasks = async (page = 1) => {
    loading.value = true;
    try {
        const params = {
            page,
            search: filters.search,
            status: filters.status || undefined,
            priority: filters.priority || undefined,
            schedule_type: filters.schedule_type || undefined,
            is_closed: filters.show_closed === 'all' ? undefined : filters.show_closed
        };

        const response = await axios.get('/tasks', { params });
        tasks.value = response.data.data;
        currentPage.value = response.data.data.current_page;
    } catch (error) {
        console.error('خطا در دریافت تسک‌ها:', error);
        Swal.fire('خطا', 'مشکل در دریافت تسک‌ها', 'error');
    } finally {
        loading.value = false;
    }
};

const getStatistics = async () => {
    try {
        const response = await axios.get('/tasks/statistics', {
            params: { assignee_id: store.user?.id }
        });
        statistics.value = response.data.data;
    } catch (error) {
        console.error('خطا در دریافت آمار:', error);
    }
};

const getUsers = async () => {
    try {
        const response = await axios.get('/users', { params: { per_page: 100 } });
        usersList.value = response.data.data || [];
    } catch (error) {
        console.error('خطا در دریافت کاربران:', error);
    }
};

const getPortfolios = async () => {
    try {
        const response = await axios.get('/portfolios', { params: { per_page: 100 } });
        portfolios.value = response.data.data || [];
    } catch (error) {
        console.error('خطا در دریافت پروژه‌ها:', error);
    }
};

const getPendingTasks = async () => {
    try {
        const response = await axios.get('/tasks', {
            params: { status: 'pending', is_closed: 'false', per_page: 100 }
        });
        pendingTasks.value = response.data.data || [];
    } catch (error) {
        console.error('خطا در دریافت تسک‌های در انتظار:', error);
    }
};

const applyFilters = () => {
    getTasks(1);
};

const debounceSearch = debounce(() => {
    getTasks(1);
}, 500);

const applyStatusFilter = (status) => {
    if (status === 'all') {
        filters.status = '';
        filters.show_closed = 'false';
    } else if (status === 'overdue') {
        getOverdueTasks();
        return;
    } else {
        filters.status = status;
        filters.show_closed = 'false';
    }
    getTasks(1);
};

const getOverdueTasks = async () => {
    loading.value = true;
    try {
        const response = await axios.get('/tasks/overdue');
        tasks.value = { data: response.data.data, total: response.data.data.length };
        currentPage.value = 1;
    } catch (error) {
        console.error('خطا در دریافت تسک‌های دیرکرد:', error);
    } finally {
        loading.value = false;
    }
};

const resetFilters = () => {
    filters.search = '';
    filters.status = '';
    filters.priority = '';
    filters.schedule_type = '';
    filters.show_closed = 'false';
    getTasks(1);
};

const changePage = (page) => {
    if (page) {
        router.replace({ name: route.name, query: { page: page } });
        getTasks(page);
    }
};

// Modal Handlers
const openCreateModal = () => {
    modalTitle.value = 'تسک جدید';
    editingTask.value = null;
    resetForm();
    errors.value = {};
    showTaskModal.value = true;
};

const openEditModal = (task) => {
    modalTitle.value = 'ویرایش تسک';
    editingTask.value = task;
    Object.assign(form, {
        title: task.title,
        description: task.description || '',
        assignee_id: task.assignee_id,
        portfolio_id: task.portfolio_id,
        priority: task.priority,
        schedule_type: task.schedule_type,
        due_date: task.due_date || '',
        specific_date: task.specific_date || '',
        month_day: task.month_day || '',
    });
    errors.value = {};
    showTaskModal.value = true;
};

const openViewModal = (task) => {
    viewingTask.value = task;
    showViewModal.value = true;
};

const openStatusModal = (task) => {
    statusForm.task_id = task.id;
    statusForm.status = task.status;
    statusForm.waiting_for_task_id = task.waiting_for_task_id || '';
    getPendingTasks();
    showStatusModal.value = true;
};

const openCloseModal = (task) => {
    closeForm.task_id = task.id;
    closeForm.reason = '';
    closeForm.description = '';
    showCloseModal.value = true;
};

const closeModal = () => {
    showTaskModal.value = false;
    resetForm();
};

const closeStatusModal = () => {
    showStatusModal.value = false;
    statusForm.task_id = null;
};

const closeCloseModal = () => {
    showCloseModal.value = false;
    closeForm.task_id = null;
};

const closeViewModal = () => {
    showViewModal.value = false;
    viewingTask.value = null;
};

const resetForm = () => {
    Object.assign(form, {
        title: '',
        description: '',
        creator_id: store.user?.id || null,
        assignee_id: '',
        portfolio_id: '',
        priority: 'medium',
        schedule_type: 'deadline',
        due_date: '',
        specific_date: '',
        month_day: '',
        week_day: 'saturday'
    });
};

const onScheduleTypeChange = () => {
    form.due_date = '';
    form.specific_date = '';
    form.month_day = '';
};

// Save Task
const saveTask = async () => {
    saving.value = true;
    errors.value = {};

    try {
        const data = { ...form };
        data.creator_id = store.user?.id;

        // حذف فیلدهای اضافی بر اساس نوع زمان‌بندی
        if (data.schedule_type !== 'deadline') delete data.due_date;
        if (data.schedule_type !== 'specific_date') delete data.specific_date;
        if (data.schedule_type !== 'monthly') delete data.month_day;

        let response;
        if (editingTask.value) {
            response = await axios.put(`/tasks/${editingTask.value.id}`, data);
            Swal.fire('موفق', 'تسک با موفقیت ویرایش شد', 'success');
        } else {
            response = await axios.post('/tasks', data);
            Swal.fire('موفق', 'تسک با موفقیت ایجاد شد', 'success');
        }

        showTaskModal.value = false;
        resetForm();
        getTasks(currentPage.value);
        getStatistics();
    } catch (error) {
        if (error.response?.status === 422) {
            errors.value = error.response.data.errors;
        } else {
            Swal.fire('خطا', error.response?.data?.message || 'مشکل در ذخیره تسک', 'error');
        }
    } finally {
        saving.value = false;
    }
};

// Update Status
const updateTaskStatus = async () => {
    saving.value = true;
    try {
        const data = {
            status: statusForm.status,
            waiting_for_task_id: statusForm.waiting_for_task_id || undefined
        };

        await axios.patch(`/tasks/${statusForm.task_id}/status`, data);
        Swal.fire('موفق', 'وضعیت تسک با موفقیت تغییر یافت', 'success');

        showStatusModal.value = false;
        getTasks(currentPage.value);
        getStatistics();
    } catch (error) {
        Swal.fire('خطا', error.response?.data?.message || 'مشکل در تغییر وضعیت', 'error');
    } finally {
        saving.value = false;
    }
};

// Close Task
const closeTask = async () => {
    saving.value = true;
    try {
        await axios.post(`/tasks/${closeForm.task_id}/close`, {
            reason: closeForm.reason,
            description: closeForm.description
        });

        Swal.fire('موفق', 'تسک با موفقیت بسته شد', 'success');

        showCloseModal.value = false;
        getTasks(currentPage.value);
        getStatistics();
    } catch (error) {
        Swal.fire('خطا', error.response?.data?.message || 'مشکل در بستن تسک', 'error');
    } finally {
        saving.value = false;
    }
};

// Utility Functions
const getStatusBadge = (status, isClosed) => {
    if (isClosed) return 'bg-secondary';
    const map = {
        pending: 'bg-warning text-dark',
        in_progress: 'bg-info text-white',
        completed: 'bg-success text-white',
        waiting_for_another: 'bg-secondary text-white'
    };
    return map[status] || 'bg-secondary';
};

const getPriorityBadge = (priority) => {
    const map = {
        low: 'bg-success text-white',
        medium: 'bg-warning text-dark',
        high: 'bg-orange text-white',
        critical: 'bg-danger text-white'
    };
    return map[priority] || 'bg-secondary';
};

const getPriorityLabel = (priority) => {
    const map = {
        low: 'کم',
        medium: 'متوسط',
        high: 'بالا',
        critical: 'بحرانی'
    };
    return map[priority] || priority;
};

const getScheduleTypeLabel = (type) => {
    const map = {
        specific_date: 'تاریخ مشخص',
        deadline: 'ددلاین',
        weekly: 'هفتگی',
        monthly: 'ماهانه'
    };
    return map[type] || type;
};

const formatScheduleDate = (task) => {
    if (task.is_closed) return 'بسته شده';
    if (task.schedule_type === 'deadline' && task.due_date) {
        return formatDate(task.due_date);
    }
    if (task.schedule_type === 'specific_date' && task.specific_date) {
        return formatDate(task.specific_date);
    }
    if (task.schedule_type === 'weekly') {
        return 'هر شنبه';
    }
    if (task.schedule_type === 'monthly' && task.month_day) {
        return `روز ${task.month_day} هر ماه`;
    }
    return '-';
};

const formatDate = (date) => {
    if (!date) return '-';
    const d = new Date(date);
    return d.toLocaleDateString('fa-IR');
};

// Lifecycle
onMounted(() => {
    currentPage.value = route.query.page ?? 1;
    Promise.all([
        getTasks(currentPage.value),
        getStatistics(),
        getUsers(),
        getPortfolios()
    ]);
});
</script>

<style scoped>
.tasks-page {
    direction: rtl;
}

.stat-card {
    cursor: pointer;
    transition: all 0.3s ease;
}

.stat-card:hover {
    transform: translateY(-3px);
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.stat-number {
    font-size: 1.5rem;
    font-weight: bold;
    margin: 0;
}

.stat-label {
    font-size: 0.8rem;
    opacity: 0.8;
}

.task-title-wrapper {
    display: flex;
    align-items: center;
    gap: 4px;
    flex-wrap: wrap;
}

.task-title {
    font-weight: 500;
}

.btn-orange {
    background-color: #ff8c00;
    border-color: #ff8c00;
    color: white;
}

.btn-orange:hover {
    background-color: #e67e00;
    border-color: #e67e00;
    color: white;
}

.bg-orange {
    background-color: #ff8c00;
}

.table-hover tbody tr:hover {
    background-color: rgba(0, 123, 255, 0.05);
}

.table-danger {
    background-color: #f8d7da !important;
}

@media (max-width: 768px) {
    .stat-number {
        font-size: 1.2rem;
    }
    
    .btn-group .btn {
        padding: 0.25rem 0.4rem;
    }
}
</style>