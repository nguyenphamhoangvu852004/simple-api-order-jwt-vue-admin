<template>
  <div class="container">
    <h2>Quản lý Sản Phẩm</h2>
    <button class="btn-add-product" @click="openAddProductForm">Thêm Sản Phẩm</button>

    <!-- Form tìm kiếm sản phẩm -->
    <div class="search-container">
      <input
          type="text"
          v-model="searchQuery"
          placeholder="Tìm kiếm sản phẩm theo mã, tên,mô tả "
      />
    </div>
    <!-- Form thêm sản phẩm -->
    <div v-if="showAddForm" class="add-product-form">
      <h3>Thêm Sản Phẩm</h3>
      <form>
        <div class="form-group">
          <label for="productName">Tên Sản Phẩm</label>
          <input type="text" id="productName" v-model="newProduct.productName" required/>
        </div>
        <div class="form-group">
          <label for="description">Mô Tả</label>
          <textarea id="description" v-model="newProduct.description" required></textarea>
        </div>

        <!-- Các giá cho các kích thước -->
        <div v-for="(size, index) in newProduct.productSizes" :key="index" class="form-group">
          <label :for="'price' + size.size">{{ 'Giá ' + size.size }}</label>
          <input
              type="text"
              :id="'price' + size.size"
              v-model="size.price"
              require
              @input="onlyNumbers(size)"
          />
        </div>

        <div class="form-actions">
          <button type="submit" class="btn-save" @click="addProduct">Lưu Sản Phẩm</button>
          <button type="button" class="btn-cancel" @click="cancelAddProduct">Hủy</button>
        </div>
      </form>
    </div>

    <!-- Danh sách sản phẩm -->
    <table class="product-table" v-if="!showAddForm">
      <thead>
      <tr>
        <th>Ảnh</th>
        <th>Mã sản phẩm</th>
        <th>Tên sản phẩm</th>
        <th>Mô tả</th>
        <th>Giá</th>
        <th>Thao tác</th>
      </tr>
      </thead>
      <tbody>
      <tr v-for="product in filteredProducts" :key="product.ProductID">
        <td class="product-image">
          <img :src="getImageUrl(product.ImageURL)" alt="Product Image"/>
        </td>
        <td>{{ product.ProductID }}</td>
        <td>{{ product.ProductName }}</td>
        <td>{{ product.Description }}</td>
        <td>
          <div class="product-sizes">
            <div v-for="size in product.ProductSizes" :key="size.ProductSizeID" class="size-option">
              <span class="size">{{ size.Size }}</span> - <span class="price">{{ size.Price }}</span>
            </div>
          </div>
        </td>
        <td class="product-actions">
          <button class="action-edit" @click="editProduct(product)">Sửa</button>
          <button class="action-remove" @click="deleteProduct(product.ProductID)">Xóa</button>
        </td>
      </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      products: [], // Danh sách sản phẩm
      showAddForm: false, // Hiển thị form thêm sản phẩm
      newProduct: {
        productName: '',
        description: '',
        productSizes: [
          {size: 'Small', price: null},
          {size: 'Medium', price: null},
          {size: 'Large', price: null}
        ]
      },
      editingProduct: null, // Sản phẩm đang chỉnh sửa
      searchQuery: '',
    };
  },

  computed: {
    filteredProducts() {
      const searchTerm = this.searchQuery.toLowerCase();
      return this.products.filter(product => {
        return (
            (product.ProductID && product.ProductID.toString().includes(searchTerm)) ||
            // Tìm theo tên sản phẩm
            (product.ProductName && product.ProductName.toLowerCase().includes(searchTerm)) ||
            // Tìm theo mô tả sản phẩm
            (product.Description && product.Description.toLowerCase().includes(searchTerm))

        );
      });
    }
  },
  mounted() {
    this.fetchProducts();
  },
  methods: {
    // Lấy token từ localStorage (hoặc sessionStorage, cookie)
    getToken() {
      return localStorage.getItem('accessToken');  // Giả sử token được lưu ở localStorage với tên 'authToken'
    },

    // Thiết lập token cho header của axios
    setAuthHeader() {
      const token = this.getToken();
      if (token) {
        axios.defaults.headers.common['Authorization'] = `Bearer ${token}`;
      }
    },
    onlyNumbers(size) {
      // Chỉ cho phép nhập số, loại bỏ các ký tự không phải là số
      size.price = size.price.replace(/[^0-9]/g, '');
    },
    // Lấy danh sách sản phẩm từ server
    async fetchProducts() {
      try {
        const response = await axios.get('http://localhost:2999/api/products',)
        this.products = response.data;
        console.log(response)
      } catch (error) {
        console.error('Lỗi khi tải sản phẩm:', error);
        alert('Có lỗi xảy ra khi tải sản phẩm.');
      }
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
    // Mở form thêm sản phẩm
    openAddProductForm() {
      this.showAddForm = true;
    },

    // Hủy thêm sản phẩm
    cancelAddProduct() {
      this.showAddForm = false;
      this.resetFormNewProduct();
    },

    addProduct() {
      console.log('Dữ liệu gửi lên backend:', this.newProduct);

      // Tạo dữ liệu cho API
      const productData = {
        ProductName: this.newProduct.productName,
        Description: this.newProduct.description,
        ImageURL: this.newProduct.imageURL,
        ProductSizes: this.newProduct.productSizes.map(size => ({
          Size: size.size,
          Price: size.price
        }))
      };

      this.setAuthHeader();
      // Gửi yêu cầu POST đến backend
      axios.post('http://localhost:2999/api/admin/products', productData)
          .then(response => {
            alert('Thêm sản phẩm thành công!');
            console.log(response)
            this.showAddForm = false;
            this.resetNewProduct();
            this.fetchProducts();
          })
          .catch(error => {
            alert('Có lỗi xảy ra khi thêm sản phẩm. Vui lòng thử lại.');
            console.error('Có lỗi khi thêm sản phẩm:', error);
          });
    },
    // Cập nhật sản phẩm
    async editProduct(product) {
      // Hàm kiểm tra giá trị hợp lệ
      function isValidNumber(input) {
        // Kiểm tra nếu input là một số dương, không chứa ký tự đặc biệt hoặc chữ cái
        const regex = /^[0-9]*\.?[0-9]+$/; // Chỉ cho phép số dương và số thập phân (không có ký tự đặc biệt)
        return regex.test(input) && Number(input) > 0;
      }

      // Cập nhật thông tin sản phẩm với kiểm tra hợp lệ
      this.editingProduct = {...product}; // Sao chép thông tin sản phẩm vào form chỉnh sửa

      const newName = prompt('Nhập tên sản phẩm mới:', this.editingProduct.ProductName);
      const newDescription = prompt('Nhập mô tả sản phẩm mới:', this.editingProduct.Description);

      // Kiểm tra giá trị nhập vào có hợp lệ không
      let newPriceSmall;
      do {
        newPriceSmall = prompt('Nhập giá nhỏ:', this.editingProduct.PriceSmall);
        if (!isValidNumber(newPriceSmall)) {
          alert('Vui lòng nhập một số dương hợp lệ, không có ký tự đặc biệt hoặc chữ cái.');
        }
      } while (!isValidNumber(newPriceSmall));

      let newPriceMedium;
      do {
        newPriceMedium = prompt('Nhập giá trung bình:', this.editingProduct.PriceMedium);
        if (!isValidNumber(newPriceMedium)) {
          alert('Vui lòng nhập một số dương hợp lệ, không có ký tự đặc biệt hoặc chữ cái.');
        }
      } while (!isValidNumber(newPriceMedium));

      let newPriceLarge;
      do {
        newPriceLarge = prompt('Nhập giá lớn:', this.editingProduct.PriceLarge);
        if (!isValidNumber(newPriceLarge)) {
          alert('Vui lòng nhập một số dương hợp lệ, không có ký tự đặc biệt hoặc chữ cái.');
        }
      } while (!isValidNumber(newPriceLarge));

      const newImageURL = prompt('Nhập URL hình ảnh:', this.editingProduct.ImageURL);
      const isActive = confirm('Sản phẩm này có hoạt động không?') ? true : false;

      if (newName && newDescription && newPriceSmall && newPriceMedium && newPriceLarge && newImageURL) {
        this.editingProduct.ProductName = newName;
        this.editingProduct.Description = newDescription;
        this.editingProduct.PriceSmall = newPriceSmall;
        this.editingProduct.PriceMedium = newPriceMedium;
        this.editingProduct.PriceLarge = newPriceLarge;
        this.editingProduct.ImageURL = newImageURL;
        this.editingProduct.IsActive = isActive;

        try {
          // Gửi yêu cầu PUT để cập nhật sản phẩm
          const updatedProduct = {
            productID: this.editingProduct.ProductID,
            productName: this.editingProduct.ProductName,
            description: this.editingProduct.Description,
            imageURL: this.editingProduct.ImageURL,
            isActive: this.editingProduct.IsActive,
            priceSmall: this.editingProduct.PriceSmall,
            priceMedium: this.editingProduct.PriceMedium,
            priceLarge: this.editingProduct.PriceLarge,
          };


          console.log('ID của sản phẩm:', this.editingProduct.ProductID);
          if (!this.editingProduct.ProductID) {
            alert('Không tìm thấy ID sản phẩm!');
            return;
          }

          const token = this.getToken();
          await axios.put(`http://localhost:2999/api/admin/products/${this.editingProduct.ProductID}`, updatedProduct, {
            headers: {
              Authorization: `Bearer ${token}`,
            }
          });
          console.log(this.products);
          await this.fetchProducts(); // Cập nhật danh sách sản phẩm sau khi chỉnh sửa
        } catch (error) {
          console.error('Lỗi khi cập nhật sản phẩm:', error);
          alert('Có lỗi xảy ra khi cập nhật sản phẩm.');
        }
      } else {
        alert('Bạn cần nhập đầy đủ thông tin.');
      }
    },

    // Xóa sản phẩm
    async deleteProduct(productId) {
      if (confirm('Bạn có chắc chắn muốn xóa sản phẩm này?')) {
        try {
          this.setAuthHeader();
          await axios.delete(`http://localhost:2999/api/admin/products/${productId}`);
          this.products = this.products.filter(product => product.ProductID !== productId); // Xóa sản phẩm khỏi danh sách
        } catch (error) {
          console.error('Lỗi khi xóa sản phẩm:', error);
          alert('Có lỗi xảy ra khi xóa sản phẩm.');
        }
      }
    },

    // Reset thông tin sản phẩm mới
    resetFormNewProduct() {
      this.newProduct = {
        productName: '',
        description: '',
        priceSmall: 0,
        priceMedium: 0,
        priceLarge: 0,
      };
    },
  },
};
</script>


