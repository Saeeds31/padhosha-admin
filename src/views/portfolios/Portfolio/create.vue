<template>
  <div class="product-create container py-4" v-if="checkPermission(['portfolio_store'])">
    <div class="bg-gray">
      <h3 class=" p-2">
        <i class="bi bi-info"></i>
        <span>ثبت اطلاعات نمونه کار</span>
      </h3>
      <form @submit.prevent="submitData">
        <div class="formSetp1  g-3">
          <div class=" border-box row">
            <div class="col-md-6 mb-3">
              <label class="form-label">عنوان نمونه کار</label>
              <input v-model="form.title" type="text" class="form-control" />
              <span v-if="errors.title" class="text-danger">{{ errors.title[0] }}</span>
            </div>
            <div class="col-md-6 mb-3">
              <label class="form-label">لینک نمونه کار</label>
              <input v-model="form.link" type="text" class="form-control" />
              <span v-if="errors.link" class="text-danger">{{ errors.link[0] }}</span>
            </div>
            <div class="col-md-6 mb-3">
              <label class="form-label">تکنولوژی ها</label>
              <Treeselect v-model="form.technologies" :multiple="true" :options="technologyOptions"
                :normalizer="normalizer" />
              <span v-if="errors.technologies" class="text-danger">{{ errors.technologies[0] }}</span>
            </div>
            <div class="col-md-6 mb-3">
              <label class="form-label">دسته‌بندی‌ها</label>
              <Treeselect v-model="form.categories" :multiple="true" :options="categoryOptions"
                :normalizer="normalizer" />
              <span v-if="errors.categories" class="text-danger">{{ errors.categories[0] }}</span>
            </div>

            <div class="col-md-12 mb-3">
              <label class="form-label">توضیحات</label>
              <Editor v-model="form.description" />
              <span v-if="errors.description" class="text-danger">{{ errors.description[0] }}</span>
            </div>
            <div class="col-md-12 mb-3">
              <label class="form-label">تصاویر</label>
              <VueFileAgent @select="imagesLoaded" :multiple="true" accept=".jpg,.png,.webp" theme="grid" deletable
                sortable />
              <span v-if="errors.images" class="text-danger">{{ errors.images[0] }}</span>
            </div>

          </div>
          <div class=" border-box">

            <div class="col-md-12 mb-3">
              <label class="form-label">slug</label>
              <input v-model="form.slug" type="text" class="form-control" />
              <span v-if="errors.slug" class="text-danger">{{ errors.slug[0] }}</span>
            </div>
            <div class="col-md-12 mb-3">
              <label class="form-label">وضعیت</label>
              <select v-model="form.status" class="form-select">
                <option value="">انتخاب کنید</option>
                <option value="0">پیشنویس</option>
                <option value="1">انتشار</option>
              </select>
              <span v-if="errors.status" class="text-danger">{{ errors.status[0] }}</span>
            </div>

            <div class="col-md-12 mb-3">
              <label class="form-label">تصویر اصلی</label>
              <VueFileAgent @select="imageLoaded" :maxFiles="1" accept=".jpg,.png,.webp" theme="grid" deletable
                sortable />
              <span v-if="errors.main_image" class="text-danger">{{ errors.main_image[0] }}</span>
            </div>
          </div>
        </div>

        <div class="metaBox row border-box g-3">

          <div class="col-md-12 mb-3">
            <label class="form-label">عنوان متا</label>
            <input v-model="form.meta_title" type="text" class="form-control" />
            <span v-if="errors.meta_title" class="text-danger">{{ errors.meta_title[0] }}</span>
          </div>

          <div class="col-md-12 mb-3">
            <label class="form-label">توضیحات متا</label>
            <textarea v-model="form.meta_description" class="form-control"></textarea>
            <span v-if="errors.meta_description" class="text-danger">{{ errors.meta_description[0]
            }}</span>
          </div>
        </div>

        <button :disabled="loading" type="submit" class="btn btn-primary mt-3">
          <i class="bi bi-save2"></i>
          <span class="mx-2">ذخیره نمونه کار</span>
        </button>
      </form>
    </div>



  </div>
</template>

<script setup>
import { ref, watch, onMounted, reactive, nextTick } from 'vue'
import Treeselect from 'vue3-treeselect'
import 'vue3-treeselect/dist/vue3-treeselect.css'
import Editor from '@/components/shared/Editor.vue'
import axios from 'axios'
import { toast } from 'vue3-toastify'
import 'vue3-toastify/dist/index.css'
import { useAdmin } from '@/stores/modules/admin';
const store = useAdmin();
const checkPermission = store.checkPermission;
const portfolio = ref(null)
function imageLoaded(files) {
  form.value.main_image = files[0].file
}
let loading = ref(false);
function imagesLoaded(files) {
  form.value.images = files.map((file) => file.file)
}
const form = ref({
  title: '', link: '', slug: '', images: [], description: '', categories: [], technologies: [], main_image: "",
  meta_title: '', meta_description: '', status: ''
})
const errors = ref({})
const categoryOptions = ref([])
const technologyOptions = ref([])
const normalizer = node => ({ id: node.id, label: node.title, children: node.children })

onMounted(() => {
  loadCategories();
  loadTechnologies();
})
async function loadTechnologies() {
  const res = await axios.get('/portfolio-technologies')
  technologyOptions.value = res.data.data.data
}

async function loadCategories() {
  const res = await axios.get('/portfolio-categories')
  categoryOptions.value = res.data.data.data
}

async function submitData() {
  errors.value = {}
  loading.value = true;
  try {
    const formData = new FormData()
    Object.keys(form.value).forEach(key => {
      if (key === 'images') {
        form.value.images.forEach(imgfile => {
          formData.append("images[]", imgfile)
        })
      } else if (key != "categories" && key != "technologies") {
        formData.append(key, form.value[key])
      }
    })
    form.value.categories.forEach(catId => {
      formData.append("categories[]", catId)
    })
    form.value.technologies.forEach(catId => {
      formData.append("technologies[]", catId)
    })
    const res = await axios.post('/portfolios', formData)
    portfolio.value = res.data
    toast.success('نمونه کار با موفقیت ذخیره شد!')
  } catch (e) {
    console.log(e);
    if (e.response?.data?.errors) errors.value = e.response.data.errors
    toast.error('خطا در ذخیره نمونه کار')
  } finally {
    loading.value = false;
  }
}

</script>

<style scoped>
.step-buttons {
  flex-wrap: wrap;
}



@media (max-width:768px) {
  .step-buttons {
    flex-direction: column;
  }

  .step-divider {
    border-top: 0;
    border-left: 2px dashed #0d6efd;
    height: 24px;
    margin: 0.5rem 0;
  }
}

.border-box {
  border: 1px solid #e2e2e2;
  border-radius: 8px;
  padding: 16px;
  box-shadow: 0 0 5px #1213;
}

.formSetp1 {
  display: grid;
  grid-template-columns: 9fr 3fr;
  width: 95%;
  margin: 24px auto;
}

.speci,
.formSetp2 {
  border: 1px solid #e2e2e2;
  border-radius: 8px;
  padding: 16px;
  box-shadow: 0 0 5px #1213;
  width: 95%;
  margin: 24px auto;
}

.metaBox {
  width: 95%;
  margin: 24px auto;
}

.g-3 {
  gap: 16px;
}

.step-buttons {
  width: max(50%, 380px);
  margin: auto;
}

.step-divider {
  border: 2px dashed #0d6efd !important;
  width: 90px;
}
</style>
