<template>
  <div class="container mt-3 mt-md-4 px-2 px-md-3" v-if="checkPermission(['employer_deposit'])">
    <div v-if="mainData" class="card mb-2 header-card">
      <div class="card-header">
        <div class="d-flex flex-column flex-sm-row justify-content-between align-items-stretch align-items-sm-center gap-2 mb-3">
          <h3 class="mb-0 page-title">
            <span>اطلاعات پرداختی
              {{ employer ? `- ${employer.bussines_label} (${employer.user.full_name})` : '' }}
            </span>
          </h3>
          <router-link to="/employers/reciepts/add" class="btn btn-success add-btn">
            <i class="bi bi-plus-circle-fill me-2"></i>
            <span>افزودن رسید مشتری</span>
          </router-link>
        </div>
      </div>
      <div class="card-body">
        <form>
          <div class="row g-3">
            <div class="col-12 col-sm-6 col-md-3">
              <label for="filter-cost" class="form-label">مبلغ (حداقل)</label>
              <input type="number" class="form-control" id="filter-cost" v-model.number="filters.amount">
            </div>

            <div class="col-12 col-sm-6 col-md-3">
              <label for="filter-cost" class="form-label">انتخاب وضعیت</label>
              <select name="" class="form-control" v-model="filters.status" id="">
                <option value="accepted">پذیرفته شده</option>
                <option value="rejected">رد شده</option>
                <option value="pending">در انتظار تایید</option>
              </select>
            </div>
            <div class="col-12 col-sm-6 col-md-3">
              <label for="filter-date-from" class="form-label">از تاریخ</label>
              <date-picker display-format="jYYYY/jMM/jDD" placeholder="از تاریخ" format="YYYY-MM-DD"
                v-model="filters.dateFrom"></date-picker>
            </div>
            <div class="col-12 col-sm-6 col-md-3">
              <label for="filter-date-to" class="form-label">تا تاریخ</label>
              <date-picker display-format="jYYYY/jMM/jDD" placeholder="تا تاریخ" format="YYYY-MM-DD"
                v-model="filters.dateTo"></date-picker>
            </div>
            <div class="col-12 text-end filter-actions">
              <button type="button" class="btn btn-secondary me-2" @click="setFilter()">اعمال فیلتر</button>
              <button type="button" class="btn btn-outline-secondary" @click="resetFilters()">حذف فیلتر</button>
            </div>
          </div>
        </form>
      </div>
    </div>


    <div class="card">
      <div class="card-header">
        <div class="d-flex flex-column flex-sm-row justify-content-between align-items-center gap-2">
          <span>لیست اطلاعات پرداختی</span>
          <!-- کنترل‌های صفحه‌بندی -->
          <b-pagination v-if="mainData.data" v-model="mainData.data.current_page" :total-rows="mainData.data.total"
            :per-page="mainData.data.per_page" @Update:modelValue="changePage" align="center"
            class="mt-3 mt-sm-0 pagination-responsive"></b-pagination>
        </div>
      </div>
      <div v-if="mainData.data" class="card-body p-2 p-md-3">
        <!-- ===== نمایش جدول در دسکتاپ ===== -->
        <div class="table-responsive d-none d-md-block">
          <table class="table table-striped table-hover mb-0">
            <thead>
              <tr>
                <th>شناسه رسید</th>
                <th>کارفرما</th>
                <th>عنوان</th>
                <th>قیمت</th>
                <th>توضیحات</th>
                <th>وضعیت</th>
                <th>تاریخ</th>
                <th>عملیات</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(item, index) in mainData.data.data" :key="index">
                <td>{{ item.id }}</td>
                <td>{{ item.employer?.bussines_label }}</td>
                <td>{{ item.title }}</td>
                <td>{{ formatCurrency(item.amount) }} تومان</td>
                <td>{{ item.description }}</td>
                <td>
                  <span :class="getStatusClass(item.status)">{{ translater(item.status) }}</span>
                </td>
                <td>{{ new Date(item.created_at).toLocaleDateString('fa') }}</td>
                <td>
                  <div class="d-flex flex-wrap gap-1">
                    <button class="btn btn-success btn-sm" v-if="item.status != 'accepted'"
                      @click="changeStatus('accepted', item)">تایید رسید</button>

                    <button class="btn btn-danger btn-sm" v-if="item.status != 'rejected'"
                      @click="changeStatus('rejected', item)">رد رسید</button>
                  </div>
                </td>
              </tr>
            </tbody>
          </table>
        </div>

        <!-- ===== نمایش کارتی در موبایل ===== -->
        <div class="d-md-none receipt-cards">
          <div
            v-for="(item, index) in mainData.data.data"
            :key="index"
            class="receipt-card"
          >
            <div class="receipt-card-header">
              <div class="receipt-id-badge">#{{ item.id }}</div>
              <div class="receipt-title">{{ item.title }}</div>
              <span :class="getStatusClass(item.status)">{{ translater(item.status) }}</span>
            </div>

            <div class="receipt-card-body">
              <div class="receipt-info-row">
                <i class="bi bi-briefcase"></i>
                <span class="info-label">کارفرما:</span>
                <span class="info-value">{{ item.employer?.bussines_label ?? '-' }}</span>
              </div>
              <div class="receipt-info-row">
                <i class="bi bi-cash-stack"></i>
                <span class="info-label">قیمت:</span>
                <span class="info-value">{{ formatCurrency(item.amount) }} تومان</span>
              </div>
              <div class="receipt-info-row" v-if="item.description">
                <i class="bi bi-text-paragraph"></i>
                <span class="info-label">توضیحات:</span>
                <span class="info-value">{{ item.description }}</span>
              </div>
              <div class="receipt-info-row">
                <i class="bi bi-calendar"></i>
                <span class="info-label">تاریخ:</span>
                <span class="info-value">{{ new Date(item.created_at).toLocaleDateString('fa') }}</span>
              </div>
            </div>

            <div class="receipt-card-actions">
              <button class="btn btn-success btn-sm flex-fill" v-if="item.status != 'accepted'"
                @click="changeStatus('accepted', item)">
                <i class="bi bi-check-circle"></i>
                <span>تایید</span>
              </button>

              <button class="btn btn-danger btn-sm flex-fill" v-if="item.status != 'rejected'"
                @click="changeStatus('rejected', item)">
                <i class="bi bi-x-circle"></i>
                <span>رد</span>
              </button>
            </div>
          </div>

          <!-- حالت خالی -->
          <div v-if="!mainData.data.data || mainData.data.data.length === 0" class="text-center py-5 text-muted">
            <i class="bi bi-inbox fs-1 d-block mb-2"></i>
            <p>رسیدی یافت نشد</p>
          </div>
        </div>
      </div>
    </div>
    <Modal v-if="statusModal" id="statusModal" @closeModal="() => statusModal = false" title="تغییر وضعیت رسید">
      <b-row>
        <b-col cols="12" md="12">
          <b-form-group :label="`دلیل ${selectedStatus == 'rejected' ? 'رد' : 'تایید'} رسید`" label-for="content">
            <b-form-textarea id="content" v-model="admin_note" />
          </b-form-group>
        </b-col>

      </b-row>
      <button :disabled="loading" @click="saveStatus()" class="btn btn-success">
        <i class="bi bi-save"></i>
        <span>ثبت وضعیت</span>
      </button>
    </Modal>

  </div>
