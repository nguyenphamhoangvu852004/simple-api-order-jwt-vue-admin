<template>
  <div class="admin-container">
    <nav>
      <ul>
        <li @click="currentTab = 'products'">Quản lý sản phẩm</li>
        <li @click="currentTab = 'orders'">Quản lý đơn hàng</li>
        <li @click="currentTab = 'users'">Quản lý người dùng</li>


      </ul>
    </nav>

    <div v-if="currentTab === 'products'">
      <ProductManagement />
    </div>
    <div v-if="currentTab === 'orders'">
      <OrderManagement />
    </div>
    <div v-if="currentTab === 'users'">
      <UserManagement />
    </div>
  </div>
</template>

<script>
import ProductManagement from './ProductManagement.vue';
import OrderManagement from './OrderManagement.vue';
import UserManagement from './UserManagement.vue';

export default {
  data() {
    return {
      currentTab: 'products', // Tab mặc định
    };
  },
  components: {
    ProductManagement,
    OrderManagement,
    UserManagement,
  },
};
</script>

<style scoped>
.admin-container {
  padding: 20px;
  background-color: #f4f4f4; /* Màu nền nhẹ cho toàn bộ trang */
  border-radius: 8px; /* Bo góc cho container */
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1); /* Hiệu ứng bóng */
}

h1 {
  color: #333; /* Màu chữ cho tiêu đề */
  margin-bottom: 20px; /* Khoảng cách dưới tiêu đề */
}

nav {
  margin-bottom: 20px; /* Khoảng cách dưới menu */
}

nav ul {
  list-style-type: none; /* Bỏ dấu chấm đầu dòng */
  padding: 0; /* Bỏ padding */
}

nav li {
  display: inline-block; /* Hiển thị các mục menu theo hàng ngang */
  margin-right: 20px; /* Khoảng cách giữa các mục */
  cursor: pointer; /* Con trỏ chuột khi hover */
  font-weight: bold; /* Chữ đậm */
  padding: 10px 15px; /* Padding cho các mục menu */
  border-radius: 4px; /* Bo góc cho các mục menu */
  transition: background-color 0.3s; /* Hiệu ứng chuyển màu nền */
}

nav li:hover {
  background-color: #007bff; /* Màu nền khi hover */
  color: white; /* Màu chữ khi hover */
}

nav li.active {
  background-color: #0056b3; /* Màu nền cho tab đang hoạt động */
  color: white; /* Màu chữ cho tab đang hoạt động */
}

</style>