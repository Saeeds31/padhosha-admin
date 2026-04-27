<template>
  <div id="ticket-page" class="container mt-4">
    <!-- بخش خلاصه تیکت‌ها -->
    <div class="row mb-4 align-items-center card-summary p-3">
      <div class="col-md-3 text-center">
        <!-- آیکن برای کل تیکت‌ها -->
        <h5 class=""><i class="bi bi-ticket-fill me-2 text-primary"></i> کل تیکت‌ها: <b class="text-primary"
            v-if="mainData.data">{{
              mainData.data.total }}</b></h5>
      </div>
      <div class="col-md-3 text-center">
        <!-- آیکن برای تیکت‌های باز -->
        <h5><i class="bi bi-exclamation-circle-fill me-2 text-warning"></i> تیکت‌های باز: <b class="text-warning">{{
          mainData.openedTotal }}</b></h5>
      </div>
      <div class="col-md-3 text-center">
        <!-- آیکن برای تیکت‌های بسته -->
        <h5><i class="bi bi-check-circle-fill me-2 text-secondary"></i> تیکت‌های بسته: <b class="text-secondary">{{
          mainData.closedTotal }}</b></h5>
      </div>
      <div class="col-md-3 text-end">
        <!-- دکمه افزودن تیکت با آیکن -->
        <router-link to="/tickets/add" v-if="checkPermission(['ticket_store'])" class="btn btn-success">
          <i class="bi bi-plus-circle-fill me-2"></i> افزودن تیکت جدید
        </router-link>
      </div>
    </div>

    <div class="card mb-4">
      <div class="card-header">
        فیلتر تیکت ها
      </div>
      <div class="card-body">
        <form @submit.prevent="applyFilters()">
          <div class="row g-3">
            <div class="col-md-4">
              <label for="filter-cost" class="form-label">وضعیت</label>
              <select class="form-control" v-model="filters.status">
                <option value="">همه</option>
                <option value="closed">بسته شده</option>
                <option value="answered">پاسخ داده شده</option>
                <option value="pending">در انتظار بررسی</option>
                <option value="awaiting_payment">در انتظار پرداخت</option>
                <option value="referred">ارجاع شده</option>
              </select>
            </div>
            <div class="col-md-4">
              <label for="filter-date-from" class="form-label">از تاریخ</label>
              <date-picker display-format="jYYYY/jMM/jDD" placeholder="از تاریخ" format="YYYY-MM-DD"
                v-model="filters.dateFrom"></date-picker>
            </div>
            <div class="col-md-4">
              <label for="filter-date-to" class="form-label">تا تاریخ</label>
              <date-picker display-format="jYYYY/jMM/jDD" placeholder="تا تاریخ" format="YYYY-MM-DD"
                v-model="filters.dateTo"></date-picker>
            </div>
            <div class="col-12 text-end">
              <button type="button" class="btn btn-secondary me-2" @click="applyFilters()">اعمال فیلتر</button>
              <button type="button" class="btn btn-outline-secondary" @click="resetFilters()">حذف فیلتر</button>
            </div>
          </div>
        </form>
      </div>
    </div>
    <!-- جدول نمایش تیکت‌ها -->
    <div class="card">
      <b-pagination v-if="mainData.data" v-model="mainData.data.current_page" :total-rows="mainData.data.total"
        :per-page="mainData.data.per_page" @Update:modelValue="changePage" align="center" class="mt-3"></b-pagination>

      <div class="card-body">
        <div class="table-responsive">
          <table class="table table-striped table-hover">
            <thead>
              <tr>
                <th>شناسه تیکت</th>
                <th>موضوع تیکت</th>
                <th>مشتری</th>
                <th>در دست اقدام</th>
                <th>وضعیت</th>
                <th>تاریخ ایجاد</th>
                <th>جزئیات</th>
              </tr>
            </thead>
            <tbody v-if="mainData.data">
              <tr v-for="(item, index) in mainData.data.data" :key="index">
                <td>{{ item.id }}</td>
                <td>{{ item.title }}</td>
                <td>{{ item.sender ? item.sender.full_name : "-" }}</td>
                <td>{{ item.doer ? item.doer.full_name : "-" }}</td>
                <td>
                  <span :class="getStatusBadgeClass(item.status)">{{ translater(item.status) }}</span>
                </td>
                <td>{{ new Date(item.created_at).toLocaleDateString('fa') }}</td>
                <td>
                  <router-link class="btn btn-primary" :to="`/tickets/${item.id}`">
                    مشاهده بیشتر
                  </router-link>
                </td>
              </tr>
            </tbody>
          </table>
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
const route = useRoute()
const router = useRouter()
const filters = reactive({
  dateFrom: '',
  status: '',
  dateTo: '',
});
let loader = ref(false)
const getStatusBadgeClass = (status) => {
  switch (status) {
    case 'answered': return 'badge bg-warning text-dark';
    case 'closed': return 'badge bg-secondary';
    case 'pending': return 'badge bg-info text-dark';
    case 'awaiting_payment': return 'badge bg-danger';
    default: return 'badge bg-primary';
  }
};
const translater = (status) => {
  switch (status) {
    case 'answered': return 'پاسخ داده شده';
    case 'closed': return 'بسته شده';
    case 'pending': return 'در انتظار بررسی';
    case 'awaiting_payment': return 'در انتظار پرداخت';
    case 'referred': return 'ارجاع شده';
  }
};
let mainData = ref({})
onMounted(() => {
  getData(route.query);
  filters.status = route.query.status;
})
function changePage(selectedPage) {
  if (selectedPage) {
    applyFilters(selectedPage)
  }
  else getData({})
}
async function getData(query = {}) {
  loader.value = true;
  try {
    const { data } = await axios.get('/tickets', {
      params: {
        ...query
      }
    })
    mainData.value = data;
  } catch (error) {
    console.log(error)
  } finally {
    loader.value = false;
  }


}
const applyFilters = (page) => {
  router.replace({ name: route.name, query: { status: filters.status, dateTo: filters.dateTo, dateFrom: filters.dateFrom, page: page ?? 1 } })
  getData({ status: filters.status, dateTo: filters.dateTo, dateFrom: filters.dateFrom, page: page ?? 1 })
};

function resetFilters() {
  filters.dateFrom = '';
  filters.dateTo = '';
  filters.status = '';
  router.replace({ name: route.name })
  getData({})
};

</script>

<style scoped>
@import url("https://cdn.jsdelivr.net/npm/bootstrap-icons@1.10.2/font/bootstrap-icons.css");
/* برای استفاده از آیکن‌های Bootstrap */

.card-summary {
  background-color: #f8f9fa;
  border-radius: 0.5rem;
  border: 1px solid #dee2e6;
}

.card-summary h5 {
  margin-bottom: 0.5rem;
  display: flex;
  /* برای هم‌ترازی آیکن و متن */
  align-items: center;
  /* برای هم‌ترازی عمودی */
  justify-content: center;
  /* برای وسط چین کردن در صورت نیاز */
}

.card-summary h5 i {
  font-size: 1.2em;
  /* اندازه آیکن */
}

.table-responsive {
  overflow-x: auto;
}

/* استایل صفحه بندی */
.page-item.active .page-link {
  background-color: #0d6efd;
  border-color: #0d6efd;
  color: white;
}

.page-link {
  cursor: pointer;
}
</style>
