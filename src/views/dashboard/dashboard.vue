<template>
    <b-container fluid class="py-4" v-if="checkPermission(['dashboard_view'])">
        <b-row>
            <!-- سفارش‌ها -->
            <b-col cols="12" md="6" lg="6" class="mb-4">
                <b-card class="h-100 shadow-sm">
                    <h5 class="mb-3">📦 سفارش‌ها</h5>

                    <!-- آمار عددی جمع‌وجور -->
                    <b-row class="mb-3 text-center g-2">
                        <b-col cols="6" md="3" class="dashboardItem" v-for="(value, key) in dashboard.orders"
                            :key="key">
                            <div class="small text-muted">{{ orderLabels[key]
                            }}</div>
                            <strong>{{
                                isNaN(value) ? value :
                                    Number(value).toLocaleString()
                            }}</strong>
                        </b-col>
                    </b-row>

                    <!-- نمودار وضعیت سفارش -->
                    <ApexChart type="bar" height="150" :options="orderChartOptions" :series="orderSeries" />
                </b-card>
            </b-col>

            <!-- محصولات -->
            <b-col cols="12" md="6" lg="6" class="mb-4">
                <b-card class="h-100 shadow-sm">
                    <h5 class="mb-3">🛒 محصولات</h5>

                    <b-row class="mb-3 text-center">
                        <b-col cols="6" md="3" class="dashboardItem" v-for="(value, key) in dashboard.products"
                            :key="key">
                            <div class="small text-muted">{{
                                productLabels[key]

                            }}</div>
                            <strong>{{ isNaN(value) ? value :
                                Number(value).toLocaleString() }}</strong>
                        </b-col>
                    </b-row>

                    <!-- نمودار محصولات -->
                    <ApexChart type="pie" height="150" :options="productChartOptions" :series="productSeries" />
                </b-card>
            </b-col>

            <!-- کاربران -->
            <b-col cols="12" md="6" lg="6" class="mb-4">
                <b-card class="h-100 shadow-sm">
                    <h5 class="mb-3">👤 کاربران</h5>

                    <b-row class="mb-3 text-center">
                        <b-col cols="6" md="3" class="dashboardItem" v-for="(value, key) in dashboard.users" :key="key">
                            <div class="small text-muted">{{
                                userLabels[key]
                            }}</div>
                            <strong>{{ isNaN(value) ? value :
                                Number(value).toLocaleString() }}</strong>
                        </b-col>
                    </b-row>

                    <!-- نمودار کاربران -->
                    <ApexChart type="donut" height="150" :options="userChartOptions" :series="userSeries" />
                </b-card>
            </b-col>

            <!-- دیدگاه‌ها -->
            <b-col cols="12" md="6" lg="6" class="mb-4">
                <b-card class="h-100 shadow-sm">
                    <h5 class="mb-3">💬 دیدگاه‌ها</h5>

                    <b-row class="mb-3 text-center">
                        <b-col cols="6" md="3" class="dashboardItem" v-for="(value, key) in dashboard.comments"
                            :key="key">
                            <div class="small text-muted">{{
                                commentLabels[key]
                            }}</div>
                            <strong>{{ isNaN(value) ? value :
                                Number(value).toLocaleString() }}</strong>
                        </b-col>
                    </b-row>

                    <!-- نمودار دیدگاه -->
                    <ApexChart type="bar" height="150" :options="commentChartOptions" :series="commentSeries" />
                </b-card>
            </b-col>
        </b-row>
    </b-container>
</template>

<script setup>
import { ref, onMounted } from "vue";
import axios from "axios";
import ApexChart from "vue3-apexcharts";

import { useAdmin } from '@/stores/modules/admin';
const store = useAdmin();
const checkPermission = store.checkPermission;
const dashboard = ref({
    orders: {},
    products: {},
    users: {},
    comments: {},
});

// برچسب‌ها فارسی
const orderLabels = {
    total_orders: "کل سفارش‌ها",
    total_sales: "مجموع فروش",
    total_discount: "مجموع تخفیف",
    today_orders: "امروز",
    month_orders: "این ماه",
    by_status: "وضعیت‌ها",
};

const productLabels = {
    total_products: "کل محصولات",
    active_products: "فعال",
    inactive_products: "غیرفعال",
    out_of_stock: "ناموجود",
    average_price: "میانگین قیمت",
    max_price: "بیشترین قیمت",
    min_price: "کمترین قیمت",
};

const userLabels = {
    total_users: "کل کاربران",
    with_addresses: "دارای آدرس",
    with_wallet: "دارای کیف پول",
    without_wallet: "بدون کیف پول",
    today_registered: "ثبت‌نام امروز",
};

const commentLabels = {
    total_comments: "کل دیدگاه‌ها",
    approved: "تأیید شده",
    pending: "در انتظار",
    rejected: "رد شده",
    with_rating: "با امتیاز",
    average_rating: "میانگین امتیاز",
    today_comments: "امروز",
    this_month: "این ماه",
};

// نمودارها
const orderSeries = ref([{ name: "سفارش‌ها", data: [] }]);
const orderChartOptions = ref({ chart: { id: "orders" }, xaxis: { categories: [] } });

const productSeries = ref([]);
const productChartOptions = ref({ labels: ["فعال", "غیرفعال", "ناموجود"] });

const userSeries = ref([]);
const userChartOptions = ref({ labels: ["دارای کیف پول", "بدون کیف پول"] });

const commentSeries = ref([{ name: "دیدگاه‌ها", data: [] }]);
const commentChartOptions = ref({ chart: { id: "comments" }, xaxis: { categories: ["تأیید شده", "در انتظار", "رد شده"] } });

// دریافت داده از API
onMounted(async () => {
    // await axios.get("/run/permissions");
    // await axios.get("/run/assign");

    const { data } = await axios.get("/dashboard");
    dashboard.value = data.data;

    // بروزرسانی نمودار سفارش‌ها
    orderSeries.value[0].data = Object.values(dashboard.value.orders.by_status || {});
    orderChartOptions.value.xaxis.categories = Object.keys(dashboard.value.orders.by_status || {});

    // نمودار محصولات
    productSeries.value = [
        dashboard.value.products.active_products || 0,
        dashboard.value.products.inactive_products || 0,
        dashboard.value.products.out_of_stock || 0,
    ];

    // نمودار کاربران
    userSeries.value = [
        dashboard.value.users.with_wallet || 0,
        dashboard.value.users.without_wallet || 0,
    ];

    // نمودار دیدگاه‌ها
    commentSeries.value[0].data = [
        dashboard.value.comments.approved || 0,
        dashboard.value.comments.pending || 0,
        dashboard.value.comments.rejected || 0,
    ];
});
</script>

<style scoped>
h5 {
    font-size: 1rem;
    font-weight: 600;
}

.small {
    font-size: 0.85rem;
    border-bottom: 1px solid #1213;
    padding-bottom: 8px;
}

.dashboardItem strong {
    width: 100%;
    min-height: 40px;
}

.dashboardItem .small {
    background-color: var(--theme-color);
    color: white !important;
    width: 100%;
    padding: 8px 0;
}

.dashboardItem {
    border: 1px solid #1213;
    border-radius: 8px;
    min-height: 100px;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: space-between;
    box-shadow: 0 0 5px #1213;
    padding: 0;
    overflow: hidden;
}
</style>