<style scoped>
/* Container for the whole page */
.container {
  width: 80%;
  margin: 0 auto;
  padding: 20px;
  font-family: 'Poppins', sans-serif;
}

/* Heading Styles */
h2 {
  text-align: center;
  margin-bottom: 20px;
}

/* Button to add a product */
.btn-add-product {
  background-color: #6cbd47;
  color: white;
  padding: 10px 20px;
  border-radius: 5px;
  border: none;
  cursor: pointer;
  font-size: 16px;
  margin-bottom: 20px;
}

.btn-add-product:hover {
  background-color: #5a9e3d;
}

/* Add Product Form */
.add-product-form {
  background-color: #f9f9f9;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  margin-bottom: 30px;
}

.add-product-form h3 {
  margin-bottom: 20px;
}

/* Form group for each input field */
.form-group {
  margin-bottom: 15px;
}

.form-group label {
  font-weight: bold;
}

.form-group input, .form-group textarea {
  width: 100%;
  padding: 8px;
  border-radius: 5px;
  border: 1px solid #ddd;
}

.form-group textarea {
  height: 100px;
}

/* Form action buttons */
.form-actions {
  display: flex;
  justify-content: space-between;
}

.btn-save {
  background-color: #6cbd47;
  color: white;
  padding: 10px 20px;
  border-radius: 5px;
  border: none;
  cursor: pointer;
}

