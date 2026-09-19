<template>
  <div class="container mt-3 mt-md-4 px-2 px-md-3" v-if="checkPermission(['menu_view'])">
    <div class="card mb-2 header-card">
      <div class="card-header">
        <div class="d-flex flex-column flex-sm-row justify-content-between align-items-stretch align-items-sm-center gap-2 mb-3">
          <h3 class="mb-0 page-title">
            <i class="bi bi-list-columns-reverse"></i>
            <span>مدیریت منو</span>
          </h3>
          <router-link to="/content/menus/create" class="btn btn-success add-btn">
            <i class="bi bi-plus"></i>
            <span>افزودن منو</span>
          </router-link>
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
                  <th>عنوان</th>
                  <th>منو والد</th>
                  <th>عملیات</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="menu in flattenedMenus" :key="menu.id">
                  <td>{{ menu.id }}</td>
                  <td>
                    <span :style="{ 'padding-right': `${menu.level * 20}px` }">
                      {{ menu.title }}
                    </span>
                  </td>
                  <td>{{ menu.parent?.title || '---' }}</td>
                  <td>
                    <div class="d-flex flex-wrap gap-1">
                      <router-link :to="`/content/menus/${menu.id}/edit`" class="btn btn-sm btn-warning">
                        <i class="bi bi-pen"></i>
                        <span>ویرایش</span>
                      </router-link>
                      <button class="btn btn-sm btn-danger" @click="deleteMenu(menu.id)">
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
          <div class="d-md-none menu-cards">
            <div
              v-for="menu in flattenedMenus"
              :key="menu.id"
              class="menu-card"
            >
              <div class="menu-card-header">
                <div class="menu-id-badge">#{{ menu.id }}</div>
                <div class="menu-title">
                  <span :style="{ 'padding-right': `${menu.level * 12}px` }">
                    {{ menu.title }}
                  </span>
                </div>
              </div>

              <div class="menu-card-body">
                <div class="menu-info-row">
                  <i class="bi bi-diagram-3"></i>
                  <span class="info-label">منو والد:</span>
                  <span class="info-value">{{ menu.parent?.title || '---' }}</span>
                </div>
              </div>

              <div class="menu-card-actions">
                <router-link :to="`/content/menus/${menu.id}/edit`" class="btn btn-sm btn-warning flex-fill">
                  <i class="bi bi-pen"></i>
                  <span>ویرایش</span>
                </router-link>
                <button class="btn btn-sm btn-danger flex-fill" @click="deleteMenu(menu.id)">
                  <i class="bi bi-trash3-fill"></i>
                  <span>حذف</span>
                </button>
              </div>
            </div>

            <!-- حالت خالی -->
            <div v-if="!flattenedMenus || flattenedMenus.length === 0" class="text-center py-5 text-muted">
              <i class="bi bi-inbox fs-1 d-block mb-2"></i>
              <p>منویی یافت نشد</p>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import axios from "axios";
import Swal from "sweetalert2";

import { useAdmin } from '@/stores/modules/admin';
const store = useAdmin();
const checkPermission = store.checkPermission;
const menus = ref([]);
const flattenedMenus = ref([]);
const loading = ref(false);
const filters = ref({ title: "" });
let currentUrl = "/menus";

// تابع برای مسطح کردن منوهای سلسله‌مراتبی
const flattenMenus = (menuItems, level = 0, parent = null) => {
  let result = [];
  menuItems.forEach((menu) => {
    // اضافه کردن منوی فعلی به لیست
    result.push({
      ...menu,
      level, // برای نمایش فاصله‌گذاری (indentation)
      parent, // برای دسترسی به والد
    });
    // اگر فرزندان وجود دارند، آن‌ها را هم به‌صورت بازگشتی اضافه کن
    if (menu.children && menu.children.length > 0) {
      result = result.concat(
        flattenMenus(menu.children, level + 1, menu)
      );
    }
  });
  return result;
};

// دریافت منوها
const getMenus = async (url = currentUrl) => {
  loading.value = true;
  try {
    const { data } = await axios.get(url, { params: filters.value });
    menus.value = data.data;
    // تبدیل داده‌های سلسله‌مراتبی به لیست مسطح
    flattenedMenus.value = flattenMenus(data.data);
  } catch (err) {
    console.error(err);
    Swal.fire("خطا", "مشکلی در دریافت منوها پیش آمد", "error");
  } finally {
    loading.value = false;
  }
};

// حذف منو
const deleteMenu = (id) => {
  Swal.fire({
    title: "حذف منو",
    text: "آیا مطمئن هستید؟",
    icon: "warning",
    showCancelButton: true,
    confirmButtonText: "بله، حذف شود",
    cancelButtonText: "انصراف",
  }).then(async (result) => {
    if (result.isConfirmed) {
      try {
        await axios.delete(`/menus/${id}`);
        Swal.fire("موفق", "منو حذف شد", "success");
        getMenus(); // به‌روزرسانی لیست منوها
      } catch (err) {
        Swal.fire("خطا", "مشکلی در حذف پیش آمد", "error");
      }
    }
  });
};

onMounted(() => {
  getMenus();
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

/* برای بهبود نمایش فاصله‌گذاری در منوهای فرزند */
td span {
  display: inline-block;
}

/* ===== کارت‌های موبایل ===== */
.menu-cards {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.menu-card {
  background: #fff;
  border: 1px solid #e9ecef;
  border-radius: 12px;
  padding: 14px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.04);
  transition: all 0.2s ease;
}

.menu-card:hover {
  box-shadow: 0 6px 18px rgba(0, 0, 0, 0.08);
  transform: translateY(-2px);
}

.menu-card-header {
  display: flex;
  align-items: center;
  gap: 10px;
  padding-bottom: 10px;
  border-bottom: 1px solid #f0f0f0;
  margin-bottom: 10px;
}

.menu-id-badge {
  background: linear-gradient(135deg, #6c5ce7, #a29bfe);
  color: white;
  font-size: 0.75rem;
  font-weight: 700;
  padding: 4px 10px;
  border-radius: 20px;
  flex-shrink: 0;
}

.menu-title {
  font-weight: 700;
  color: #2d3436;
  font-size: 1rem;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.menu-card-body {
  display: flex;
  flex-direction: column;
  gap: 8px;
  margin-bottom: 12px;
}

.menu-info-row {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 0.85rem;
}

.menu-info-row i {
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

.menu-card-actions {
  display: flex;
  gap: 6px;
  padding-top: 10px;
  border-top: 1px solid #f0f0f0;
}

.menu-card-actions .btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 4px;
  font-size: 0.75rem;
  padding: 6px 8px;
  white-space: nowrap;
}

.menu-card-actions .btn span {
  display: none;
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
  .menu-card-actions .btn span {
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

  .menu-card {
    padding: 12px;
  }

  .menu-title {
    font-size: 0.9rem;
  }

  .menu-info-row {
    font-size: 0.78rem;
  }

  .menu-card-actions .btn {
    font-size: 0.7rem;
    padding: 5px 6px;
  }
}

/* ========================================= */
/* ===== دسکتاپ: مخفی کردن کارت‌ها ===== */
/* ========================================= */
@media (min-width: 768px) {
  .menu-cards {
    display: none;
  }
}
</style>