</template>

<script setup>
import { ref, onMounted, reactive } from "vue";
import axios from "axios";
import Swal from "sweetalert2";
import Modal from "@/components/shared/modal.vue";
import { useAdmin } from '@/stores/modules/admin';
import { useRoute, useRouter } from "vue-router";
const store = useAdmin();
const route = useRoute();
const router = useRouter();
const checkPermission = store.checkPermission;
const error = reactive({});
const loading = ref(false);
let currentUrl = `employer-deposit`;
let statusModal = ref(false);
const formatCurrency = (amount) => {
  if (typeof amount !== 'number') return '0';
  return amount.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ',');
};
let subsmodal = ref(false);

let filters = ref({
  amount: '',
  status: '',
  dateTo: '',
  dateFrom: ''
})

let mainData = ref({})
let admin_note = ref('');
const getStatusClass = (status) => {
  switch (status) {
    case 'accepted': return 'badge bg-success';
    case 'rejected': return 'badge bg-danger';
    case 'pending': return 'badge bg-warning text-dark';
    default: return 'badge bg-secondary';
  }
};
const translater = (status) => {
  switch (status) {
    case 'accepted': return 'تایید شده';
    case 'rejected': return 'رد شده';
    case 'pending': return 'در انتظار';
  }
};
function resetFilters() {
  filters.value.amount = null;
  filters.value.dateFrom = '';
  filters.value.status = '';
  filters.value.dateTo = '';
  router.replace({ name: route.name })
  getReciepts({})
};
const setFilter = (page) => {
  router.replace({ name: route.name, query: { amount: filters.value.amount, dateTo: filters.value.dateTo, dateFrom: filters.value.dateFrom, status: filters.value.status, page: page ?? 1 } })
  getReciepts({ amount: filters.value.amount, dateTo: filters.value.dateTo, dateFrom: filters.value.dateFrom, status: filters.value.status, page: page ?? 1 })
};

async function getReciepts(query = {}) {
  loading.value = true;
  try {
    const { data } = await axios.get(`/employer-deposit`, {
      params: {
        ...query,
        employer_id: route.params.id
      }
    });
    mainData.value = data;
  } catch (err) {
    console.error(err);
  } finally {
    loading.value = false;
  }
};
async function saveStatus() {
  loading.value = true;
  try {
    const fd = new FormData();
    fd.append('status', selectedStatus.value);
    fd.append('admin_note', admin_note.value);
    fd.append('_method', "PUT");
    const { data } = await axios.post(`/employer-deposit/${selectedItem.value.id}`, fd);
    getReciepts(currentUrl)
    Swal.fire("موفق", "با موفقیت ثبت شد", "success");
    statusModal.value = false;
    admin_note.value = "";
  } catch (err) {
    console.error(err);
  } finally {
    loading.value = false;
  }
}
let selectedItem = ref('');
let selectedStatus = ref('');
function changeStatus(status, item) {
  selectedStatus.value = status;
  selectedItem.value = item;
  admin_note.value = item.admin_note;
  statusModal.value = true;
}




