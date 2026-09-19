<template>
  <div class="container mt-3 mt-md-4 px-2 px-md-3 sliders" v-if="checkPermission(['slider_view'])">
    <div class="card mb-3 header-card">
      <div class="card-header">
        <div class="d-flex flex-column flex-sm-row justify-content-between align-items-stretch align-items-sm-center gap-2 mb-3">
          <h3 class="mb-0 page-title">
            <i class="bi bi-sliders"></i>
            <span>مدیریت اسلایدرها</span>
          </h3>
          <router-link to="/content/sliders/create" class="btn btn-success add-btn">
            <i class="bi bi-plus"></i>
            <span>افزودن اسلایدر</span>
          </router-link>
        </div>
      </div>
    </div>

    <!-- افزودن -->


    <!-- جدول -->
    <div class="card">
      <div class="card-body p-2 p-md-3">
        <div v-if="loading" class="text-center p-4">
          <div class="spinner-border text-primary"></div>
        </div>

        <div v-else>
          <!-- ===== نمایش جدول در دسکتاپ ===== -->
          <div class="table-responsive d-none d-md-block">
            <table class="table table-bordered table-hover mb-0">
              <thead>
                <tr>
                  <th>عنوان</th>
                  <th>نوع</th>
                  <th>تصویر</th>
                  <th>عملیات</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="slider in sliders" :key="slider.id">
                  <td>{{ slider.title }}</td>
                  <td>{{ slider.type == "desktop" ? "دسکتاپ" : "موبایل" }}</td>
                  <td>
                    <img v-if="slider.image" :src="slider.image" alt="" class="img-thumbnail" style="max-width:100px;">
                  </td>
                  <td>
                    <div class="d-flex flex-wrap gap-1">
                      <router-link :to="`/content/sliders/${slider.id}/edit`" class="btn btn-sm btn-primary">
                        <i class="bi bi-pen"></i>
                        <span>ویرایش</span>
                      </router-link>
                      <button @click="deleteSlider(slider.id)" class="btn btn-sm btn-danger">
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
          <div class="d-md-none slider-cards">
            <div
              v-for="slider in sliders"
              :key="slider.id"
              class="slider-card"
            >
              <div class="slider-card-header">
                <div class="slider-title">{{ slider.title }}</div>
                <span class="badge" :class="slider.type == 'desktop' ? 'bg-primary' : 'bg-info'">
                  {{ slider.type == "desktop" ? "دسکتاپ" : "موبایل" }}
                </span>
              </div>

              <div class="slider-card-body" v-if="slider.image">
                <img :src="slider.image" alt="" class="slider-image">
              </div>

              <div class="slider-card-actions">
                <router-link :to="`/content/sliders/${slider.id}/edit`" class="btn btn-sm btn-primary flex-fill">
                  <i class="bi bi-pen"></i>
                  <span>ویرایش</span>
                </router-link>
                <button @click="deleteSlider(slider.id)" class="btn btn-sm btn-danger flex-fill">
                  <i class="bi bi-trash3-fill"></i>
                  <span>حذف</span>
                </button>
              </div>
            </div>

            <!-- حالت خالی -->
            <div v-if="!sliders || sliders.length === 0" class="text-center py-5 text-muted">
              <i class="bi bi-inbox fs-1 d-block mb-2"></i>
              <p>اسلایدری یافت نشد</p>
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
const sliders = ref({ data: [] });
const loading = ref(false);
const filters = ref({ search: "" });
let currentUrl = "/sliders";

const fetchData = async (url = currentUrl) => {
  loading.value = true;
  try {
    const { data } = await axios.get(url, { params: filters.value });
    sliders.value = data.data.map(slide => ({ id: slide.id, title: slide.title, type: slide.type, image: baseImageAddress + slide.image, }));

  } finally {
    loading.value = false;
  }
};


const deleteSlider = async (id) => {
  const result = await Swal.fire({
    title: "آیا مطمئن هستید؟",
    text: "این عملیات قابل بازگشت نیست!",
    icon: "warning",
    showCancelButton: true,
    confirmButtonText: "بله، حذف شود",
    cancelButtonText: "انصراف",
  });

  if (result.isConfirmed) {
    await axios.delete(`/sliders/${id}`);
    Swal.fire("حذف شد!", "اسلایدر با موفقیت حذف شد.", "success");
    fetchData();
  }
};

onMounted(() => {
  fetchData();
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

/* ===== کارت‌های موبایل ===== */
.slider-cards {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.slider-card {
  background: #fff;
  border: 1px solid #e9ecef;
  border-radius: 12px;
  padding: 14px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.04);
  transition: all 0.2s ease;
}

.slider-card:hover {
  box-shadow: 0 6px 18px rgba(0, 0, 0, 0.08);
  transform: translateY(-2px);
}

.slider-card-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 10px;
  padding-bottom: 10px;
  border-bottom: 1px solid #f0f0f0;
  margin-bottom: 10px;
}

.slider-title {
  font-weight: 700;
  color: #2d3436;
  font-size: 1rem;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  flex: 1;
  min-width: 0;
}

.slider-card-body {
  display: flex;
  justify-content: center;
  margin-bottom: 12px;
}

.slider-image {
  max-width: 100%;
  max-height: 150px;
  border-radius: 8px;
  border: 1px solid #e9ecef;
  object-fit: cover;
}

.slider-card-actions {
  display: flex;
  gap: 6px;
  padding-top: 10px;
  border-top: 1px solid #f0f0f0;
}

.slider-card-actions .btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 4px;
  font-size: 0.75rem;
  padding: 6px 8px;
  white-space: nowrap;
}

.slider-card-actions .btn span {
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
  .slider-card-actions .btn span {
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

  .slider-card {
    padding: 12px;
  }

  .slider-title {
    font-size: 0.9rem;
  }

  .slider-card-actions .btn {
    font-size: 0.7rem;
    padding: 5px 6px;
  }

  .slider-image {
    max-height: 120px;
  }
}

/* ========================================= */
/* ===== دسکتاپ: مخفی کردن کارت‌ها ===== */
/* ========================================= */
@media (min-width: 768px) {
  .slider-cards {
    display: none;
  }
}
</style>