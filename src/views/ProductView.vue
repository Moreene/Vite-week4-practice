<template>
    <div class="container" v-if="isShow">
        <div class="text-end mt-4">
            <button class="btn btn-primary" @click="openModal('new')">
                建立新的產品
            </button>
        </div>
        <table class="table mt-4">
            <thead>
                <tr>
                    <th width="120">
                        商品照
                    </th>
                    <th width="120">
                        分類
                    </th>
                    <th width="120">
                        產品名稱
                    </th>
                    <th width="120">
                        原價
                    </th>
                    <th width="120">
                        售價
                    </th>
                    <th width="100">
                        是否啟用
                    </th>
                    <th width="120">
                        編輯
                    </th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="item in products" :key="item.id">
                    <td><img :src="item.imageUrl" alt="" class="img-fluid" style="height: 160px;"></td>
                    <td>{{ item.category }}</td>
                    <td>{{ item.title }}</td>
                    <td>{{ item.origin_price }}</td>
                    <td>{{ item.price }}</td>
                    <td>
                        <span class="text-success" v-if="item.is_enabled">啟用</span>
                        <span class="text-secondary" v-else>未啟用</span>
                    </td>
                    <td>
                        <div class="btn-group">
                            <button type="button" class="btn btn-outline-primary btn-sm" @click="openModal('edit', item)">
                                編輯
                            </button>
                            <button type="button" class="btn btn-outline-danger btn-sm" @click="openModal('delete', item)">
                                刪除
                            </button>
                        </div>
                    </td>
                </tr>
            </tbody>
        </table>
        <!-- 分頁 -->
        <PaginationComponent :pagination="pagination" @emit-pages="getProduct"></PaginationComponent>
    </div>
    <!-- 新增產品 -->
    <ModalComponent ref="modal" :temp-product="tempProduct" :is-new="isNew" :clear-input="clearInput" @update="getProduct">
    </ModalComponent>
    <!-- 刪除產品 -->
    <ModalDelComponent ref="delModal" :temp-product="tempProduct" @update="getProduct"></ModalDelComponent>
</template>

<style lang="scss">
img {
    object-fit: contain;
    max-width: 100%;
}

.primary-image {
    height: 300px;
}

.images {
    height: 150px;
}
</style>

<script>
import axios from 'axios';
import Modal from 'bootstrap/js/dist/modal';

const { VITE_URL, VITE_PATH } = import.meta.env;

import ModalComponent from '@/components/ModalComponent.vue';
import ModalDelComponent from '@/components/ModalDelComponent.vue';
import PaginationComponent from '@/components/PaginationComponent.vue';

export default {
    components: {
        ModalComponent,
        ModalDelComponent,
        PaginationComponent
    },
    data() {
        return {
            products: [],
            tempProduct: {
                imagesUrl: []// 儲存小圖
            },
            // 判斷：是否為新增產品
            isNew: false,
            isShow: false,
            // 分頁
            pagination: {},
        };
    },
    methods: {
        // 確認使用者權限
        checkLogin() {
            axios.post(`${VITE_URL}/api/user/check`)
                .then(res => {
                    this.getProduct();
                    this.isShow = true;
                })
                .catch(err => {
                    alert('您沒有權限進入!');
                    location.href = "index.html";
                });
        },
        // 取得所有產品資訊
        getProduct(page = 1) {
            axios.get(`${VITE_URL}/api/${VITE_PATH}/admin/products?page=${page}`)
                .then(res => {
                    this.products = res.data.products;
                    this.pagination = res.data.pagination;
                })
                .catch(err => {
                    console.log(err);
                });
        },
        // 打開modal（透過status判斷要打開的哪一個modal）
        openModal(status, item) {
            if (status === 'new') {
                // 新增產品
                this.tempProduct = {};
                this.isNew = true;
                this.$refs.modal.productModal.show();
            } else if (status === 'edit') {
                // 修改產品
                this.tempProduct = { ...item };
                this.isNew = false;
                this.$refs.modal.productModal.show();
            } else if (status === 'delete') {
                // 刪除產品
                this.tempProduct = { ...item };
                this.isNew = false;
                this.$refs.delModal.delProductModal.show();
            };
        },
        // 取消新增產品時，清空輸入框
        clearInput() {
            this.tempProduct = {
                imagesUrl: []
            };
        },
        // 將使用分頁邏輯分割好的資料/方法回傳至sliceProduct陣列
        updateProduct(data) {
            this.sliceProduct = data;
        }
    },
    mounted() {
        const token = document.cookie.replace(
            /(?:(?:^|.*;\s*)myToken\s*\=\s*([^;]*).*$)|^.*$/, "$1",);
        axios.defaults.headers.common['Authorization'] = token;
        this.checkLogin();
    },
}
</script>