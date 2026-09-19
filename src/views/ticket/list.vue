<template>
  <div id="ticket-page" class="container mt-3 mt-md-4 px-2 px-md-3 py-3 py-md-4">

    <!-- ================= خلاصه ================= -->
    <div class="summary-grid mb-4">
      <div class="summary-card">
        <div class="summary-icon bg-primary-subtle text-primary">
          <i class="bi bi-ticket-fill"></i>
        </div>
        <div>
          <div class="summary-title">کل تیکت‌ها</div>
          <div class="summary-value text-primary">
            {{ mainData.data ? mainData.data.total : 0 }}
          </div>
        </div>
      </div>

      <div class="summary-card">
        <div class="summary-icon bg-warning-subtle text-warning">
          <i class="bi bi-exclamation-circle-fill"></i>
        </div>
        <div>
          <div class="summary-title">تیکت‌های باز</div>
          <div class="summary-value text-warning">{{ mainData.openedTotal ?? 0 }}</div>
        </div>
      </div>

      <div class="summary-card">
        <div class="summary-icon bg-secondary-subtle text-secondary">
          <i class="bi bi-check-circle-fill"></i>
        </div>
        <div>
          <div class="summary-title">تیکت‌های بسته</div>
          <div class="summary-value text-secondary">{{ mainData.closedTotal ?? 0 }}</div>
        </div>
      </div>

      <div class="summary-card summary-action">
        <router-link
          to="/tickets/add"
          v-if="checkPermission(['ticket_store'])"
          class="btn btn-success w-100"
        >
          <i class="bi bi-plus-circle-fill me-1"></i>
          افزودن تیکت جدید
        </router-link>
      </div>
    </div>

    <!-- ================= فیلتر ================= -->
    <div class="card mb-4">
      <div class="card-header fw-bold">
        <i class="bi bi-funnel me-1"></i>
        فیلتر تیکت‌ها
      </div>
      <div class="card-body">
        <form @submit.prevent="applyFilters()" class="row g-3">
          <div class="col-12 col-sm-6 col-md-4">
            <label class="form-label">وضعیت</label>
            <select class="form-select" v-model="filters.status">
              <option value="">همه</option>
              <option value="closed">بسته شده</option>
              <option value="answered">پاسخ داده شده</option>
              <option value="pending">در انتظار بررسی</option>
              <option value="awaiting_payment">در انتظار پرداخت</option>
              <option value="awaiting_response">در انتظار پاسخ کارفرما</option>
              <option value="referred">ارجاع شده</option>
            </select>
          </div>

          <div class="col-12 col-sm-6 col-md-4">
            <label class="form-label">از تاریخ</label>
            <date-picker
              display-format="jYYYY/jMM/jDD"
              placeholder="از تاریخ"
              format="YYYY-MM-DD"
              v-model="filters.dateFrom"
            />
          </div>

          <div class="col-12 col-sm-6 col-md-4">
            <label class="form-label">تا تاریخ</label>
            <date-picker
              display-format="jYYYY/jMM/jDD"
              placeholder="تا تاریخ"
              format="YYYY-MM-DD"
              v-model="filters.dateTo"
            />
          </div>

          <div class="col-12 d-flex flex-column flex-sm-row justify-content-end gap-2 filter-actions">
            <button type="button" class="btn btn-outline-secondary" @click="resetFilters">
              <i class="bi bi-x-circle me-1"></i>
              حذف فیلتر
            </button>
            <button type="button" class="btn btn-primary" @click="applyFilters()">
              <i class="bi bi-search me-1"></i>
              اعمال فیلتر
            </button>
          </div>
        </form>
      </div>
    </div>

    <!-- ================= جدول ================= -->
    <div class="card">
      <div class="card-header d-flex flex-column flex-sm-row align-items-stretch align-items-sm-center justify-content-between gap-2">
        <span class="fw-bold">
          <i class="bi bi-list-ul me-1"></i>
          لیست تیکت‌ها
        </span>
        <b-pagination
          v-if="mainData.data && mainData.data.total > mainData.data.per_page"
          v-model="mainData.data.current_page"
          :total-rows="mainData.data.total"
          :per-page="mainData.data.per_page"
          @update:modelValue="changePage"
          size="sm"
          class="mb-0 pagination-responsive"
        />
      </div>

      <div class="card-body p-2 p-md-0">
        <!-- بارگذاری -->
        <div v-if="loader" class="text-center py-5 text-muted">
          <div class="spinner-border spinner-border-sm me-2"></div>
          در حال بارگذاری...
        </div>

        <!-- خالی -->
        <div
          v-else-if="mainData.data && !mainData.data.data.length"
          class="text-center py-5 text-muted"
        >
          <i class="bi bi-inbox fs-1 d-block mb-2"></i>
          تیکتی برای نمایش وجود ندارد.
        </div>

        <!-- ===== نمایش داده‌ها (جدول دسکتاپ + کارت موبایل) ===== -->
        <div v-else-if="mainData.data">

          <!-- جدول فقط دسکتاپ -->
          <div class="table-responsive d-none d-md-block">
            <table class="table table-hover align-middle mb-0">
              <thead class="table-light">
                <tr>
                  <th>شناسه</th>
                  <th>موضوع</th>
                  <th>مشتری</th>
                  <th>در دست اقدام</th>
                  <th>وضعیت</th>
                  <th>تاریخ ایجاد</th>
                  <th class="text-end">جزئیات</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="item in mainData.data.data" :key="item.id">
                  <td class="fw-bold">#{{ item.id }}</td>
                  <td class="text-truncate" style="max-width: 220px;">{{ item.title }}</td>
                  <td>{{ item.sender ? item.sender.full_name : '-' }}</td>
                  <td>{{ item.doer ? item.doer.full_name : '-' }}</td>
                  <td>
                    <span :class="getStatusBadgeClass(item.status)">
                      {{ translater(item.status) }}
                    </span>
                  </td>
                  <td>{{ new Date(item.created_at).toLocaleDateString('fa') }}</td>
                  <td class="text-end">
                    <router-link class="btn btn-sm btn-primary" :to="`/tickets/${item.id}`">
                      <i class="bi bi-eye me-1"></i>
                      مشاهده
                    </router-link>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>

          <!-- کارت‌ها فقط موبایل -->
          <div class="d-md-none ticket-cards p-2">
            <div
              v-for="item in mainData.data.data"
              :key="item.id"
              class="ticket-card"
            >
              <div class="ticket-card-header">
                <div class="ticket-id-badge">#{{ item.id }}</div>
                <div class="ticket-title">{{ item.title }}</div>
              </div>

              <div class="ticket-card-body">
                <div class="ticket-info-row">
                  <i class="bi bi-person"></i>
                  <span class="info-label">مشتری:</span>
                  <span class="info-value">{{ item.sender ? item.sender.full_name : '-' }}</span>
                </div>
                <div class="ticket-info-row">
                  <i class="bi bi-person-badge"></i>
                  <span class="info-label">در دست اقدام:</span>
                  <span class="info-value">{{ item.doer ? item.doer.full_name : '-' }}</span>
                </div>
                <div class="ticket-info-row">
                  <i class="bi bi-calendar"></i>
                  <span class="info-label">تاریخ:</span>
                  <span class="info-value">{{ new Date(item.created_at).toLocaleDateString('fa') }}</span>
                </div>
              </div>

              <div class="ticket-card-footer">
                <span :class="getStatusBadgeClass(item.status)">
                  {{ translater(item.status) }}
                </span>
                <router-link class="btn btn-sm btn-primary" :to="`/tickets/${item.id}`">
                  <i class="bi bi-eye me-1"></i>
                  مشاهده
                </router-link>
              </div>
            </div>
          </div>

        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import axios from 'axios';
