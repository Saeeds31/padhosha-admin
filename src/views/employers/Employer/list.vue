<template>
  <div class="container mt-3 mt-md-4 px-2 px-md-3" v-if="checkPermission(['employer_view'])">

    <div class="card mb-2 header-card">
      <div class="card-header">
        <div class="d-flex flex-column flex-sm-row justify-content-between align-items-stretch align-items-sm-center gap-2 mb-3">
          <h3 class="mb-0 page-title">
            <i class="bi bi-book-half"></i>
            <span>مدیریت کارفرمایان</span>
          </h3>
          <router-link v-if="checkPermission(['employer_store'])" to="/employers/create" class="btn btn-success add-btn">

            <i class="bi bi-plus"></i>
            <span>افزودن کارفرما</span>
          </router-link>
        </div>
      </div>
    </div>


    <!-- جدول -->
    <div class="card">
      <div class="card-body p-2 p-md-3">
        <div v-if="loading" class="text-center py-5">
          <div class="spinner-border text-primary" role="status">
            <span class="visually-hidden">در حال بارگذاری...</span>
          </div>
        </div>

        <div v-else>
          <!-- ===== نمایش جدول در دسکتاپ ===== -->
          <div class="table-responsive d-none d-md-block">
            <table class="table table-bordered table-striped mb-0">
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
                    <div class="d-flex flex-wrap gap-1">
                      <router-link v-if="checkPermission(['employer_update'])" :to="`/employers/${employer.id}/edit`"
                        class="btn btn-sm btn-warning">
                        <i class="bi bi-pen"></i>
                        <span>ویرایش</span>
                      </router-link>
                      <button v-if="checkPermission(['employer_comment'])" class="btn btn-sm btn-dark"
                        @click="showModal(employer.user.id)">
                        <i class="bi bi-chat-dots"></i>
                        <span>کامنت</span>
                      </button>
                      <button v-if="checkPermission(['employer_subscription'])" class="btn btn-sm"
                        :class="!employer.subscription ? 'btn-success' : 'btn-primary'"
                        @click="showSubsModal(employer.id, employer.subscription)">
                        <i class="bi bi-subscript"></i>
                        <span>{{ !employer.subscription ? 'ثبت اشتراک' : 'ویرایش اشتراک' }}</span>
                      </button>
                      <router-link v-if="checkPermission(['employer_cost'])" :to="`/employers/${employer.id}/cost`"
                        class="btn btn-sm btn-danger">
                        <i class="bi bi-list-check"></i>
                        <span>هزینه ها</span>
                      </router-link>
                      <router-link v-if="checkPermission(['employer_deposit'])" :to="`/employers/${employer.id}/deposit`"
                        class="btn btn-sm btn-success">
                        <i class="bi bi-list-check"></i>
                        <span>پرداختی ها</span>
                      </router-link>
                    </div>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>

          <!-- ===== نمایش کارتی در موبایل ===== -->
          <div class="d-md-none employer-cards">
            <div
              v-for="employer in employers.data"
              :key="employer.id"
              class="employer-card"
            >
              <div class="employer-card-header">
                <div class="employer-id-badge">#{{ employer.id }}</div>
                <div class="employer-name">{{ employer.user.full_name }}</div>
              </div>

              <div class="employer-card-body">
                <div class="employer-info-row">
                  <i class="bi bi-briefcase"></i>
                  <span class="info-label">کسب و کار:</span>
                  <span class="info-value">{{ employer.bussines_label }}</span>
                </div>
              </div>

              <div class="employer-card-actions">
                <router-link v-if="checkPermission(['employer_update'])" :to="`/employers/${employer.id}/edit`"
                  class="btn btn-sm btn-warning flex-fill">
                  <i class="bi bi-pen"></i>
                  <span>ویرایش</span>
                </router-link>
                <button v-if="checkPermission(['employer_comment'])" class="btn btn-sm btn-dark flex-fill"
                  @click="showModal(employer.user.id)">
                  <i class="bi bi-chat-dots"></i>
                  <span>کامنت</span>
                </button>
              </div>

              <div class="employer-card-actions">
                <button v-if="checkPermission(['employer_subscription'])" class="btn btn-sm flex-fill"
                  :class="!employer.subscription ? 'btn-success' : 'btn-primary'"
                  @click="showSubsModal(employer.id, employer.subscription)">
                  <i class="bi bi-subscript"></i>
                  <span>{{ !employer.subscription ? 'ثبت اشتراک' : 'ویرایش اشتراک' }}</span>
                </button>
              </div>

              <div class="employer-card-actions">
                <router-link v-if="checkPermission(['employer_cost'])" :to="`/employers/${employer.id}/cost`"
                  class="btn btn-sm btn-danger flex-fill">
                  <i class="bi bi-list-check"></i>
                  <span>هزینه ها</span>
                </router-link>
                <router-link v-if="checkPermission(['employer_deposit'])" :to="`/employers/${employer.id}/deposit`"
                  class="btn btn-sm btn-success flex-fill">
                  <i class="bi bi-list-check"></i>
                  <span>پرداختی ها</span>
                </router-link>
              </div>
            </div>

            <!-- حالت خالی -->
            <div v-if="!employers.data || employers.data.length === 0" class="text-center py-5 text-muted">
              <i class="bi bi-inbox fs-1 d-block mb-2"></i>
              <p>کارفرمایی یافت نشد</p>
            </div>
          </div>

          <!-- صفحه بندی  -->
          <b-pagination v-model="currentPage" :total-rows="employers.total" v-if="employers.last_page != 1"
            :per-page="employers.per_page" @Update:modelValue="changePage" align="center"
            class="mt-3 pagination-responsive"></b-pagination>

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
        <span>ثبت کامنت</span>
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
        <span>ثبت همکاری</span>
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
}

