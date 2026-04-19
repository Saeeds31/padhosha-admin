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
                  <button v-if="checkPermission(['employer_comment'])" class="btn btn-sm btn-dark"
                    @click="showModal(employer.user.id)">
                    <i class="bi bi-chat-dots"></i>
                    <span>کامنت </span>
                  </button>

                  <!-- <button v-if="checkPermission(['employer_delete'])" class="btn btn-sm btn-danger"
                    @click="deleteLogo(employer.id)">
                    <i class="bi bi-trash3-fill"></i>
                    <span>حذف</span>
                  </button> -->
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
let selectedUser = ref(null);
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