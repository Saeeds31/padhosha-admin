<template>
  <div class="container mt-3 mt-md-4 px-2 px-md-3" v-if="checkPermission(['portfolio_view'])">
    <div class="card mb-2 header-card">
      <div class="card-header">
        <div class="d-flex flex-column flex-sm-row justify-content-between align-items-stretch align-items-sm-center gap-2 mb-3">
          <h3 class="mb-0 page-title">
            <i class="bi bi-box-fill"></i>
            <span>مدیریت نمونه کارها</span>
          </h3>
          <router-link v-if="checkPermission(['portfolio_store'])" to="/portfolios/create" class="btn btn-success add-btn">
            <i class="bi bi-plus"></i>
            <span>افزودن نمونه کار</span>
          </router-link>
        </div>
      </div>
      <div class="card-body">
        <form @submit.prevent="getPortfolios()">
          <div class="row g-2">
            <div class="col-12 col-sm-6 col-md-4">
              <input v-model="filters.search" type="text" class="form-control search-input"
                placeholder="جستجو بر اساس نام نمونه کار" />
            </div>
            <div class="col-12 col-sm-6 col-md-2">
              <select v-model="filters.status" class="form-select">
                <option value="">همه وضعیت‌ها</option>
                <option value="1">فعال</option>
                <option value="0">غیرفعال</option>
              </select>
            </div>
            <div class="col-12 col-sm-6 col-md-2">
              <button class="btn btn-primary w-100" type="submit">جستجو</button>
            </div>
          </div>
        </form>
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
                  <th>عنوان</th>
                  <th>وضعیت</th>
                  <th>عملیات</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="portfolio in portfolios.data" :key="portfolio.id">
                  <td>{{ portfolio.id }}</td>
                  <td>{{ portfolio.title }}</td>
                  <td>
                    <span :class="portfolio.status ? 'badge bg-success' : 'badge bg-secondary'">
                      {{ Boolean(portfolio.status) ? 'فعال' : 'غیرفعال' }}
                    </span>
                  </td>
                  <td>
                    <div class="d-flex flex-wrap gap-1">
                      <router-link v-if="checkPermission(['portfolio_update'])" :to="`/portfolios/${portfolio.id}/edit`"
                        class="btn btn-sm btn-warning">
                        <i class="bi bi-pen"></i>
                        <span>ویرایش</span>
                      </router-link>
                      <button class="btn btn-sm btn-danger" v-if="checkPermission(['portfolio_delete'])"
                        @click="deleteportfolio(portfolio.id)">
                        <i class="bi bi-trash3-fill"></i>
                        <span>حذف</span>
                      </button>
                    </div>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>

          <!-- ===== نمایش کارتی در موبایل ===== -->
          <div class="d-md-none portfolio-cards">
            <div
              v-for="portfolio in portfolios.data"
              :key="portfolio.id"
              class="portfolio-card"
            >
              <div class="portfolio-card-header">
                <div class="portfolio-id-badge">#{{ portfolio.id }}</div>
                <div class="portfolio-title">{{ portfolio.title }}</div>
              </div>

              <div class="portfolio-card-body">
                <div class="portfolio-info-row">
                  <i class="bi bi-toggle-on"></i>
                  <span class="info-label">وضعیت:</span>
                  <span :class="portfolio.status ? 'badge bg-success' : 'badge bg-secondary'">
                    {{ Boolean(portfolio.status) ? 'فعال' : 'غیرفعال' }}
                  </span>
                </div>
              </div>

              <div class="portfolio-card-actions">
                <router-link v-if="checkPermission(['portfolio_update'])" :to="`/portfolios/${portfolio.id}/edit`"
                  class="btn btn-sm btn-warning flex-fill">
                  <i class="bi bi-pen"></i>
                  <span>ویرایش</span>
                </router-link>
                <button class="btn btn-sm btn-danger flex-fill" v-if="checkPermission(['portfolio_delete'])"
                  @click="deleteportfolio(portfolio.id)">
                  <i class="bi bi-trash3-fill"></i>
                  <span>حذف</span>
                </button>
              </div>
            </div>

            <!-- حالت خالی -->
            <div v-if="!portfolios.data || portfolios.data.length === 0" class="text-center py-5 text-muted">
              <i class="bi bi-inbox fs-1 d-block mb-2"></i>
              <p>نمونه کاری یافت نشد</p>
            </div>
          </div>

          <!-- Pagination -->
          <b-pagination v-model="currentPage" :total-rows="portfolios.total" v-if="portfolios.last_page != 1"
            :per-page="portfolios.per_page" @Update:modelValue="changePage" align="center"
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
const portfolios = ref({ data: [], meta: null });
const loading = ref(false);
const filters = ref({ search: "", status: "" });
let currentUrl = "/portfolios";

