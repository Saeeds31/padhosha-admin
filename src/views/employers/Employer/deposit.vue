<template>
  <div class="container mt-4" v-if="checkPermission(['employer_deposit'])">

    <div v-if="mainData" class="card mb-2">
      <div class="card-header d-flex justify-content-between align-items-center mb-3">
        <h3>اطلاعات پرداختی
          {{ employer ? `- ${employer.bussines_label} (${employer.user.full_name})` : '' }}
        </h3>
      </div>
      <div class="card-body">
        <form>
          <div class="row g-3">
            <div class="col-md-3">
              <label for="filter-cost" class="form-label">مبلغ (حداقل)</label>
              <input type="number" class="form-control" id="filter-cost" v-model.number="filters.amount">
            </div>

            <div class="col-md-3">
              <label for="filter-cost" class="form-label">انتخاب وضعیت</label>
              <select name="" class="form-control" v-model="filters.status" id="">
                <option value="accepted">پذیرفته شده</option>
                <option value="rejected">رد شده</option>
                <option value="pending">در انتظار تایید</option>
              </select>
            </div>
            <div class="col-md-3">
              <label for="filter-date-from" class="form-label">از تاریخ</label>
              <date-picker display-format="jYYYY/jMM/jDD" placeholder="از تاریخ" format="YYYY-MM-DD"
                v-model="filters.dateFrom"></date-picker>
            </div>
            <div class="col-md-3">
              <label for="filter-date-to" class="form-label">تا تاریخ</label>
              <date-picker display-format="jYYYY/jMM/jDD" placeholder="تا تاریخ" format="YYYY-MM-DD"
                v-model="filters.dateTo"></date-picker>
            </div>
            <div class="col-12 text-end">
              <button type="button" class="btn btn-secondary me-2" @click="setFilter()">اعمال فیلتر</button>
              <button type="button" class="btn btn-outline-secondary" @click="resetFilters()">حذف فیلتر</button>
            </div>
          </div>
        </form>
      </div>
    </div>


    <div class="card">
      <div class="card-header d-flex justify-content-between align-items-center">
        <span>لیست اطلاعات پرداختی</span>
        <!-- کنترل‌های صفحه‌بندی -->
        <b-pagination v-if="mainData.data" v-model="mainData.data.current_page" :total-rows="mainData.data.total"
          :per-page="mainData.data.per_page" @Update:modelValue="changePage" align="center" class="mt-3"></b-pagination>

      </div>
      <div v-if="mainData.data" class="card-body">
        <div class="table-responsive">
          <table class="table table-striped table-hover">
            <thead>
              <tr>
                <th>شناسه رسید</th>
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
                <td>{{ item.title }}</td>
                <td>{{ formatCurrency(item.amount) }} تومان</td>
                <td>{{ item.description }}</td>
                <td>
                  <span :class="getStatusClass(item.status)">{{ translater(item.status) }}</span>
                </td>
                <td>{{ new Date(item.created_at).toLocaleDateString('fa') }}</td>
                <td>
                  <button class="btn btn-success" v-if="item.status != 'accepted'"
                    @click="changeStatus('accepted', item)">تایید
                    رسید</button>

                  <button class="btn btn-danger me-2" v-if="item.status != 'rejected'"
                    @click="changeStatus('rejected', item)">رد
                    رسید</button>
                </td>
              </tr>
            </tbody>
          </table>
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
        <span>
          ثبت وضعیت
        </span>
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
.table th,
.table td {
  vertical-align: middle;
}
</style>