<template>
  <div class="product-edit container py-4" v-if="checkPermission(['product_update'])">
    <!-- دکمه‌های مرحله‌ای -->
    <div class="step-buttons d-flex flex-wrap align-items-center mb-4">
      <template v-for="(step, index) in steps" :key="index">
        <button class="btn btn-primary d-flex align-items-end me-2 mb-2 step-btn"
          :class="{ active: currentStep === index }" :disabled="!step.enabled" @click="currentStep = index">
          <i :class="step.icon" class="me-1"></i>
          {{ step.label }}
          <span v-if="step.completed" class="ms-1 text-success">&#10003;</span>
        </button>
        <div v-if="index < steps.length - 1">
          <i class="bi bi-caret-left"></i>
          <i class="bi bi-caret-left"></i>
          <i class="bi bi-caret-left"></i>
          <i class="bi bi-caret-left"></i>
          <i class="bi bi-caret-left"></i>
        </div>
      </template>
    </div>

    <!-- مرحله اول: اطلاعات اصلی محصول -->
    <div class="bg-gray" v-if="currentStep === 0">
      <h3>
        <i class="bi bi-info"></i>
        <span>
          مرحله اول: اطلاعات اصلی محصول
        </span>
      </h3>
      <form @submit.prevent="saveStep1">
        <div class="formSetp1 g-3">
          <div class=" border-box row">
            <!-- عنوان و قیمت -->
            <div class="col-md-6 mb-3">
              <label class="form-label">عنوان محصول</label>
              <input v-model="form.title" type="text" class="form-control" />
              <span v-if="errors.step1.title" class="text-danger">{{ errors.step1.title[0] }}</span>
            </div>
            <div class="col-md-6 mb-3">
              <label class="form-label">قیمت</label>
              <input v-model="form.price" type="number" class="form-control" />
              <span v-if="errors.step1.price" class="text-danger">{{ errors.step1.price[0] }}</span>
            </div>

            <!-- دسته‌بندی‌ها -->
            <div class="col-md-12 mb-3">
              <label class="form-label">دسته‌بندی‌ها</label>
              <Treeselect v-if="form.categories" v-model="form.categories" :multiple="true" :options="categoryOptions"
                :normalizer="normalizer" />
              <span v-if="errors.step1.categories" class="text-danger">{{ errors.step1.categories[0] }}</span>
            </div>

            <!-- بارکد، SKU، موجودی -->
            <div class="col-md-4 mb-3">
              <label class="form-label">بارکد</label>
              <input v-model="form.barcode" type="text" class="form-control" />
              <span v-if="errors.step1.barcode" class="text-danger">{{ errors.step1.barcode[0] }}</span>
            </div>
            <div class="col-md-4 mb-3">
              <label class="form-label">SKU</label>
              <input v-model="form.sku" type="text" class="form-control" />
              <span v-if="errors.step1.sku" class="text-danger">{{ errors.step1.sku[0] }}</span>
            </div>
            <div class="col-md-4 mb-3">
              <label class="form-label">موجودی</label>
              <input v-model="form.stock" type="number" class="form-control" />
              <span v-if="errors.step1.stock" class="text-danger">{{ errors.step1.stock[0] }}</span>
            </div>

            <!-- توضیحات -->
            <div class="col-md-12 mb-3">
              <label class="form-label">توضیحات</label>
              <Editor v-model="form.description" />
              <span v-if="errors.step1.description" class="text-danger">{{ errors.step1.description[0] }}</span>
            </div>

            <!-- تصاویر -->
            <div class="col-md-12 mb-3">
              <label class="form-label">تصاویر</label>
              <VueFileAgent @update:rawModelValue="imageDeleted" @select="imagesLoaded"
                v-model:rawModelValue="oldImages" :multiple="true" accept=".jpg,.png,.webp,.pdf" theme="grid" deletable
                sortable />
              <span v-if="errors.step1.images" class="text-danger">{{ errors.step1.images[0] }}</span>
            </div>



          </div>

          <div class="border-box">
            <!-- وضعیت و تخفیف -->
            <div class="col-md-12 mb-3">
              <label class="form-label">وضعیت</label>
              <select v-model="form.status" class="form-select">
                <option value="">انتخاب کنید</option>
                <option value="draft">پیشنویس</option>
                <option value="published">انتشار</option>
                <option value="unavailable">ناموجود</option>
              </select>
              <span v-if="errors.step1.status" class="text-danger">{{ errors.step1.status[0] }}</span>
            </div>
            <div class="col-md-12 mb-3">
              <label class="form-label">نوع تخفیف</label>
              <select v-model="form.discount_type" class="form-select">
                <option value="">انتخاب کنید</option>
                <option value="fixed">ثابت</option>
                <option value="percent">درصدی</option>
              </select>
              <span v-if="errors.step1.discount_type" class="text-danger">{{ errors.step1.discount_type[0] }}</span>
            </div>
            <div class="col-md-12 mb-3">
              <label class="form-label">مقدار تخفیف</label>
              <input v-model="form.discount_value" type="number" class="form-control" />
              <span v-if="errors.step1.discount_value" class="text-danger">{{ errors.step1.discount_value[0] }}</span>
            </div>

            <!-- تصویر اصلی و ویدئو -->
            <div class="col-md-12 mb-3">
              <label class="form-label">تصویر اصلی</label>
              <VueFileAgent @select="imageLoaded" @update:rawModelValue="changeMainImage"
                :raw-model-value="oldMainImage" :maxFiles="1" accept=".jpg,.png,.webp" theme="grid" deletable sortable />
              <span v-if="errors.step1.main_image" class="text-danger">{{ errors.step1.main_image[0] }}</span>
            </div>
            <div class="col-md-12 mb-3">
              <label class="form-label">ویدئو</label>
              <VueFileAgent @select="videoLoaded" @update:rawModelValue="changeMainVideo"
                :raw-model-value="oldMainVideo" :maxFiles="1" accept=".mp4,.mov,.avi" theme="grid" deletable sortable />
              <span v-if="errors.step1.video" class="text-danger">{{ errors.step1.video[0] }}</span>
            </div>

          </div>
        </div>
        <div class="metaBox row border-box g-3">
          <!-- متا -->
          <div class="col-md-12 mb-3">
            <label class="form-label">عنوان متا</label>
            <input v-model="form.meta_title" type="text" class="form-control" />
            <span v-if="errors.step1.meta_title" class="text-danger">{{ errors.step1.meta_title[0] }}</span>
          </div>
          <div class="col-md-12 mb-3">
            <label class="form-label">توضیحات متا</label>
            <textarea v-model="form.meta_description" class="form-control"></textarea>
            <span v-if="errors.step1.meta_description" class="text-danger">{{ errors.step1.meta_description[0]
            }}</span>
          </div>
        </div>

        <button type="submit" :disabled="loading" class="btn btn-primary mt-3">
          <i class="bi bi-save2"></i>
          <span>
            ذخیره مرحله اول
          </span>
        </button>
      </form>
    </div>

    <!-- مرحله دوم: واریانت‌ها -->
    <div class="bg-gray" v-else-if="currentStep === 1">
      <h3>
        <i class="bi bi-info"></i>
        <span>
          مرحله دوم: تنوع ها
        </span>
      </h3>
      <form>
        <div class="row formSetp2">
          <div class="col-md-12 mb-3">
            <label>ویژگی ها:</label>
            <Treeselect v-model="selectedAttibutes" :multiple="true" :options="attributes"
              :normalizer="attributeNormalizer" />
          </div>
          <template v-for="attributeId in selectedAttibutes" :key="attributeId">
            <div class="col-md-12 mb-3">
              <label>انتخاب {{ attrName(attributeId) }}:</label>
              <Treeselect :valueFormat="'object'" v-model="attributeValue[attributeId]" :multiple="true"
                :options="attributes.find(attr => attr.id == attributeId).values"
                :normalizer="attributeValuesNormalizer" />
            </div>
          </template>
        </div>

        <div v-if="variantCombinations.length" class="table-responsive mt-3 formSetp2">
          <table class="table table-bordered">
            <thead>
              <tr>
                <template v-for="attributeId in selectedAttibutes" :key="attributeId">
                  <th v-if="attributeValue[attributeId] && attributeValue[attributeId].length">

                    {{ attrName(attributeId) }}</th>
                </template>
                <th>SKU</th>
                <th>قیمت</th>
                <th>موجودی</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(variant, index) in variantCombinations" :key="index">
                <template v-for="AV in variant.values">
                  <td>{{ AV ? AV.value : "" }}</td>
                </template>
                <td><input v-model="variant.sku" class="form-control" /></td>
                <td><input v-model="variant.price" type="number" class="form-control" /></td>
                <td><input v-model="variant.stock" type="number" class="form-control" /></td>
              </tr>
            </tbody>
          </table>
        </div>
      </form>

      <button class="btn btn-primary mt-3" @click="saveStep2" :disabled="!variantCombinations.length && loading">
        <i class="bi bi-save2"></i>
        <span>
          ذخیره مرحله دوم

        </span>
      </button>
    </div>
    <div class="bg-gray" v-else-if="currentStep === 2">
      <div class="border-box speci">
        <h3>
          <i class="bi bi-table"></i>
          <span>

            جدول مشخصات
          </span>
        </h3>
        <form>
          <div class="col-md-12 mb-3">
            <label>مشخصات:</label>
            <Treeselect :valueFormat="'object'" v-model="selectedSpecification" :multiple="true"
              :options="specification" :normalizer="specificationNormalizer" />
          </div>
          <template v-for="ss in selectedSpecification">
            <div class="col-md-12 mb-3">
              <label>انتخاب {{ ss.title }}:</label>
              <Treeselect v-model="selectedSpecificationValues[ss.id]" :multiple="true"
                :options="specification.find(spe => spe.id == ss.id)?.values" :normalizer="attributeValuesNormalizer" />
            </div>
          </template>
        </form>
      </div>
      <button class="btn btn-primary mt-3" @click="saveStep3" :disabled="!selectedSpecification.length && loading">
        <i class="bi bi-save2"></i>
        <span>
          ذخیره مرحله سوم
        </span>
      </button>
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
const productId = route.params.id // آی‌دی محصول مورد ویرایش
const currentStep = ref(0)
const product = ref(null)
const steps = ref([
  { label: 'محصول', icon: 'bi bi-file-text', enabled: true, completed: false },
  { label: 'تنوع ها', icon: 'bi bi-palette', enabled: false, completed: false },
  { label: 'مشخصات', icon: 'bi bi-table', enabled: false, completed: false },
])
let loading = ref(false);

