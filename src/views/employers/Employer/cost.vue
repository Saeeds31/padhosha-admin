<template>
  <div class="container mt-4" v-if="checkPermission(['employer_cost'])">

    <div v-if="mainData" class="card mb-2">
      <div class="card-header d-flex justify-content-between align-items-center mb-3">
        <h3 v-if="mainData.employer">
          <i class="bi bi-book-half"></i>
          <span>مدیریت هزینه های -{{ mainData.employer.bussines_label }} </span>
        </h3>
        <span>جمع بدهی به تومان :{{ formatCurrency(Number(mainData.total)) }}</span>
        <button @click="showModal()" class="btn btn-success">
          <i class="bi bi-plus"></i>
          <span>
            افزودن هزینه
          </span>
        </button>
      </div>
    </div>


    <div class="card">
      <div class="card-header d-flex justify-content-between align-items-center">
        <span>لیست اطلاعات مالی</span>
        <!-- کنترل‌های صفحه‌بندی -->
        <b-pagination v-if="mainData.data" v-model="mainData.data.current_page" :total-rows="mainData.data.total"
          :per-page="mainData.data.per_page" @Update:modelValue="changePage" align="center" class="mt-3"></b-pagination>

      </div>
      <div v-if="mainData.data" class="card-body">
        <div class="table-responsive">
          <table class="table table-striped table-hover">
            <thead>
              <tr>
                <th>شناسه هزینه</th>
                <th>عنوان</th>
                <th>قیمت</th>
                <th>توضیحات</th>
                <th>تاریخ</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(item, index) in mainData.data.data" :key="index">
                <td>{{ item.id }}</td>
                <td>{{ item.title }}</td>
                <td>{{ formatCurrency(item.amount) }} تومان</td>
                <td>{{ item.description }}</td>
                <td>{{ new Date(item.created_at).toLocaleDateString('fa') }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
    <Modal v-if="costModal" id="costModal" @closeModal="() => costModal = false" title="هزینه مشتری">
      <b-row>

        <b-col cols="12" md="12">
          <b-form-group label="عنوان هزینه" label-for="content">
            <b-form-input v-model="cost.title" />
            <small v-if="error.title" class="text-danger">{{ error.title[0] }}</small>
          </b-form-group>
        </b-col>
        <b-col cols="12" md="12">
          <b-form-group label="توضیحات" label-for="content">
            <b-form-textarea id="content" v-model="cost.description" />
            <small v-if="error.description" class="text-danger">{{ error.description[0] }}</small>
          </b-form-group>
        </b-col>
        <b-col>
          <b-form-group label="نوع هزینه" label-for="content">
            <select name="" v-model="cost.costable_type" class="form-control" id="">
              <option value="Support">قرارداد پشتیبانی</option>
              <option value="Kavengar">کاوه نگار</option>
              <option value="Ticket">تیکت</option>
            </select>
            <small v-if="error.costable_type" class="text-danger">{{ error.costable_type[0] }}</small>

          </b-form-group>
        </b-col>
        <b-col v-if="cost.costable_type == 'Ticket'" cols="12" md="12">
          <b-form-group label="شماره تیکت" label-for="content">
            <b-form-input type="number" v-model="cost.costable_id"></b-form-input>
          </b-form-group>
        </b-col>
        <b-col cols="12" md="12">
          <b-form-group :label="`مبلغ به تومان : ${Number(cost.amount).toLocaleString('fa')}`" label-for="content">
            <b-form-input type="number" v-model="cost.amount"></b-form-input>
            <small v-if="error.amount" class="text-danger">{{ error.amount[0] }}</small>
          </b-form-group>
        </b-col>
      </b-row>
      <button :disabled="loading" @click="saveCost()" class="btn btn-success">
        <i class="bi bi-save"></i>
        <span>
          ثبت هزینه
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
import { useRoute } from "vue-router";
const store = useAdmin();
const route = useRoute();
const checkPermission = store.checkPermission;
const error = reactive({});
const loading = ref(false);
let currentUrl = `employer-cost/${route.params.id}`;
let costModal = ref(false);
const formatCurrency = (amount) => {
  if (typeof amount !== 'number') return '0';
  return amount.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ',');
};
let subsmodal = ref(false);
let selectedUser = ref(null);
let form = ref({
  'start_date': null,
  'expiration_date': null,
  'level_type': '',
})
let cost = ref({
  'title': '',
  'description': '',
  'amount': '',
  'costable_id': '1',
  'costable_type': '',
  'employer_id': route.params.id
})
let mainData = ref({})
let content = ref('');
async function getCost(url) {
  loading.value = true;
  try {
    const { data } = await axios.get(url);
    mainData.value = data;
  } catch (err) {
    console.error(err);
  } finally {
    loading.value = false;
  }
};
async function saveCost() {
  loading.value = true;
  try {
    const fd = new FormData();
    for (const key in cost.value) {
      fd.append(key, cost.value[key]);
    }
    const { data } = await axios.post("/employers-cost", fd);
    getCost(currentUrl)
    Swal.fire("موفق", "با موفقیت ثبت شد", "success");
    costModal.value = false;
    cost.value = {
      'title': '',
      'description': '',
      'amount': '',
      'costable_id': '1',
      'costable_type': '',
      'employer_id': route.params.id
    }
  } catch (err) {
    console.error(err);
    if (err.response?.status === 422) {
      Object.assign(error, err.response.data.errors)
    }
  } finally {
    loading.value = false;
  }
}
function showModal() {
  costModal.value = true;
}
async function saveSubscription() {
  const fd = new FormData();
  for (const key in form.value) {
    fd.append(key, form.value[key]);
  }
  loading.value = true;
  try {
    const { data } = await axios.post("/employers-subscription", fd);
    subsmodal.value = false;
    Swal.fire("موفق", "با موفقیت انجام شد", "success");
    getCost(currentUrl);

  } catch (err) {
    console.error(err);
  } finally {
    loading.value = false;
  }
}
function showSubsModal(employerID, subscription) {
  if (subscription) {
    form.value = {
      'start_date': subscription.start_date,
      'expiration_date': subscription.expiration_date,
      'level_type': subscription.level_type,
    };
  } else {
    form.value = {
      'start_date': null,
      'expiration_date': null,
      'level_type': '',
    };
  }
  form.value.employer_id = employerID;
  subsmodal.value = true;
}

const changePage = (page) => {
  if (page) getCost(`${currentUrl}?page=${page}`);
  else currentUrl = "/employers"
};

const deleteLogo = (id) => {
  Swal.fire({
    title: "حذف کارفرما",
    text: "آیا مطمئن هستید؟",
    icon: "warning",
    showCancelButton: true,
    confirmButtonText: "بله، حذف شود",
    cancelButtonText: "انصراف",
  }).then(async (result) => {
    if (result.isConfirmed) {
      try {
        await axios.delete(`/employers/${id}`);
        Swal.fire("موفق", "کارفرما حذف شد", "success");
        getCost(currentUrl);
      } catch (err) {
        Swal.fire("خطا", "مشکلی در حذف پیش آمد", "error");
      }
    }
  });
};

onMounted(() => {
  getCost(currentUrl);
});
</script>

<style scoped>
.table th,
.table td {
  vertical-align: middle;
}
</style>