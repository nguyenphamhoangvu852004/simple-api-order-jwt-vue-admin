<template>
  <main>
    <div class="container">
      <div class="search-filter">
        <input
            type="text"
            v-model="searchQuery"
            placeholder="Tìm kiếm sản phẩm..."
        />
        <input
            type="number"
            v-model.number="minPrice"
            placeholder="Giá tối thiểu (VND)"
        />
        <input
            type="number"
            v-model.number="maxPrice"
            placeholder="Giá tối đa (VND)"
        />
        <button @click="applyFilter">Lọc</button>
      </div>
      <div class="product">
        <div class="product-item" v-for="product in paginatedProducts" :key="product.ProductID">
          <router-link :to="{name:'ProductDetail', params:{id:product.ProductID}}">
            <img :src="getImageUrl(product.ImageURL)" alt="logo">
            <div class="item-description">
              <div class="item-name">{{ product.ProductName }}</div>
              <div class="item-price">{{ getPriceRange(product.ProductSizes) }}</div>
            </div>
          </router-link>
        </div>
      </div>

      <div class="pagination">
        <button @click="prevPage" :disabled="currentPage === 1">Previous</button>
        <span>Page {{ currentPage }} of {{ totalPages }}</span>
        <button @click="nextPage" :disabled="currentPage === totalPages">Next</button>
      </div>
    </div>
  </main>
</template>

<script>
export default {
  props: ['products'],
  data() {
    return {
      currentPage: 1,
      itemsPerPage: 8,
      searchQuery: '',
      minPrice: null,
      maxPrice: null,
      filteredProducts: []
    };
  },
  computed: {
    totalPages() {
      return Math.ceil(this.filteredProducts.length / this.itemsPerPage);
    },
    paginatedProducts() {
      const start = (this.currentPage - 1) * this.itemsPerPage;
      return this.filteredProducts.slice(start, start + this.itemsPerPage);
    }
  },
  methods: {
    formatCurrency(value) {
      return new Intl.NumberFormat('vi-VN', {
        style: 'currency',
        currency: 'VND',
      }).format(value);
    },
    getImageUrl(imageName) {
      // Đảm bảo ảnh mặc định đã được yêu cầu trước
      const defaultImage = require('@/assets/image/default-product-image.jpg'); // Đường dẫn ảnh mặc định

      // Kiểm tra nếu imageName là null hoặc không có giá trị hợp lệ
      if (!imageName || imageName.trim() === '') {
        return defaultImage; // Trả về ảnh mặc định
      }

      try {
        // Kiểm tra đường dẫn ảnh có hợp lệ không
        return require(`@/assets/image/${imageName}`);
      } catch (error) {
        // Nếu có lỗi xảy ra, trả về ảnh mặc định
        return defaultImage;
      }
    },


    getPriceRange(sizes) {
      if (!sizes || sizes.length === 0) return 'Liên hệ';

      const prices = sizes.map(size => size.Price);
      const minPrice = Math.min(...prices);
      const maxPrice = Math.max(...prices);

      if (minPrice === maxPrice) {
        return this.formatCurrency(minPrice);
      } else {
        return `${this.formatCurrency(minPrice)} - ${this.formatCurrency(maxPrice)}`;
      }
    },
    nextPage() {
      if (this.currentPage < this.totalPages) {
        this.currentPage++;
      }
    },
    prevPage() {
      if (this.currentPage > 1) {
        this.currentPage--;
      }
    },
    applyFilter() {
      this.currentPage = 1; // Reset về trang đầu khi lọc
      this.filteredProducts = this.products.filter(product => {
        const priceRange = this.getPriceRange(product.ProductSizes);
        const prices = priceRange.split(' - ').map(price => {
          return parseInt(price.replace(/[^0-9]/g, ''), 10);
        });

        const minProductPrice = prices[0];
        const maxProductPrice = prices.length > 1 ? prices[1] : minProductPrice;

        const matchesSearch = product.ProductName.toLowerCase().includes(this.searchQuery.toLowerCase());
        const matchesMinPrice = this.minPrice ? minProductPrice >= this.minPrice : true;
        const matchesMaxPrice = this.maxPrice ? maxProductPrice <= this.maxPrice : true;

        return matchesSearch && matchesMinPrice && matchesMaxPrice;
      });

      // Nếu không có sản phẩm nào phù hợp với bộ lọc, hãy đảm bảo rằng trang hiện tại không vượt quá tổng số trang.

      if (this.currentPage > this.totalPages) {
        this.currentPage = this.totalPages;
      }
    }
  }
  ,
  mounted() {
    this.filteredProducts = this.products; // Khởi tạo danh sách sản phẩm đã lọc
  }
  ,
  watch: {
    products: {
      immediate: true,
      handler(newProducts) {
        this.filteredProducts = newProducts; // Cập nhật danh sách sản phẩm khi props thay đổi
        this.applyFilter(); // Áp dụng lại bộ lọc nếu cần
      }
    }
  }
}
</script>