.add-btn {
  white-space: nowrap;
  display: inline-flex;
  align-items: center;
  gap: 6px;
  justify-content: center;
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
.employer-cards {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.employer-card {
  background: #fff;
  border: 1px solid #e9ecef;
  border-radius: 12px;
  padding: 14px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.04);
  transition: all 0.2s ease;
}

.employer-card:hover {
  box-shadow: 0 6px 18px rgba(0, 0, 0, 0.08);
  transform: translateY(-2px);
}

.employer-card-header {
  display: flex;
  align-items: center;
  gap: 10px;
  padding-bottom: 10px;
  border-bottom: 1px solid #f0f0f0;
  margin-bottom: 10px;
}

.employer-id-badge {
  background: linear-gradient(135deg, #6c5ce7, #a29bfe);
  color: white;
  font-size: 0.75rem;
  font-weight: 700;
  padding: 4px 10px;
  border-radius: 20px;
  flex-shrink: 0;
}

.employer-name {
  font-weight: 700;
  color: #2d3436;
  font-size: 1rem;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.employer-card-body {
  display: flex;
  flex-direction: column;
  gap: 8px;
  margin-bottom: 12px;
}

.employer-info-row {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 0.85rem;
}

.employer-info-row i {
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

.employer-card-actions {
  display: flex;
  gap: 6px;
  padding-top: 10px;
  border-top: 1px solid #f0f0f0;
}

.employer-card-actions + .employer-card-actions {
  padding-top: 6px;
  border-top: none;
}

.employer-card-actions .btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 4px;
  font-size: 0.75rem;
  padding: 6px 8px;
  white-space: nowrap;
}

.employer-card-actions .btn span {
  display: none;
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
    font-size: 1.15rem;
    justify-content: center;
    text-align: center;
    width: 100%;
  }

  .add-btn {
    width: 100%;
  }

  /* نمایش label دکمه‌ها در موبایل */
  .employer-card-actions .btn span {
    display: inline;
  }
}

/* ========================================= */
/* ===== موبایل کوچک (کمتر از 400px) ===== */
/* ========================================= */
@media (max-width: 399.98px) {
  .page-title {
    font-size: 1rem;
  }

  .employer-card {
    padding: 12px;
  }

  .employer-name {
    font-size: 0.9rem;
  }

  .employer-info-row {
    font-size: 0.78rem;
  }

  .employer-card-actions .btn {
    font-size: 0.7rem;
    padding: 5px 6px;
  }
}

/* ========================================= */
/* ===== دسکتاپ: مخفی کردن کارت‌ها ===== */
/* ========================================= */
@media (min-width: 768px) {
  .employer-cards {
    display: none;
  }
}
</style>