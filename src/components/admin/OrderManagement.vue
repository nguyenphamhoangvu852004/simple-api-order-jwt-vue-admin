<template>
  <div>
    <h2>Quản lý Đơn Hàng</h2>

    <!-- Tìm kiếm -->
    <div class="order-filter">
      <input v-model="search" placeholder="Tìm kiếm đơn hàng..." class="search-box"/>
    </div>

    <!-- Danh sách đơn hàng -->
    <div v-if="filteredOrders && filteredOrders.length">
      <div v-for="order in filteredOrders" :key="order.OrderID" class="order-card">
        <div class="order-card-header">
          <h3>Đơn hàng #{{ order.OrderID }}</h3>
          <span :class="'status ' + order.Status">{{ order.Status }}</span>
        </div>

        <div class="order-details">
          <div class="user-info">
            <p><strong>Email:</strong> {{ order.User.Email }}</p>
            <p><strong>Họ tên:</strong> {{ order.User.FullName }}</p>
            <p><strong>Địa chỉ:</strong> {{ order.User.Address }}</p>
          </div>

          <!-- Dropdown để chọn trạng thái mới -->
          <div class="order-status">
            <label for="status">Trạng thái:</label>
            <select v-model="order.Status"
                    :disabled="order.Status === 'Closed'"
                    @change="updateOrderStatus(order.OrderID, order.Status)">
              <option value="InProgress">Đang xử lý</option>
              <option value="Resolved">Đã giải quyết</option>
              <option value="Closed">Đã đóng</option>
            </select>
            <span v-if="order.Status === 'Closed'" class="status-locked">Đơn hàng đã đóng</span>
          </div>

          <button @click="toggleOrderDetails(order.OrderID)" class="toggle-btn">
            {{ order.showDetails ? 'Ẩn chi tiết' : 'Xem chi tiết' }}
          </button>
          <button
              class="delete-btn"
              style="background: red; color: white"
              @click="removeOrder(order.OrderID)"
              :disabled="order.Status !== 'Closed'">
            Xóa
          </button>
          <div v-if="order.showDetails" class="order-items">
            <h4>Sản phẩm trong đơn hàng</h4>
            <table>
              <thead>
              <tr>
                <th>Sản phẩm</th>
                <th>Số lượng</th>
                <th>Giá</th>
                <th>Tổng tiền</th>
              </tr>
              </thead>
              <tbody>
              <tr v-for="item in order.OrderItems" :key="item.OrderItemID">
                <td>{{ item.Products.ProductName }}</td>
                <td>{{ item.Quantity }}</td>
                <td>{{ formatPrice(item.ProductSize.Price) }}</td>
                <td>{{ formatPrice(item.Amount) }}</td>
              </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>
    </div>

    <!-- Nếu không có đơn hàng nào -->
    <div v-else>
      <p>Đang tải thông tin đơn hàng...</p>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      orders: [], // Danh sách đơn hàng
      search: '', // Từ khóa tìm kiếm
    };
  },
  mounted() {
    this.fetchOrders(); // Lấy dữ liệu đơn hàng khi component được mounted
  },
  computed: {
    filteredOrders() {
      const searchTerm = this.search.toLowerCase();
      return this.orders.filter(order => {
        return (
            (order.User.FullName && order.User.FullName.toLowerCase().includes(searchTerm)) ||
            (order.User.Email && order.User.Email.toLowerCase().includes(searchTerm)) ||
            (order.User.Address && order.User.Address.toLowerCase().includes(searchTerm))
        );
      });
    },
  },

  methods: {
    // formatPrice(price) {
    //   return price.toLocaleString('vi-VN', {style: 'currency', currency: 'VND'});
    // },
    async fetchOrders() {
      const token = localStorage.getItem('accessToken');
      const response = await axios.get('http://localhost:2999/api/admin/orders', {
        headers: {
          Authorization: `Bearer ${token}`,
        },
      });
      this.orders = response.data; // Lưu danh sách đơn hàng vào data
      console.log(this.orders)
    },
    toggleOrderDetails(orderID) {
      const order = this.orders.find(order => order.OrderID === orderID);
      order.showDetails = !order.showDetails; // Toggle hiển thị chi tiết đơn hàng
    },
    async updateOrderStatus(orderId, status) {
      const token = localStorage.getItem('accessToken');
      try {
        // Gửi yêu cầu cập nhật trạng thái đơn hàng
        await axios.post(`http://localhost:2999/api/admin/orders/status/${orderId}`, {status}, {
          headers: {
            Authorization: `Bearer ${token}`,
          },
        });
        // Sau khi cập nhật thành công, tải lại danh sách đơn hàng
        await this.fetchOrders();
      } catch (error) {
        console.error('Không thể cập nhật trạng thái đơn hàng:', error);
      }
    },

    async removeOrder(id) {
      const token = localStorage.getItem('accessToken');
      const order = this.orders.find(order => order.OrderID === id);

      if (order.Status !== 'Closed') {
        alert('Chỉ có thể xóa đơn hàng đã đóng!');
        return;
      }

      try {
        await axios.delete(`http://localhost:2999/api/admin/orders/${id}`, {
          headers: {
            Authorization: `Bearer ${token}`,
          },
        });
        this.fetchOrders(); // Tải lại danh sách đơn hàng sau khi xóa
      } catch (error) {
        console.error('Không thể xóa đơn hàng:', error);
      }
    }

  },
};
</script>

