<template>
  <div class="container mt-3 mt-md-4 px-2 px-md-3" v-if="checkPermission(['contact_view'])">
    <div class="card mb-2 header-card">
      <div class="card-header">
        <div class="d-flex flex-column flex-sm-row justify-content-between align-items-stretch align-items-sm-center gap-2 mb-3">
          <h3 class="mb-0 page-title">
            <i class="bi bi-box-fill"></i>
            <span>مدیریت درخواست ها</span>
          </h3>
        </div>
      </div>
    </div>
    <!-- جدول -->
    <div class="card">
      <div class="card-body p-2 p-md-3">
        <div v-if="loading" class="text-center py-5">
          <div class="spinner-border text-primary"></div>
        </div>

        <div v-else>
          <!-- ===== نمایش جدول در دسکتاپ ===== -->
          <div class="table-responsive d-none d-md-block">
            <table class="table table-bordered table-striped mb-0">
              <thead>
                <tr>
                  <th>شناسه</th>
                  <th>نام کامل</th>
                  <th>موضوع</th>
                  <th>موبایل</th>
                  <th>ایمیل</th>
                  <th>پیام</th>
                  <th>تاریخ</th>
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
                  <td>{{ new Date(item.created_at).toLocaleDateString('fa') }}</td>
                </tr>
              </tbody>
            </table>
          </div>

          <!-- ===== نمایش کارتی در موبایل ===== -->
          <div class="d-md-none contact-cards">
            <div
              v-for="item in contacts.data"
              :key="item.id"
              class="contact-card"
            >
              <div class="contact-card-header">
                <div class="contact-id-badge">#{{ item.id }}</div>
                <div class="contact-name">{{ item.full_name }}</div>
              </div>

              <div class="contact-card-body">
                <div class="contact-info-row" v-if="item.subject">
                  <i class="bi bi-tag"></i>
                  <span class="info-label">موضوع:</span>
                  <span class="info-value">{{ item.subject }}</span>
                </div>
                <div class="contact-info-row" v-if="item.mobile">
                  <i class="bi bi-phone"></i>
                  <span class="info-label">موبایل:</span>
                  <span class="info-value">{{ item.mobile }}</span>
                </div>
                <div class="contact-info-row" v-if="item.email">
                  <i class="bi bi-envelope"></i>
                  <span class="info-label">ایمیل:</span>
                  <span class="info-value">{{ item.email }}</span>
                </div>
                <div class="contact-info-row" v-if="item.body">
                  <i class="bi bi-chat-left-text"></i>
                  <span class="info-label">پیام:</span>
                  <span class="info-value">{{ item.body }}</span>
                </div>
                <div class="contact-info-row">
                  <i class="bi bi-calendar"></i>
                  <span class="info-label">تاریخ:</span>
                  <span class="info-value">{{ new Date(item.created_at).toLocaleDateString('fa') }}</span>
                </div>
              </div>
            </div>

            <!-- حالت خالی -->
            <div v-if="!contacts.data || contacts.data.length === 0" class="text-center py-5 text-muted">
              <i class="bi bi-inbox fs-1 d-block mb-2"></i>
              <p>درخواستی یافت نشد</p>
            </div>
          </div>

          <!-- Pagination -->
          <b-pagination v-model="currentPage" :total-rows="contacts.total" v-if="contacts.last_page != 1"
            :per-page="contacts.per_page" @Update:modelValue="changePage" align="center"
            class="mt-3 pagination-responsive"></b-pagination>

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

/* ===== جدول ===== */
.table {
  margin-bottom: 0;
}

.table thead th {
  background: #f8f9fa;
  font-weight: 600;
  color: #2d3436;
  white-space: nowrap;
  font-size: 0.9rem;
}

.table tbody td {
  vertical-align: middle;
  font-size: 0.9rem;
}

/* ===== کارت‌های موبایل ===== */
.contact-cards {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.contact-card {
  background: #fff;
  border: 1px solid #e9ecef;
  border-radius: 12px;
  padding: 14px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.04);
  transition: all 0.2s ease;
}

.contact-card:hover {
  box-shadow: 0 6px 18px rgba(0, 0, 0, 0.08);
  transform: translateY(-2px);
}

.contact-card-header {
  display: flex;
  align-items: center;
  gap: 10px;
  padding-bottom: 10px;
  border-bottom: 1px solid #f0f0f0;
  margin-bottom: 10px;
}

.contact-id-badge {
  background: linear-gradient(135deg, #6c5ce7, #a29bfe);
  color: white;
  font-size: 0.75rem;
  font-weight: 700;
  padding: 4px 10px;
  border-radius: 20px;
  flex-shrink: 0;
}

.contact-name {
  font-weight: 700;
  color: #2d3436;
  font-size: 1rem;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.contact-card-body {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.contact-info-row {
  display: flex;
  align-items: flex-start;
  gap: 8px;
  font-size: 0.85rem;
}

.contact-info-row i {
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
}

/* ========================================= */
/* ===== موبایل کوچک (کمتر از 400px) ===== */
/* ========================================= */
@media (max-width: 399.98px) {
  .page-title {
    font-size: 1rem;
  }

  .contact-card {
    padding: 12px;
  }

  .contact-name {
    font-size: 0.9rem;
  }

  .contact-info-row {
    font-size: 0.78rem;
  }
}

/* ========================================= */
/* ===== دسکتاپ: مخفی کردن کارت‌ها ===== */
/* ========================================= */
@media (min-width: 768px) {
  .contact-cards {
    display: none;
  }
}
</style>