import { ref, onMounted, reactive } from 'vue';
import { useRoute, useRouter } from 'vue-router';
import { useAdmin } from '@/stores/modules/admin';

const store = useAdmin();
const checkPermission = store.checkPermission;

const route = useRoute();
const router = useRouter();

const filters = reactive({
  dateFrom: '',
  status: '',
  dateTo: '',
});

const loader = ref(false);
const mainData = ref({});

const getStatusBadgeClass = (status) => {
  switch (status) {
    case 'answered': return 'badge bg-warning text-dark';
    case 'closed': return 'badge bg-secondary';
    case 'pending': return 'badge bg-info text-dark';
    case 'awaiting_payment': return 'badge bg-danger';
    case 'awaiting_response': return 'badge bg-danger';
    case 'referred': return 'badge bg-primary';
    default: return 'badge bg-light text-dark';
  }
};

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

onMounted(() => {
  filters.status = route.query.status || '';
  filters.dateFrom = route.query.dateFrom || '';
  filters.dateTo = route.query.dateTo || '';
  getData(route.query);
});

function changePage(selectedPage) {
  if (selectedPage) applyFilters(selectedPage);
  else getData({});
}

async function getData(query = {}) {
  loader.value = true;
  try {
    const { data } = await axios.get('/tickets', { params: { ...query } });
    mainData.value = data;
  } catch (error) {
    console.log(error);
  } finally {
    loader.value = false;
  }
}