.btn-save:hover {
  background-color: #5a9e3d;
}

.btn-cancel {
  background-color: #e0e0e0;
  padding: 10px 20px;
  border-radius: 5px;
  border: none;
  cursor: pointer;
}

.btn-cancel:hover {
  background-color: #b0b0b0;
}

/* Product Table Styles */
.product-table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 20px;
}

.product-table th, .product-table td {
  padding: 12px;
  text-align: center;
  border: 1px solid #ddd;
}

.product-table th {
  background-color: #f4f4f4;
}

.product-image img {
  max-width: 100px;
  height: auto;
  border-radius: 5px;
}

.product-sizes .size-option {
  margin: 5px 0;
}

.product-actions button {
  padding: 6px 12px;
  border-radius: 5px;
  cursor: pointer;
  margin: 0 5px;
}

.action-edit {
  background-color: #4caf50;
  color: white;
  border: none;
}

.action-edit:hover {
  background-color: #45a049;
}

.action-remove {
  background-color: #f44336;
  color: white;
  border: none;
}

.action-remove:hover {
  background-color: #e53935;
}

/* Container chứa ô tìm kiếm */
.search-container {
  position: relative;
  width: 100%;
  max-width: 400px; /* Giới hạn chiều rộng tối đa của ô tìm kiếm */
  margin: 0 auto; /* Căn giữa */
  padding: 10px;
}

/* Kiểu cho ô input */
.search-container input {
  width: 100%; /* Chiều rộng 100% để ô tìm kiếm đầy đủ chiều rộng của container */
  padding: 12px 20px; /* Thêm padding bên trong ô tìm kiếm */
  font-size: 16px; /* Kích thước chữ vừa phải */
  border: 2px solid #ccc; /* Đường viền mờ */
  border-radius: 20px; /* Bo góc mượt mà */
  outline: none; /* Loại bỏ viền khi ô được chọn */
  transition: all 0.3s ease; /* Thêm hiệu ứng khi hover hoặc focus */
}

/* Hiệu ứng khi ô input được focus */
.search-container input:focus {
  border-color: rgb(6, 208, 6); /* Đổi màu viền khi focus */
  box-shadow: 0 0 8px rgb(6, 208, 6); /* Thêm bóng khi focus */
}

/* Hiệu ứng hover cho ô tìm kiếm */
.search-container input:hover {
  border-color: #888; /* Đổi màu viền khi hover */
}

</style>