<style scoped>
.search-filter {
  display: flex;
  justify-content: space-between; /* Căn giữa các trường tìm kiếm */
  margin-bottom: 20px; /* Thêm khoảng cách dưới cho trường tìm kiếm */
}

.search-filter input {
  padding: 10px; /* Thêm khoảng cách cho các trường nhập */
  border: 1px solid #ccc; /* Viền cho các trường nhập */
  border-radius: 4px; /* Bo góc cho các trường nhập */
  width: 30%; /* Đặt chiều rộng cho các trường nhập */
}

.search-filter button {
  padding: 10px 15px; /* Thêm khoảng cách cho nút lọc */
  background-color: #ff8a6c; /* Màu nền cho nút lọc */
  color: white; /* Màu chữ cho nút lọc */
  border: none; /* Loại bỏ viền */
  cursor: pointer; /* Hiển thị con trỏ khi hover */
  transition: background-color 0.3s; /* Thêm hiệu ứng chuyển tiếp cho màu nền */
}

.search-filter button:hover {
  background-color: #ff6f54; /* Thay đổi màu nền khi hover */
}

body {
  font-family: Mulish, sans-serif;
}

.container {
  max-width: 1200px; /* Giới hạn chiều rộng của container */
  margin: 0 auto; /* Căn giữa container */
  padding: 20px; /* Thêm khoảng cách cho container */
}

.product {
  display: grid;
  grid-template-columns: repeat(4, 1fr); /* Thay đổi số cột thành 4 để phù hợp hơn */
  gap: 15px; /* Tăng khoảng cách giữa các sản phẩm */
}

.product-item {
  border: 1px solid #cdb6b6;
  display: flex;
  flex-direction: column;
  transition: border 0.3s; /* Thêm hiệu ứng chuyển tiếp cho border */
}

.product-item:hover {
  cursor: pointer;
  border: 2px solid red;
}

.product-item img {
  width: 100%;
  height: auto;
}

.product-item a {
  text-decoration: none; /* Loại bỏ gạch chân */
  color: inherit; /* Kế thừa màu sắc từ phần tử cha */
}

.item-description {
  padding: 10px; /* Tăng khoảng cách cho mô tả sản phẩm */
  height: auto;
  text-align: center;
  color: black;
}

.item-name {
  font-size: 16px; /* Tăng kích thước chữ */
  font-family: "Quicksand", sans-serif;
  line-height: 1.5; /* Tăng khoảng cách dòng */
  margin: 10px 0; /* Giảm khoảng cách trên và dưới */
  font-weight: bold;
  word-break: break-word;
  overflow: hidden;
  text-align: center;
  display: -webkit-box;
  -webkit-line-clamp: 2; /* Cho phép hiển thị 2 dòng */
  -webkit-box-orient: vertical;
  text-overflow: ellipsis;
  overflow: hidden;
  height: 44px; /* Tăng chiều cao để phù hợp với 2 dòng */
}

.item-price {
  font-size: 16px; /* Tăng kích thước chữ cho giá */
  font-weight: bold;
  color: #ff8a6c;
}

.pagination {
  display: flex;
  justify-content: center; /* Căn giữa các nút phân trang */
  margin-top: 20px; /* Thêm khoảng cách trên cho phân trang */
}

.pagination button {
  padding: 10px 15px; /* Thêm khoảng cách cho nút */
  margin: 0 5px; /* Thêm khoảng cách giữa các nút */
  border: none; /* Loại bỏ viền */
  background-color: #ff8a6c; /* Màu nền cho nút */
  color: white; /* Màu chữ cho nút */
  font-size: 16px; /* Kích thước chữ cho nút */
  cursor: pointer; /* Hiển thị con trỏ khi hover */
  transition: background-color 0.3s; /* Thêm hiệu ứng chuyển tiếp cho màu nền */
}

.pagination button:hover {
  background-color: #ff6f54; /* Thay đổi màu nền khi hover */
}

.pagination button:disabled {
  background-color: #ccc; /* Màu nền cho nút không hoạt động */
  cursor: not-allowed; /* Hiển thị con trỏ không cho phép */
}
</style>
