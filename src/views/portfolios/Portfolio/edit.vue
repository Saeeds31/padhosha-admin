<template>
  <div class="product-edit container py-4" v-if="checkPermission(['product_update'])">
    <div class="bg-gray">
      <h3>
        <i class="bi bi-info"></i>
        <span>
          ویرایش نمونه کار
        </span>
      </h3>
      <form @submit.prevent="submitData">
        <div class="formSetp1 g-3">
          <div class=" border-box row">
            <!-- عنوان و قیمت -->
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
              <Treeselect v-if="form.technologies" v-model="form.technologies" :multiple="true"
                :options="technologyOptions" :normalizer="normalizer" />
              <span v-if="errors.technologies" class="text-danger">{{ errors.technologies[0] }}</span>
            </div>
            <div class="col-md-6  mb-3">
              <label class="form-label">دسته‌بندی‌ها</label>
              <Treeselect v-if="form.categories" v-model="form.categories" :multiple="true" :options="categoryOptions"
                :normalizer="normalizer" />
              <span v-if="errors.categories" class="text-danger">{{ errors.categories[0] }}</span>
            </div>

            <!-- توضیحات -->
            <div class="col-md-12 mb-3">
              <label class="form-label">توضیحات</label>
              <Editor v-model="form.description" />
              <span v-if="errors.description" class="text-danger">{{ errors.description[0] }}</span>
            </div>

            <!-- تصاویر -->
            <div class="col-md-12 mb-3">
              <label class="form-label">تصاویر</label>
              <VueFileAgent @update:rawModelValue="imageDeleted" @select="imagesLoaded"
                v-model:rawModelValue="oldImages" :multiple="true" accept=".jpg,.png,.webp,.pdf" theme="grid" deletable
                sortable />
              <span v-if="errors.images" class="text-danger">{{ errors.images[0] }}</span>
            </div>



          </div>

          <div class="border-box">
            <!-- وضعیت و تخفیف -->
            <div class="col-md-12 mb-3">
              <label class="form-label">وضعیت</label>
              <select v-model="form.status" class="form-select">
                <option value="">انتخاب کنید</option>
                <option value="0">پیشنویس</option>
                <option value="1">انتشار</option>
              </select>
              <span v-if="errors.status" class="text-danger">{{ errors.status[0] }}</span>
            </div>


            <!-- تصویر اصلی و ویدئو -->
            <div class="col-md-12 mb-3">
              <label class="form-label">تصویر اصلی</label>
              <VueFileAgent @select="imageLoaded" @update:rawModelValue="changeMainImage"
                :raw-model-value="oldMainImage" :maxFiles="1" accept=".jpg,.png,.webp" theme="grid" deletable sortable />
              <span v-if="errors.main_image" class="text-danger">{{ errors.main_image[0] }}</span>
            </div>


          </div>
        </div>
        <div class="metaBox row border-box g-3">
          <!-- متا -->
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

        <button type="submit" :disabled="loading" class="btn btn-primary mt-3">
          <i class="bi bi-save2"></i>
          <span>
            ویرایش نمونه کار
          </span>
        </button>
      </form>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, watch, onMounted } from 'vue'
import Treeselect from 'vue3-treeselect'
import 'vue3-treeselect/dist/vue3-treeselect.css'
import Editor from '@/components/shared/Editor.vue'
import { toast } from 'vue3-toastify'
import axios from 'axios'
import { useRoute, useRouter } from 'vue-router'
import { useAdmin } from '@/stores/modules/admin';
const store = useAdmin();
const checkPermission = store.checkPermission;
let route = useRoute();
let router = useRouter();
const portfolioId = route.params.id // آی‌دی محصول مورد ویرایش
const portfolio = ref(null)

let loading = ref(false);

let mainImageDeleted = ref(false);
const form = ref({
  title: '', images: [], description: '', categories: "", technologies: "", main_image: "",
  meta_title: '', meta_description: '', status: '',
})
let firstInit = ref(true);
let deleted_images = ref([]);
let backupImages = ref([])
let oldMainImage = ref([])