const applyFilters = (page) => {
  const query = {
    status: filters.status,
    dateTo: filters.dateTo,
    dateFrom: filters.dateFrom,
    page: page ?? 1,
  };
  router.replace({ name: route.name, query });
  getData(query);
};

function resetFilters() {
  filters.dateFrom = '';
  filters.dateTo = '';
  filters.status = '';
  router.replace({ name: route.name });
  getData({});
}
</script>

<style scoped>
@import url("https://cdn.jsdelivr.net/npm/bootstrap-icons@1.10.2/font/bootstrap-icons.css");

#ticket-page {
  direction: rtl;
}

/* ================= خلاصه ================= */
.summary-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 12px;
}
.summary-card {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 14px 16px;
  background: #fff;
  border: 1px solid #eef0f2;
  border-radius: 12px;
  box-shadow: 0 6px 14px rgba(0, 0, 0, 0.04);
  transition: transform 0.15s ease, box-shadow 0.15s ease;
}
.summary-card:hover {
  transform: translateY(-3px);
  box-shadow: 0 10px 22px rgba(0, 0, 0, 0.07);
}
.summary-action {
  padding: 12px;
  justify-content: center;
}
.summary-icon {
  width: 44px;
  height: 44px;
  border-radius: 50%;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  font-size: 20px;
  flex-shrink: 0;
}
.summary-title {
  font-size: 0.8rem;
  color: #6c757d;
}
.summary-value {
  font-size: 1.15rem;
  font-weight: 700;
}

/* ================= جدول ================= */
.table thead th {
  font-size: 0.85rem;
  color: #495057;
  white-space: nowrap;
}
.table tbody td {
  vertical-align: middle;
}

/* ================= صفحه‌بندی ================= */
:deep(.page-item.active .page-link) {
  background-color: #0d6efd;
  border-color: #0d6efd;
  color: #fff;
}
:deep(.page-link) {
  cursor: pointer;
}
.pagination-responsive {
  flex-wrap: wrap;
  justify-content: center;
}

/* ================= کارت‌های موبایل ================= */
.ticket-cards {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.ticket-card {
  background: #fff;
  border: 1px solid #e9ecef;
  border-radius: 12px;
  padding: 14px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.04);
  transition: all 0.2s ease;
}

.ticket-card:hover {
  box-shadow: 0 6px 18px rgba(0, 0, 0, 0.08);
  transform: translateY(-2px);
}

.ticket-card-header {
  display: flex;
  align-items: center;
  gap: 10px;
  padding-bottom: 10px;
  border-bottom: 1px solid #f0f0f0;
  margin-bottom: 10px;
}

.ticket-id-badge {
  background: linear-gradient(135deg, #6c5ce7, #a29bfe);
  color: white;
  font-size: 0.75rem;
  font-weight: 700;
  padding: 4px 10px;
  border-radius: 20px;
  flex-shrink: 0;
}

.ticket-title {
  font-weight: 700;
  color: #2d3436;
  font-size: 0.95rem;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.ticket-card-body {
  display: flex;
  flex-direction: column;
  gap: 8px;
  margin-bottom: 12px;
}

.ticket-info-row {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 0.85rem;
}

.ticket-info-row i {
  color: #6c5ce7;
  font-size: 0.95rem;
  width: 18px;
  text-align: center;
  flex-shrink: 0;
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

.ticket-card-footer {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
  padding-top: 10px;
  border-top: 1px solid #f0f0f0;
}

.ticket-card-footer .btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 4px;
  font-size: 0.75rem;
  padding: 6px 10px;
  white-space: nowrap;
}

/* ================= موبایل ================= */
@media (max-width: 992px) {
  .summary-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}
@media (max-width: 576px) {
  .summary-grid {
    grid-template-columns: 1fr;
  }

  .filter-actions {
    width: 100%;
  }

  .filter-actions .btn {
    flex: 1;
  }
}
</style>