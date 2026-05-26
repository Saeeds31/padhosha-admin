<template>
  <div class="container mt-4" v-if="checkPermission(['contact_view'])">
    <div class="card mb-2">
      <div class="card-header d-flex justify-content-between align-items-center mb-3">
        <h3>
          <i class="bi bi-box-fill"></i>
          <span>مدیریت درخواست ها</span>
        </h3>
      </div>

    </div>
    <!-- جدول -->
    <div class="card">
      <div class="card-body">
        <div v-if="loading" class="text-center py-5">
          <div class="spinner-border text-primary"></div>
        </div>

        <div v-else>
          <table class="table table-bordered table-striped">
            <thead>
              <tr>
                <th>شناسه</th>
                <th>نام کامل</th>
                <th>موضوع</th>
                <th>موبایل</th>
                <th>ایمیل</th>
                <th>پیام</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="item in contacts.data" :key="item.id">
                <td>{{ item.id }}</td>
                <td>{{ item.full_name }}</td>
                <td>{{ item.subject }}</td>
                <td>{{ item.mobile }}</td>
                <td>{{ item.email }}</td>
                <td>{{ item.body }}</td>
              </tr>
            </tbody>
          </table>


          <b-pagination v-model="currentPage" :total-rows="contacts.total" v-if="contacts.last_page != 1"
            :per-page="contacts.per_page" @Update:modelValue="changePage" align="center" class="mt-3"></b-pagination>

        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import axios from "axios";
import Swal from "sweetalert2";
import { useRoute, useRouter, } from "vue-router";
import { useAdmin } from '@/stores/modules/admin';
const store = useAdmin();
const checkPermission = store.checkPermission;
const currentPage = ref(1)
const router = useRouter();
const route = useRoute();
const contacts = ref({ data: [], meta: null });
const loading = ref(false);
const filters = ref({ search: "", status: "" });
let currentUrl = "/contacts";

async function getContacts(url = currentUrl) {
  loading.value = true;
  try {
    const { data } = await axios.get(url);
    contacts.value = data.data;
    currentPage.value = data.data.current_page
  } catch (err) {
    console.error(err);
  } finally {
    loading.value = false;
  }
};

function changePage(selectedPage) {
  if (selectedPage) {
    router.replace({ name: route.name, query: { page: selectedPage } })
    getContacts(`${currentUrl}?page=${selectedPage}`)
  }
  else currentUrl = "/contacts"
}


onMounted(() => {
  currentPage.value = route.query.page ?? 1;
  getContacts(`${currentUrl}?page=${currentPage.value}`);
});
</script>