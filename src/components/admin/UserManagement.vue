<template>
  <div class="user-management">
    <h2 class="title">Quản lý người dùng</h2>

    <!-- Tìm kiếm đơn giản -->
    <div class="search-box">
      <input
          v-model="searchQuery"
          type="text"
          placeholder="Tìm kiếm người dùng theo ID, Email, Họ tên, Số điện thoại, Địa chỉ"
          class="search-input"
      />
    </div>

    <!-- Bảng danh sách người dùng -->
    <table class="user-table">
      <thead>
      <tr>
        <th>ID</th>
        <th>Email</th>
        <th>Họ tên</th>
        <th>Số điện thoại</th>
        <th>Địa chỉ</th>
        <th>Hành động</th>
      </tr>
      </thead>
      <tbody>
      <tr v-for="user in filteredUsers" :key="user.UserID">
        <td>{{ user.UserID }}</td>
        <td>{{ user.Email}}</td>
        <td>{{ user.FullName }}</td>
        <td>{{ user.PhoneNumber }}</td>
        <td>{{ user.Address  }}</td>
        <td>
          <button @click="editUser(user)">Sửa</button>
          <button @click="removeUser(user.UserID)">Xóa</button>
        </td>
      </tr>
      </tbody>
    </table>

    <!-- Form sửa người dùng -->
    <div v-if="editMode" class="edit-user-form">
      <form @submit.prevent="updateUser">
        <div>
          <label for="userId">ID:</label>
          <input type="text" v-model="editUserData.userId" disabled/>
        </div>
        <div>
          <label for="email">Email:</label>
          <input type="email" v-model="editUserData.email" required/>
        </div>
        <div>
          <label for="fullname">Họ tên:</label>
          <input type="text" v-model="editUserData.fullname" required/>
        </div>
        <div>
          <label for="phoneNumber">Số điện thoại:</label>
          <input type="text" v-model="editUserData.phoneNumber" required/>
        </div>
        <div>
          <label for="address">Địa chỉ:</label>
          <input type="text" v-model="editUserData.address" required/>
        </div>
        <button type="submit">Cập nhật</button>
        <button type="button" @click="cancelEdit">Hủy</button>
      </form>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      users: [],
      editMode: false,
      editUserData: {
        userId: 0,
        email: '',
        fullname: '',
        phoneNumber: '',
        address: '',
      },
      searchQuery: '',
    };
  },

  computed: {
    filteredUsers() {
      const searchTerm = this.searchQuery.toLowerCase();
      return this.users.filter(user => {
        return (
            // Tìm theo ID
            (user.UserID && user.UserID.toString().includes(searchTerm)) ||
            // Tìm theo Email
            (user.Email && user.Email.toLowerCase().includes(searchTerm)) ||
            // Tìm theo Họ tên
            (user.FullName && user.FullName.toLowerCase().includes(searchTerm)) ||
            // Tìm theo Số điện thoại
            (user.PhoneNumber && user.PhoneNumber.toLowerCase().includes(searchTerm)) ||
            // tìm theo địa chỉ
            (user.Address && user.Address.toLowerCase().includes(searchTerm))

        );
      });
    }
  },

  mounted() {
    this.fetchUsers();
  },

  methods: {
    async fetchUsers() {
      try {
        const token = localStorage.getItem('accessToken');
        const response = await axios.get('http://localhost:2999/api/admin/users', {
          headers: {Authorization: `Bearer ${token}`},
        });
        this.users = response.data;
        console.log(this.users)
      } catch (error) {
        console.error('Error fetching users:', error);
      }
    },

    async removeUser(id) {
      try {
        const token = localStorage.getItem('accessToken');
        await axios.delete(`http://localhost:2999/api/admin/users/${id}`, {
          headers: {Authorization: `Bearer ${token}`},
        });
        this.users = this.users.filter(user => user.UserID !== id);
        alert('Người dùng đã được xóa thành công!');
      } catch (error) {
        console.error('Error removing user:', error);
        alert('Xóa người dùng thất bại.');
      }
    },

    editUser(user) {
      this.editMode = true;
      this.editUserData = {
        userId: user.UserID || 0,
        email: user.Email || '',
        fullname: user.FullName || '',
        phoneNumber: user.PhoneNumber || '',
        address: user.Address || '',
      };
    },

    cancelEdit() {
      this.editMode = false;
      this.editUserData = {userId: 0, email: '', fullname: '', phoneNumber: '', address: ''};
    },

    async updateUser() {
      try {
        const token = localStorage.getItem('accessToken');
        await axios.put('http://localhost:2999/api/admin/users', this.editUserData, {
          headers: {Authorization: `Bearer ${token}`},
        });
        alert('Cập nhật người dùng thành công!');
        this.fetchUsers();
        this.cancelEdit();
      } catch (error) {
        console.error('Error updating user:', error);
        alert('Cập nhật người dùng thất bại.');
      }
    },
  },
};
</script>

<style scoped>
.search-box {
  margin: 20px 0;
}

.search-input {
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
  min-width: 300px;
}

.user-table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 20px;
}

.user-table th,
.user-table td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: left;
}

.user-table th {
  background-color: #f5f5f5;
}

.edit-user-form {
  margin-top: 20px;
  padding: 20px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.edit-user-form div {
  margin-bottom: 10px;
}

.edit-user-form label {
  display: block;
  margin-bottom: 5px;
}

.edit-user-form input {
  width: 100%;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

button {
  padding: 8px 16px;
  margin-right: 8px;
  border: none;
  border-radius: 4px;
  background-color: #4CAF50;
  color: white;
  cursor: pointer;
}

button:hover {
  background-color: #45a049;
}

button[type="button"] {
  background-color: #f44336;
}

button[type="button"]:hover {
  background-color: #da190b;
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

/* Container chứa ô tìm kiếm */
.search-box {
  display: flex;
  justify-content: center; /* Căn giữa ô tìm kiếm */
  align-items: center;
  margin-bottom: 20px; /* Thêm khoảng cách dưới */
}

/* Kiểu cho ô input tìm kiếm */
.search-input {
  width: 100%; /* Chiều rộng 100% */
  max-width: 600px; /* Giới hạn chiều rộng tối đa của ô tìm kiếm */
  padding: 12px 20px; /* Thêm padding cho ô input */
  font-size: 16px; /* Kích thước chữ */
  border: 2px solid #ccc; /* Đường viền mờ */
  border-radius: 30px; /* Bo góc mềm mại */
  outline: none; /* Loại bỏ viền khi ô được chọn */
  transition: all 0.3s ease; /* Hiệu ứng chuyển đổi mượt mà */

}

/* Hiệu ứng khi ô input được focus */
.search-input:focus {
  border-color: rgb(6, 208, 6); /* Đổi màu viền khi focus */
  box-shadow: 0 0 8px rgb(6, 208, 6); /* Thêm bóng khi focus */
}

/* Hiệu ứng khi hover vào ô input */
.search-input:hover {
  border-color: #888; /* Đổi màu viền khi hover */
}
</style>
