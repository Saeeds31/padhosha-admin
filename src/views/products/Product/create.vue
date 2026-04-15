<template>
  <div class="product-create container py-4" v-if="checkPermission(['product_update'])">
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
      <h3 class=" p-2">
        <i class="bi bi-info"></i>
        <span>
          مرحله اول: اطلاعات اصلی محصول</span>
      </h3>
      <form @submit.prevent="saveStep1">
        <div class="formSetp1  g-3">
          <div class=" border-box row">
            <div class="col-md-6 mb-3">
              <label class="form-label">عنوان محصول</label>
              <input v-model="form.title" type="text" class="form-control" />
              <span v-if="errors.step1.title" class="text-danger">{{ errors.step1.title[0] }}</span>
            </div>
            <div class="col-md-6 mb-3">
              <label class="form-label">
                قیمت
                <span v-if="form.price">
                  {{ Number(form.price).toLocaleString('fa-ir') }}
                </span>
                (تومان)
              </label>
              <input v-model="form.price" type="number" class="form-control" />
              <span v-if="errors.step1.price" class="text-danger">{{ errors.step1.price[0] }}</span>
            </div>
            <div class="col-md-12 mb-3">
              <label class="form-label">دسته‌بندی‌ها</label>
              <Treeselect v-model="form.categories" :multiple="true" :options="categoryOptions"
                :normalizer="normalizer" />
              <span v-if="errors.step1.categories" class="text-danger">{{ errors.step1.categories[0] }}</span>
            </div>
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
              <label class="form-label">
                موجودی
                <span v-if="form.stock">
                  {{ Number(form.stock).toLocaleString('fa-ir') }}

                </span>

              </label>
              <input v-model="form.stock" type="number" class="form-control" />
              <span v-if="errors.step1.stock" class="text-danger">{{ errors.step1.stock[0] }}</span>
            </div>
            <div class="col-md-12 mb-3">
              <label class="form-label">توضیحات</label>
              <Editor v-model="form.description" />
              <span v-if="errors.step1.description" class="text-danger">{{ errors.step1.description[0] }}</span>
            </div>
            <div class="col-md-12 mb-3">
              <label class="form-label">تصاویر</label>
              <VueFileAgent @select="imagesLoaded" :multiple="true" accept=".jpg,.png,.webp,.pdf" theme="grid" deletable
                sortable />
              <span v-if="errors.step1.images" class="text-danger">{{ errors.step1.images[0] }}</span>
            </div>

          </div>
          <div class=" border-box">


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
                <option value="percent">درصدی</option>
                <option value="fixed">ثابت</option>
              </select>
              <span v-if="errors.step1.discount_type" class="text-danger">{{ errors.step1.discount_type[0] }}</span>
            </div>
            <div class="col-md-12 mb-3">
              <label class="form-label">تخفیف</label>
              <input v-model="form.discount_value" type="number" class="form-control" />
              <span v-if="errors.step1.discount_value" class="text-danger">{{ errors.step1.discount_value[0] }}</span>
            </div>
            <div class="col-md-12 mb-3">
              <label class="form-label">تصویر اصلی</label>
              <VueFileAgent @select="imageLoaded" :maxFiles="1" accept=".jpg,.png,.webp" theme="grid" deletable sortable />
              <span v-if="errors.step1.main_image" class="text-danger">{{ errors.step1.main_image[0] }}</span>
            </div>

            <div class="col-md-12 mb-3">
              <label class="form-label">ویدئو</label>
              <VueFileAgent @select="videoLoaded" :maxFiles="1" accept=".mp4,.mov,.avi" theme="grid" deletable
                sortable />
              <span v-if="errors.step1.video" class="text-danger">{{ errors.step1.video[0] }}</span>
            </div>

          </div>
        </div>

        <div class="metaBox row border-box g-3">

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

        <button :disabled="loading" type="submit" class="btn btn-primary mt-3">
          <i class="bi bi-save2"></i>
          <span class="mx-2">ذخیره مرحله اول</span>
        </button>
      </form>
    </div>

    <!-- مرحله دوم: واریانت‌ها -->
    <div class="bg-gray" v-else-if="currentStep === 1">
      <h3 class=" p-2">
        <i class="bi bi-list-task"></i>
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
          <template v-for="attributeId in selectedAttibutes">

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
                <template v-for="attributeId in selectedAttibutes">
                  <th v-if="attributeValue[attributeId] && attributeValue[attributeId].length">
                    {{ attrName(attributeId) }}
                  </th>
                </template>
                <th>SKU</th>
                <th>قیمت</th>
                <th>موجودی</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(variant, index) in variantCombinations" :key="index">
                <template v-for="AV in variant.values">
                  <td>
                    {{ AV ? AV.value : "" }}
                  </td>
                </template>
                <td><input v-model="variant.sku" class="form-control" /></td>
                <td><input v-model="variant.price" type="number" class="form-control" /></td>
                <td><input v-model="variant.stock" type="number" class="form-control" /></td>
              </tr>
            </tbody>
          </table>
        </div>
      </form>

      <button class="btn btn-primary mt-3" v-if="currentStep === 1" @click="saveStep2"
        :disabled="!variantCombinations.length && loading">
        <i class="bi bi-save2"></i>
        <span>
          ذخیره مرحله دوم

        </span>

      </button>
    </div>
    <div v-else-if="currentStep === 2">
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
              <Treeselect :valueFormat="'object'" v-model="selectedSpecificationValues[ss.id]" :multiple="true"
                :options="ss.values" :normalizer="attributeValuesNormalizer" />
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
const currentStep = ref(0)
const product = ref(null)
const steps = ref([
  { label: 'محصول', icon: 'bi bi-file-text', enabled: true, completed: false },
  { label: 'تنوع ها', icon: 'bi bi-palette', enabled: false, completed: false },
  { label: 'مشخصات', icon: 'bi bi-table', enabled: false, completed: false },
])
function imageLoaded(files) {
  form.value.main_image = files[0].file
}
let loading = ref(false);