let oldImages = ref([])
const errors = ref({})
const categoryOptions = ref([])
const technologyOptions = ref([])
const normalizer = node => ({ id: node.id, label: node.title, children: node.children })

onMounted(async () => {
  await loadCategories()
  await loadTechology()
  await loadPortfolio()
})

async function loadTechology() {
  const res = await axios.get('/portfolio-technologies')
  technologyOptions.value = res.data.data.data
}
async function loadCategories() {
  const res = await axios.get('/portfolio-categories')
  categoryOptions.value = res.data.data.data
}

async function loadPortfolio() {
  const res = await axios.get(`/portfolios/${portfolioId}`)
  portfolio.value = res.data.data
  Object.assign(form.value, {
    title: portfolio.value.title,
    description: portfolio.value.description,
    categories: portfolio.value.categories.map(c => c.id),
    technologies: portfolio.value.technologies.map(c => c.id),
    meta_title: portfolio.value.meta_title,
    meta_description: portfolio.value.meta_description,
    status:Number( portfolio.value.status),
  })
  backupImages.value = portfolio.value.images;

  if (portfolio.value.main_image) {
    oldMainImage.value =
      [{
        name: portfolio.value.main_image.split('/').pop(),
        size: 0,
        type: 'image/jpeg',
        ext: portfolio.value.main_image.split('.').pop(),
        url: `${baseImageAddress}${portfolio.value.main_image}`,
      }];
  }
  if (portfolio.value.images && portfolio.value.images.length) {
    let images = [];
    portfolio.value.images.forEach((img) => {
      images.push(
        {
          id: img.id,
          name: img.path.split('/').pop(),
          size: 0,
          type: 'image/jpeg',
          ext: img.path.split('.').pop(),
          url: `${baseImageAddress}${img.path}`,
        }
      )
    })
    oldImages.value = images;
  }
}
function imageLoaded(files) { form.value.main_image = files[0].file }
function imagesLoaded(files) {
  form.value.images = files.map(f => f.file)
}
function imageDeleted(files) {
  backupImages.value.forEach(
    (oldimg, index) => {
      let finded = files.find((newImg) => newImg.id == oldimg.id);
      if (!finded) {
        deleted_images.value.push(oldimg.id);
        backupImages.value.splice(index, 1)
      }
    }
  );
}
function changeMainImage() {
  if (!mainImageDeleted.value) {
    mainImageDeleted.value = true;
  }
}
async function submitData() {
  errors.value = {}
  loading.value = true;
  try {
    const formData = new FormData()
    Object.keys(form.value).forEach(key => {
      if (key === 'images') {
        form.value.images.forEach(img => formData.append("images[]", img))
      }

      else if (key == "main_image") {
        if (mainImageDeleted.value) {
          if (form.value.main_image) {
            formData.append(key, form.value.main_image)
          }
        } else {
          formData.append(key, product.value.main_image)
        }
      }
      else if (key != "categories" && key != "technologies" && form.value[key]) {
        formData.append(key, form.value[key] ?? "")
      }
    })
    form.value.categories.forEach(catId => formData.append("categories[]", catId))
    form.value.technologies.forEach(catId => formData.append("technologies[]", catId))
    if (deleted_images.value.length) {
      deleted_images.value.forEach(id => {
        formData.append("deleted_images[]", id)
      })
    };
    formData.append("_method", "PUT")
    const res = await axios.post(`/portfolios/${portfolioId}`, formData)
    portfolio.value = res.data
    toast.success('نمونه کار با موفقیت بروزرسانی شد!')
  } catch (e) {
    if (e.response?.data?.errors) errors.value.step1 = e.response.data.errors
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
    border-left: 2px dashed #6c757d;
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
  border: 2px dashed gray !important;
  width: 90px;
}
</style>