let mainVideoDeleted = ref(false);
let mainImageDeleted = ref(false);
const form = ref({
  title: '', images: [], description: '', categories: "", main_image: "",
  meta_title: '', meta_description: '', status: '', discount_value: '',
  discount_type: '', barcode: '', sku: '', stock: '', price: '', video: ""
})
let firstInit = ref(true);
let deleted_images = ref([]);
let backupImages = ref([])
let oldMainImage = ref([])
let oldMainVideo = ref([])
let oldImages = ref([])
let attributeValue = reactive({})
const errors = ref({ step1: {}, step2: {} })
const categoryOptions = ref([])
const attributes = ref([])
const variantCombinations = ref([]);
const backupVariantCombinations = ref([]);
let selectedAttibutes = ref([])
const specification = ref([])
const selectedSpecification = ref([]);
const selectedSpecificationValues = reactive({});
const normalizer = node => ({ id: node.id, label: node.title, children: node.children })
const attributeNormalizer = node => ({ id: node.id, label: node.name })
const attributeValuesNormalizer = node => ({ id: node.id, label: node.value })
const specificationNormalizer = node => ({ id: node.id, label: node.title, values: node.values })

function attrName(id) {
  const attr = attributes.value.find(a => a.id == id)
  return attr ? attr.name : "گزینه ها"
}

