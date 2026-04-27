<template>
  <div class="container mt-4" v-if="checkPermission(['employer_view'])">

    <div class="card mb-2">
      <div class="card-header d-flex justify-content-between align-items-center mb-3">
        <h3>
          <i class="bi bi-book-half"></i>
          <span>مدیریت کارفرمایان</span>
        </h3>
        <router-link v-if="checkPermission(['employer_store'])" to="/employers/create" class="btn btn-success">

          <i class="bi bi-plus"></i>
          <span>
            افزودن کارفرما

          </span>
        </router-link>
      </div>
    </div>


    <!-- جدول -->
    <div class="card">
      <div class="card-body">
        <div v-if="loading" class="text-center py-5">
          <div class="spinner-border text-primary" role="status">
            <span class="visually-hidden">در حال بارگذاری...</span>
          </div>
        </div>

        <div v-else>
          <table class="table table-bordered table-striped">
            <thead>
              <tr>
                <th>شناسه</th>
                <th>نام کامل</th>
                <th>نام کسب و کار</th>
                <th>عملیات</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="employer in employers.data" :key="employer.id">
                <td>{{ employer.id }}</td>
                <td>{{ employer.user.full_name }}</td>
                <td>{{ employer.bussines_label }}</td>
                <td>
                  <router-link v-if="checkPermission(['employer_update'])" :to="`/employers/${employer.id}/edit`"
                    class="btn btn-sm btn-warning me-2">
                    <i class="bi bi-pen"></i>
                    <span> ویرایش</span>
                  </router-link>
                  <button v-if="checkPermission(['employer_comment'])" class="btn btn-sm me-2 btn-dark"
                    @click="showModal(employer.user.id)">
                    <i class="bi bi-chat-dots"></i>
                    <span>کامنت </span>
                  </button>
                  <button v-if="checkPermission(['employer_subscription'])" class="btn  btn-sm"
                    :class="!employer.subscription ? ' btn-success' : ' btn-primary'"
                    @click="showSubsModal(employer.id, employer.subscription)">
                    <i class="bi bi-subscript"></i>
                    <span>
                      {{ !employer.subscription ? 'ثبت اشتراک' : 'ویرایش اشتراک' }}
                    </span>
                  </button>
                  <router-link v-if="checkPermission(['employer_cost'])" :to="`/employers/${employer.id}/cost`"
                    class="btn btn-sm btn-danger me-2">
                    <i class="bi bi-list-check"></i>
                    <span> هزینه ها</span>
                  </router-link>
                  <router-link v-if="checkPermission(['employer_deposit'])" :to="`/employers/${employer.id}/deposit`"
                  class="btn btn-sm btn-success me-2">
                  <i class="bi bi-list-check"></i>
                  <span> پرداختی ها</span>
                </router-link>
                </td>
              </tr>
            </tbody>
          </table>
          <!-- صفحه بندی  -->

          <b-pagination v-model="currentPage" :total-rows="employers.total" v-if="employers.last_page != 1"
            :per-page="employers.per_page" @Update:modelValue="changePage" align="center" class="mt-3"></b-pagination>

        </div>
      </div>
    </div>
    <Modal v-if="commentModal" id="commentModal" @closeModal="() => commentModal = false" title="کامنت مشتری">
      <b-row>
        <b-col cols="12" md="12">
          <b-form-group label="نظر کارفرما" label-for="content">
            <b-form-textarea id="content" v-model="content" />
          </b-form-group>
        </b-col>
      </b-row>
      <button :disabled="loading" @click="saveComment()" class="btn btn-success">
        <i class="bi bi-save"></i>
        <span>
          ثبت کامنت
        </span>
      </button>
    </Modal>

    <Modal v-if="subsmodal" id="subsmodal" @closeModal="() => subsmodal = false" title=" اشتراک کارفرما">
      <b-row>
        <b-col cols="12" md="12">
          <b-form-group label="شروع همکاری" label-for="content">
            <date-picker class="ms-4" display-format="jYYYY/jMM/jDD" placeholder="از تاریخ" format="YYYY-MM-DD"
              v-model="form.start_date"></date-picker>
          </b-form-group>

          <b-form-group label="انقضاء پشتیبانی" label-for="content">
            <date-picker class="ms-4" display-format="jYYYY/jMM/jDD" placeholder="از تاریخ" format="YYYY-MM-DD"
              v-model="form.expiration_date"></date-picker>
          </b-form-group>

          <b-form-group label="انتخاب سطح همکاری" label-for="content">

            <select name="" v-model="form.level_type" class="form-control" id="">
              <option value="silver">نقره ای</option>
              <option value="gold">طلا</option>
              <option value="bronze">برنزی</option>
              <option value="diamond">الماس</option>
            </select>
          </b-form-group>


        </b-col>
      </b-row>
      <button :disabled="loading" @click="saveSubscription()" class=" my-3 btn btn-success">
        <i class="bi bi-save"></i>
        <span>
          ثبت همکاری
        </span>
      </button>
    </Modal>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import axios from "axios";
import Swal from "sweetalert2";
import Modal from "@/components/shared/modal.vue";
import { useAdmin } from '@/stores/modules/admin';
const store = useAdmin();
const checkPermission = store.checkPermission;
const currentPage = ref(1)
const employers = ref({ data: [], meta: null });
const loading = ref(false);
let currentUrl = "/employers";
let commentModal = ref(false);
let subsmodal = ref(false);
let selectedUser = ref(null);
let form = ref({
  'start_date': null,
  'expiration_date': null,
  'level_type': '',
})
let content = ref('');
async function getemployers(url) {
  loading.value = true;
  try {
    const { data } = await axios.get(url);
    employers.value = data.data;
    currentPage.value = data.data.current_page
  } catch (err) {
    console.error(err);
  } finally {
    loading.value = false;
  }
};
async function saveComment() {
  loading.value = true;
  try {
    const fd = new FormData();
    fd.append('content', content.value);
    fd.append('user_id', selectedUser.value);
    fd.append('commentable_type', "Support");
    fd.append('commentable_id', "1");
    fd.append('rating', 5);
    fd.append('status', 1);
    const { data } = await axios.post("/employers-comment", fd);
    commentModal.value = false;

  } catch (err) {
    console.error(err);
  } finally {
    loading.value = false;
  }
}
function showModal(userId) {
  selectedUser.value = userId;
  commentModal.value = true;
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
    getemployers(currentUrl);

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
  if (page) getemployers(`${currentUrl}?page=${page}`);
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
        getemployers(currentUrl);
      } catch (err) {
        Swal.fire("خطا", "مشکلی در حذف پیش آمد", "error");
      }
    }
  });
};

onMounted(() => {
  getemployers(currentUrl);
});
</script>

<style scoped>
.table th,
.table td {
  vertical-align: middle;
}
</style>