<style scoped>
/* Container styles */
.order-filter {
  margin: 20px 0;
}

.search-box {
  width: 300px;
  padding: 8px 12px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
}

/* Order card styles */
.order-card {
  border: 1px solid #ddd;
  border-radius: 4px;
  margin-bottom: 15px;
  background: white;
}

.order-card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 15px;
  background: #f5f5f5;
  border-bottom: 1px solid #ddd;
}

.order-card-header h3 {
  margin: 0;
  font-size: 15px;
}

/* Status badges */
.status {
  padding: 4px 8px;
  border-radius: 4px;
  font-size: 12px;
}

.status.InProgress {
  background: #fff3cd;
  color: #856404;
}

.status.Resolved {
  background: #d4edda;
  color: #155724;
}

.status.Closed {
  background: #e9ecef;
  color: #6c757d;
}

/* Order details section */
.order-details {
  padding: 15px;
}

.user-info p {
  margin: 5px 0;
  font-size: 14px;
}

/* Status dropdown */
.order-status {
  margin: 10px 0;
}

.order-status select {
  margin-left: 10px;
  padding: 4px 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.status-locked {
  margin-left: 10px;
  font-size: 13px;
  color: #6c757d;
}

/* Toggle button */
.toggle-btn {
  background: #007bff;
  color: white;
  border: none;
  padding: 6px 12px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 13px;
  margin: 10px 0;
}

.toggle-btn:hover {
  background: #0056b3;
}

/* Order items table */
.order-items {
  margin-top: 15px;
}

.order-items h4 {
  font-size: 14px;
  margin-bottom: 10px;
}

table {
  width: 100%;
  border-collapse: collapse;
  font-size: 13px;
}

th, td {
  padding: 8px;
  text-align: left;
  border: 1px solid #ddd;
}

th {
  background: #f5f5f5;
  font-weight: 600;
}

/* Responsive styles */
@media (max-width: 768px) {
  .search-box {
    width: 100%;
  }

  .order-card-header {
    flex-direction: column;
    align-items: flex-start;
  }

  .status {
    margin-top: 5px;
  }
}

/* Nút xóa */
.delete-btn {
  background: #dc3545;
  color: white;
  border: none;
  padding: 6px 12px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 13px;
  margin: 10px 0;
  transition: background 0.3s;
}

.delete-btn:disabled {
  background: #e9ecef;
  color: #6c757d;
  cursor: not-allowed;
}

.delete-btn:hover:not(:disabled) {
  background: #c82333;
}

/* Cải tiến cho badge trạng thái */
.status {
  text-transform: uppercase;
  font-weight: bold;
  letter-spacing: 0.5px;
  display: inline-block;
}

/* Cải thiện hiển thị bảng */
.order-items table {
  background: white;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
}

.order-items th {
  text-transform: uppercase;
  font-size: 12px;
  color: #333;
}

.order-items td {
  font-size: 13px;
  color: #555;
}

.order-items tr:nth-child(even) {
  background: #f9f9f9;
}

.order-items tr:hover {
  background: #f1f1f1;
}

/* Hiệu ứng hover cho thẻ đơn hàng */
.order-card {
  transition: box-shadow 0.3s, transform 0.3s;
}

.order-card:hover {
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.15);
  transform: translateY(-5px);
}

/* Hiệu ứng focus cho hộp tìm kiếm */
.search-box:focus {
  outline: none;
  border-color: #007bff;
  box-shadow: 0 0 4px rgba(0, 123, 255, 0.6);
}

/* Mobile-friendly styles */
@media (max-width: 576px) {
  .order-card-header h3 {
    font-size: 14px;
  }

  .user-info p, .order-status label {
    font-size: 12px;
  }

  .toggle-btn, .delete-btn {
    font-size: 11px;
    padding: 5px 10px;
  }

  table th, table td {
    font-size: 11px;
    padding: 6px;
  }
}
</style>