function changePage(selectedPage) {
  if (selectedPage) {
    applyFilters(selectedPage)
  }
  else getReciepts({})
}
let employer = ref();
onMounted(() => {
  if (route.params.id) {
    axios.get(`employers/${route.params.id}`).then((res) => {
      employer.value = res.data.data;
    });
  }
  getReciepts(route.query);
});
</script>

<style scoped>
/* ===== هدر صفحه ===== */
.header-card .card-header {
  padding: 16px 20px;
  background: transparent;
  border-bottom: 2px solid #f8f9fa;
}

.page-title {
  font-weight: 700;
  color: #2d3436;
  font-size: 1.5rem;
  display: flex;
  align-items: center;
  gap: 8px;
  flex-wrap: wrap;
}

.add-btn {
  white-space: nowrap;
  display: inline-flex;
  align-items: center;
  gap: 6px;
  justify-content: center;
}

.filter-actions {
  display: flex;
  justify-content: flex-end;
  gap: 8px;
  flex-wrap: wrap;
}

/* ===== جدول ===== */
.table {
  margin-bottom: 0;
}

.table th,
.table td {
  vertical-align: middle;
}

.table thead th {
  background: #f8f9fa;
  font-weight: 600;
  color: #2d3436;
  white-space: nowrap;
  font-size: 0.9rem;
}

.table tbody td {
  font-size: 0.9rem;
}

/* ===== کارت‌های موبایل ===== */
.receipt-cards {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.receipt-card {
  background: #fff;
  border: 1px solid #e9ecef;
  border-radius: 12px;
  padding: 14px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.04);
  transition: all 0.2s ease;
}

.receipt-card:hover {
  box-shadow: 0 6px 18px rgba(0, 0, 0, 0.08);
  transform: translateY(-2px);
}

.receipt-card-header {
  display: flex;
  align-items: center;
  gap: 8px;
  padding-bottom: 10px;
  border-bottom: 1px solid #f0f0f0;
  margin-bottom: 10px;
  flex-wrap: wrap;
}

.receipt-id-badge {
  background: linear-gradient(135deg, #6c5ce7, #a29bfe);
  color: white;
  font-size: 0.75rem;
  font-weight: 700;
  padding: 4px 10px;
  border-radius: 20px;
  flex-shrink: 0;
}

.receipt-title {
  font-weight: 700;
  color: #2d3436;
  font-size: 0.95rem;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  flex: 1;
  min-width: 0;
}

.receipt-card-body {
  display: flex;
  flex-direction: column;
  gap: 8px;
  margin-bottom: 12px;
}

.receipt-info-row {
  display: flex;
  align-items: flex-start;
  gap: 8px;
  font-size: 0.85rem;
}

.receipt-info-row i {
  color: #6c5ce7;
  font-size: 0.95rem;
  width: 18px;
  text-align: center;
  flex-shrink: 0;
  margin-top: 2px;
}

.info-label {
  color: #6c757d;
  flex-shrink: 0;
}

.info-value {
  color: #2d3436;
  font-weight: 600;
  margin-right: auto;
  word-break: break-word;
  text-align: left;
}

.receipt-card-actions {
  display: flex;
  gap: 6px;
  padding-top: 10px;
  border-top: 1px solid #f0f0f0;
}

.receipt-card-actions .btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 4px;
  font-size: 0.75rem;
  padding: 6px 8px;
  white-space: nowrap;
}

/* ===== Pagination ===== */
.pagination-responsive {
  flex-wrap: wrap;
  justify-content: center;
}

/* ========================================= */
/* ===== موبایل (کمتر از 768px) ===== */
/* ========================================= */
@media (max-width: 767.98px) {
  .header-card .card-header {
    padding: 12px 14px;
  }

  .header-card .card-body {
    padding: 12px 14px;
  }

  .page-title {
    font-size: 1.1rem;
    justify-content: center;
    text-align: center;
    width: 100%;
  }

  .add-btn {
    width: 100%;
  }

  .filter-actions {
    justify-content: stretch;
  }

  .filter-actions .btn {
    flex: 1;
  }

  .card-header .d-flex.flex-sm-row {
    flex-direction: column;
  }
}

/* ========================================= */
/* ===== موبایل کوچک (کمتر از 400px) ===== */
/* ========================================= */
@media (max-width: 399.98px) {
  .page-title {
    font-size: 0.95rem;
  }

  .receipt-card {
    padding: 12px;
  }

  .receipt-title {
    font-size: 0.85rem;
  }

  .receipt-info-row {
    font-size: 0.78rem;
  }

  .receipt-card-actions .btn {
    font-size: 0.7rem;
    padding: 5px 6px;
  }
}

/* ========================================= */
/* ===== دسکتاپ: مخفی کردن کارت‌ها ===== */
/* ========================================= */
@media (min-width: 768px) {
  .receipt-cards {
    display: none;
  }
}
</style>