watch(() => attributeValue, () => {
  if (firstInit.value == false) {
    generateCombinations()
  }
}, { deep: true })

onMounted(async () => {
  await loadCategories()
  await loadAttributes()
  await loadSpecification();
  await loadProduct()
})
async function loadSpecification() {
  const res = await axios.get('/all-specification')
  specification.value = res.data.data;


}
async function loadCategories() {
  const res = await axios.get('/categories')
  categoryOptions.value = res.data.data
}

async function loadAttributes() {
  const res = await axios.get('/attributes')
  attributes.value = res.data.data
}

async function loadProduct() {
  const res = await axios.get(`/products/${productId}`)
  product.value = res.data
  Object.assign(form.value, {
    title: product.value.title,
    description: product.value.description,
    categories: product.value.categories.map(c => c.id),
    meta_title: product.value.meta_title,
    meta_description: product.value.meta_description,
    status: product.value.status,
    discount_value: product.value.discount_value,
    discount_type: product.value.discount_type,
    barcode: product.value.barcode,
    sku: product.value.sku,
    stock: product.value.stock,
    price: product.value.price,
  })
  backupImages.value = product.value.images;
  if (product.value.video) {
    oldMainVideo.value =
      [{
        name: product.value.video.split('/').pop(),
        size: 0,
        type: 'video/*',
        ext: product.value.video.split('.').pop(),
        url: `${baseImageAddress}${product.value.video}`,
      }];
  }
  if (product.value.main_image) {
    oldMainImage.value =
      [{
        name: product.value.main_image.split('/').pop(),
        size: 0,
        type: 'image/jpeg',
        ext: product.value.main_image.split('.').pop(),
        url: `${baseImageAddress}${product.value.main_image}`,
      }];
  }
  if (product.value.images && product.value.images.length) {
    let images = [];
    product.value.images.forEach((img) => {
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
  steps.value[0].completed = true
  steps.value[1].enabled = true
  steps.value[2].enabled = true

  if (product.value.variants) {
    selectedAttibutes.value = []
    variantCombinations.value = []
    await product.value.variants.forEach(v => {
      let uid = "uid";
      v.values.forEach(val => {
        uid += `_${val.id}`;
        if (!selectedAttibutes.value.includes(val.attribute_id)) selectedAttibutes.value.push(val.attribute_id)
        if (!attributeValue[val.attribute_id]) attributeValue[val.attribute_id] = [];
        if (attributeValue[val.attribute_id].findIndex(av => av.id == val.id) == -1) {
          attributeValue[val.attribute_id].push({ id: val.id, value: val.value })

        }
      })
      variantCombinations.value.push({
        price: v.price,
        uid: uid,
        id: v.id,
        sku: v.sku,
        stock: v.stock,
        values: v.values.map(val => ({ id: val.id, value: val.value }))
      })
    })
    backupVariantCombinations.value = variantCombinations.value;
    firstInit.value = false;
  }

  if (product.value.specifications) {
    product.value.specifications.forEach((item) => {
      selectedSpecification.value.push(item)
      selectedSpecificationValues[item.id] = item.values;
    })
    console.log(selectedSpecification.value, selectedSpecificationValues);
  }
}

function generateCombinations() {
  let newVariants = [];
  const keys = Object.keys(attributeValue)
  if (!keys.length) return
  if (
    attributeValue[keys[0]] &&
    attributeValue[keys[0]].length &&
    attributeValue[keys[1]] &&
    attributeValue[keys[1]].length) {
    // دو ویژگی
    for (let f of attributeValue[keys[0]]) {
      for (let l of attributeValue[keys[1]]) {
        let obj = {
          id: "",
          sku: "",
          price: "",
          stock: "", values: [{ id: f.id, value: f.value }, { id: l.id, value: l.value }]
        };
        obj.uid = `uid_${f.id}_${l.id}`;
        newVariants.push(obj)
      }
    }
  } else {
    // یک وِیژگی
    let list = [];
    if (attributeValue[keys[0]] && attributeValue[keys[0]].length) {
      list = attributeValue[keys[0]]
    } else if (attributeValue[keys[1]] && attributeValue[keys[1]].length) {
      list = attributeValue[keys[1]]
    }
    if (list.length) {
      for (let f of list) {
        let obj = {
          sku: "",
          price: "",
          id: "",
          values: [{ id: f.id, value: f.value }],
          stock: ""
        };
        obj.uid = `uid_${f.id}`
        newVariants.push(obj)
      }
    }
  }
  newVariants = newVariants.map(
    (newVariant) => {
      let finded = backupVariantCombinations.value.find((oldVariant) => oldVariant.uid == newVariant.uid);
      if (finded) {
        return finded;
      } else {
        return newVariant;
      }
    }
  );
  variantCombinations.value = [...newVariants];
}

function imageLoaded(files) { form.value.main_image = files[0].file }
function imagesLoaded(files) {
  form.value.images = files.map(f => f.file)
}
function videoLoaded(files) { form.value.video = files[0].file }
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
function changeMainVideo() {
  if (!mainVideoDeleted.value) {
    mainVideoDeleted.value = true
  }
}
function changeMainImage() {
  if (!mainImageDeleted.value) {
    mainImageDeleted.value = true;
  }
}
async function saveStep1() {
  errors.value.step1 = {}
  loading.value = true;
  try {
    const formData = new FormData()
    Object.keys(form.value).forEach(key => {
      if (key === 'images') {
        form.value.images.forEach(img => formData.append("images[]", img))
      }
      else if (key == "video") {
        if (mainVideoDeleted.value) {
          if (form.value.video) {
            formData.append(key, form.value.video)
          }
        } else {
          formData.append(key, product.value.video)
        }
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
      else if (key != "categories" && form.value[key]) {
        formData.append(key, form.value[key] ?? "")
      }
    })
    form.value.categories.forEach(catId => formData.append("categories[]", catId))
    if (deleted_images.value.length) {
      deleted_images.value.forEach(id => {
        formData.append("deleted_images[]", id)
      })
    };
    formData.append("_method", "PUT")
    const res = await axios.post(`/products/${productId}`, formData)
    product.value = res.data
    steps.value[0].completed = true
    steps.value[1].enabled = true
    steps.value[2].enabled = true
    toast.success('مرحله اول با موفقیت بروزرسانی شد!')
  } catch (e) {
    if (e.response?.data?.errors) errors.value.step1 = e.response.data.errors
    toast.error('خطا در ذخیره مرحله اول')
  } finally {
    loading.value = false;
  }
}

async function saveStep2() {
  errors.value.step2 = {}
  if (!variantCombinations.value.length) return
  const formData = new FormData()
  formData.append('product_id', product.value.id)
  variantCombinations.value.forEach((v, index) => {
    formData.append(`variants[${index}][id]`, v.id)
    formData.append(`variants[${index}][sku]`, v.sku)
    formData.append(`variants[${index}][price]`, v.price)
    formData.append(`variants[${index}][stock]`, v.stock ?? 0)
    v.values.forEach(AV => formData.append(`variants[${index}][values][]`, AV.id))
  })
  loading.value = true;
  try {
    await axios.post(`/products-variants/${product.value.id}/update-all`, formData)
    steps.value[1].completed = true
    toast.success('مرحله دوم با موفقیت بروزرسانی شد!')
    router.push('/products')
  } catch (e) {
    if (e.response?.data?.errors) errors.value.step2 = e.response.data.errors
    toast.error('خطا در ذخیره مرحله دوم')
  } finally {
    loading.value = false;
  }
}
async function saveStep3() {
  if (!selectedSpecification.value.length) return
  const formData = new FormData()
  let index = 0;

  for (const key in selectedSpecificationValues) {
    if (selectedSpecificationValues[key]) {
      selectedSpecificationValues[key].forEach((spec) => {
        formData.append(`specifications[${index}][specification_value_id]`, spec);
        formData.append(`specifications[${index}][specification_id]`, key);
        index++;
      })
    }
  }
  loading.value = true;
  try {
    await axios.post(`/sync-specification/${product.value.id}`, formData)
    steps.value[1].completed = true
    toast.success('مرحله سوم با موفقیت ذخیره شد!')
  } catch (e) {
    if (e.response?.data?.errors) errors.value.step2 = e.response.data.errors
    toast.error('خطا در ذخیره مرحله سوم')
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