function imagesLoaded(files) {
  form.value.images = files.map((file) => file.file)
}
function videoLoaded(files) {
  form.value.video = files[0].file
}
const form = ref({
  title: '', images: [], description: '', categories: [], main_image: "",
  meta_title: '', meta_description: '', status: '', discount_value: '',
  discount_type: '', barcode: '', sku: '', stock: '', price: '', video: ""
})
let attributeValue = reactive({});
const errors = ref({ step1: {}, step2: {} })
const categoryOptions = ref([])
const specification = ref([])
const attributes = ref([])
const variantCombinations = ref([])
const selectedSpecification = ref([]);
const selectedSpecificationValues = reactive({});

let selectedAttibutes = ref([])
const normalizer = node => ({ id: node.id, label: node.title, children: node.children })
const attributeNormalizer = node => ({ id: node.id, label: node.name })
const attributeValuesNormalizer = node => ({ id: node.id, label: node.value })
const specificationNormalizer = node => ({ id: node.id, label: node.title, values: node.values })

function attrName(id) {
  let findedAttribute = attributes.value.find(attr => attr.id === id);
  if (findedAttribute) {
    return findedAttribute.name
  } else {
    return "گزینه ها"
  }
}
watch(() => attributeValue, () => {
  generateCombinations()
}, { deep: true })

onMounted(() => {
  loadCategories();
  loadAttributes();
  loadSpecification();
})
async function loadSpecification() {
  const res = await axios.get('/all-specification')
  specification.value = res.data.data
}
async function loadCategories() {
  const res = await axios.get('/categories')
  categoryOptions.value = res.data.data
}
async function loadAttributes() {
  const res = await axios.get('/attributes')
  attributes.value = res.data.data;
}
async function generateCombinations() {
  await nextTick();
  let newVariantList = [];
  // variantCombinations.value = []
  let keys = Object.keys(attributeValue);
  if (
    attributeValue[keys[0]] &&
    attributeValue[keys[0]].length &&
    attributeValue[keys[1]] &&
    attributeValue[keys[1]].length) {
    // دو ویژگی
    for (let f of attributeValue[keys[0]]) {
      for (let l of attributeValue[keys[1]]) {
        let obj = {
          sku: '', price: product.value ? product.value.price : '', stock: product.value ? product.value.stock : '', values: []
        };
        obj.values.push({ id: f.id, value: f.value });
        obj.values.push({ id: l.id, value: l.value });
        obj.id = `uid_${f.id}_${l.id}`;
        newVariantList.push(obj)
      }
    }
  } else {
    // یک وِیژگی
    let list = [];
    if (attributeValue[keys[0]] && attributeValue[keys[0]].length) {
      list = attributeValue[keys[0]]
    } else if (attributeValue[keys[1]] && attributeValue[keys[1]].length) {
      console.log(22);
      list = attributeValue[keys[1]]
    }
    if (list.length) {
      for (let f of list) {
        let obj = {
          sku: '', price: product.value ? product.value.price : '', stock: product.value ? product.value.stock : '', values: []
        };
        obj.values.push({ id: f.id, value: f.value });
        obj.id = `uid_${f.id}`
        newVariantList.push(obj)
      }
    }
  }
  newVariantList = newVariantList.map(
    (newVariant) => {
      let finded = variantCombinations.value.find((oldVariant) => oldVariant.id == newVariant.id);
      if (finded) {
        return finded;
      } else {
        return newVariant;
      }
    }
  );
  variantCombinations.value = newVariantList;
}

async function saveStep1() {
  errors.value.step1 = {}
  loading.value = true;
  try {
    const formData = new FormData()
    Object.keys(form.value).forEach(key => {
      if (key === 'images') {
        form.value.images.forEach(imgfile => {
          formData.append("images[]", imgfile)
        })
      } else if (key != "categories") {
        formData.append(key, form.value[key])
      }
    })
    form.value.categories.forEach(catId => {
      formData.append("categories[]", catId)
    })
    const res = await axios.post('/products', formData)
    console.log(res.data);

    product.value = res.data
    steps.value[0].completed = true
    steps.value[1].enabled = true
    steps.value[2].enabled = true
    toast.success('مرحله اول با موفقیت ذخیره شد!')
  } catch (e) {
    console.log(e);
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

  variantCombinations.value.forEach((v, index) => {
    formData.append(`variants[${index}][sku]`, v.sku)
    formData.append(`variants[${index}][price]`, v.price)
    formData.append(`variants[${index}][stock]`, v.stock ?? 0)
    v.values.forEach(AV => {
      formData.append(`variants[${index}][values][]`, AV.id)
    })
  })
  loading.value = true;
  try {
    await axios.post(`/product-variant/${product.value.id}/variants`, formData)
    steps.value[1].completed = true
    toast.success('مرحله دوم با موفقیت ذخیره شد!')
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
        formData.append(`specifications[${index}][specification_value_id]`, spec.id);
        formData.append(`specifications[${index}][specification_id]`, key);
        index++;
      })
    }
  }
  loading.value = true;
  try {
    await axios.post(`/sync-specification/${product.value.id}`, formData)
    steps.value[1].completed = true
    toast.success('مرحله دوم با موفقیت ذخیره شد!')
  } catch (e) {
    if (e.response?.data?.errors) errors.value.step2 = e.response.data.errors
    toast.error('خطا در ذخیره مرحله دوم')
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
