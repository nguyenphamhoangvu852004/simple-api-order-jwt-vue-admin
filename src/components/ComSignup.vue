<template>
  <div class="container">
    <h1>Đăng Ký</h1>
    <form id="registrationForm" @submit.prevent="handleSignup">
      <label for="email">Email</label>
      <input type="email" id="email" v-model="email" required />

      <label for="phone">Số điện thoại</label>
      <input type="text" id="phone" v-model="phoneNumber" required />

      <label for="password">Mật khẩu</label>
      <input type="password" id="password" v-model="password" required />

      <label for="confirmPassword">Xác nhận mật khẩu</label>
      <input type="password" id="confirmPassword" v-model="confirmPassword" required />

      <button type="submit">Đăng Ký</button>
      <div id="message">{{ message }}</div>
    </form>
    <p>Đã có tài khoản? <router-link to="/">Đăng Nhập</router-link></p>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      email: '',
      phoneNumber: '',
      password: '',
      confirmPassword: '',
      message: ''
    };
  },
  methods: {
    async handleSignup() {
      // Kiểm tra xác nhận mật khẩu
      if (this.password !== this.confirmPassword) {
        this.message = "Mật khẩu không khớp. Vui lòng thử lại.";
        return;
      }

      // Gọi API để đăng ký người dùng
      try {
        // eslint-disable-next-line no-debugger
        debugger
        const response = await this.registerUser ({
          email: this.email,
          phoneNumber: this.phoneNumber,
          password: this.password,
          passwordConfirm: this.confirmPassword
        });

        // Kiểm tra phản hồi từ API
        if (response.data.success) {
          alert("Đăng ký thành công!")
          await this.$router.push('/login');
        } else {
          this.message = response.data.message || "Đăng ký không thành công. Vui lòng thử lại.";
        }
      } catch (error) {
        console.error('Có lỗi xảy ra khi đăng ký:', error);
        this.message = "Có lỗi xảy ra. Vui lòng thử lại sau.";
      }
    },
    async registerUser (userData) {
      return await axios.post('http://localhost:2999/api/auth/register', userData);
    }
  }
};
</script>



<style scoped>
body {
  font-family: Mulish, sans-serif;
  background-color: #f4f4f4;
  margin: 0;
  padding: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}

.container {
  width: 80%;
  max-width: 400px;
  background: #fff;
  padding: 20px;
  margin: 0 auto;
  border-radius: 10px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

h1 {
  text-align: center;
  color: #333;
}

form {
  display: flex;
  flex-direction: column;
}

label {
  text-align: left;
  margin-top: 10px;
  color: #333;
}

input {
  padding: 10px;
  margin-top: 5px;
  border: 1px solid #ccc;
  border-radius: 5px;
}

button {
  margin-top: 20px;
  padding: 10px;
  background-color: #4CAF50;
  color: #fff;
  border: none;
  border-radius: 5px;
  font-size: 1em;
  cursor: pointer;
}

button:hover {
  background-color: #45a049;
}

#message {
  margin-top: 20px;
  text-align: center;
  font-size: 1em;
  color: red;
}

</style>