async function getPortfolios(url = currentUrl) {
  loading.value = true;
  try {
    const { data } = await axios.get(url, { params: filters.value });
    portfolios.value = data;
    currentPage.value = data.current_page
  } catch (err) {
    console.error(err);
  } finally {
    loading.value = false;
  }
};

function changePage(selectedPage) {
  if (selectedPage) {
    router.replace({ name: route.name, query: { page: selectedPage } })
    getPortfolios(`${currentUrl}?page=${selectedPage}`)
  }
  else currentUrl = "/portfolios"
}
const deleteportfolio = (id) => {
  Swal.fire({
    title: "حذف نمونه کار",
    text: "آیا مطمئن هستید؟",
    icon: "warning",
    showCancelButton: true,
    confirmButtonText: "بله، حذف شود",
    cancelButtonText: "انصراف",
  }).then(async (result) => {
    if (result.isConfirmed) {
      try {
        await axios.delete(`/portfolios/${id}`);
        Swal.fire("موفق", "نمونه کار حذف شد", "success");
        getPortfolios();
      } catch (err) {
        Swal.fire("خطا", "مشکلی در حذف پیش آمد", "error");
      }
    }
  });
};

onMounted(() => {
  currentPage.value = route.query.page ?? 1;
  getPortfolios(`${currentUrl}?page=${currentPage.value}`);
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

.search-input {
  border-radius: 10px;
  padding: 10px 14px;
  border: 1px solid #e0e0e0;
  transition: all 0.2s ease;
}

.search-input:focus {
  border-color: #6c5ce7;
  box-shadow: 0 0 0 3px rgba(108, 92, 231, 0.1);
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
.portfolio-cards {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.portfolio-card {
  background: #fff;
  border: 1px solid #e9ecef;
  border-radius: 12px;
  padding: 14px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.04);
  transition: all 0.2s ease;
}

.portfolio-card:hover {
  box-shadow: 0 6px 18px rgba(0, 0, 0, 0.08);
  transform: translateY(-2px);
}

.portfolio-card-header {
  display: flex;
  align-items: center;
  gap: 10px;
  padding-bottom: 10px;
  border-bottom: 1px solid #f0f0f0;
  margin-bottom: 10px;
}

.portfolio-id-badge {
  background: linear-gradient(135deg, #6c5ce7, #a29bfe);
  color: white;
  font-size: 0.75rem;
  font-weight: 700;
  padding: 4px 10px;
  border-radius: 20px;
  flex-shrink: 0;
}

.portfolio-title {
  font-weight: 700;
  color: #2d3436;
  font-size: 1rem;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.portfolio-card-body {
  display: flex;
  flex-direction: column;
  gap: 8px;
  margin-bottom: 12px;
}

.portfolio-info-row {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 0.85rem;
}

.portfolio-info-row i {
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

.portfolio-card-actions {
  display: flex;
  gap: 6px;
  padding-top: 10px;
  border-top: 1px solid #f0f0f0;
}

.portfolio-card-actions .btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 4px;
  font-size: 0.75rem;
  padding: 6px 8px;
  white-space: nowrap;
}

.portfolio-card-actions .btn span {
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

  .search-input {
    padding: 9px 12px;
    font-size: 0.9rem;
  }

  /* نمایش label دکمه‌ها در موبایل */
  .portfolio-card-actions .btn span {
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

  .portfolio-card {
    padding: 12px;
  }

  .portfolio-title {
    font-size: 0.9rem;
  }

  .portfolio-info-row {
    font-size: 0.78rem;
  }

  .portfolio-card-actions .btn {
    font-size: 0.7rem;
    padding: 5px 6px;
  }
}

/* ========================================= */
/* ===== دسکتاپ: مخفی کردن کارت‌ها ===== */
/* ========================================= */
@media (min-width: 768px) {
  .portfolio-cards {
    display: none;
  }
}
</style>