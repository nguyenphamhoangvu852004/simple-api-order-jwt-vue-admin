<template>
  <nav class="navbar navbar-expand-lg bg-body-tertiary">
    <div class="container-fluid">
      <router-link to="/">
        <a class="navbar-brand">
          <img src="../assets/image/icon/online-shopping.png" alt="logo">
        </a>
      </router-link>
      <div class="collapse navbar-collapse" id="navbarSupportedContent">
        <ul class="navbar-nav me-auto mb-2 mb-lg-0">
          <router-link to="/">
            <li class="nav-item">
              <a class="nav-link">Trang chủ</a>
            </li>
          </router-link>
          <li class="nav-item" v-if="!isLoggedIn">
            <router-link to="/signup">
              <a class="nav-link">Đăng ký</a>
            </router-link>
          </li>
          <li class="nav-item" v-if="!isLoggedIn">
            <router-link to="/login">
              <a class="nav-link">Đăng nhập</a>
            </router-link>
          </li>
          <router-link to="/user">
          <li class="nav-item" v-if="isLoggedIn">
            <a class="nav-link">{{ userEmail }}</a>
          </li>
          </router-link>
          <li class="nav-item" v-if="isLoggedIn" style="cursor: pointer">
            <a class="nav-link" @click="logout">Đăng xuất</a>
          </li>
          <router-link to="/cart" style="cursor: pointer;
            margin-left: 8px;
            font-size: 26px;
            color: black">
          <li class="nav-item" style="cursor: pointer">
            <i class="fa-solid fa-cart-shopping" style="color: #E65A30"></i>
          </li>
          </router-link>
        </ul>
      </div>
    </div>
  </nav>
</template>

<script>
export default {
  data() {
    return {
      userEmail: '',
      isLoggedIn: false,
    };
  },
  created() {
    this.checkLoginStatus();
    this.startWatchingLocalStorage();
  },
  beforeUnmount() {
    clearInterval(this.localStorageInterval); // Dọn dẹp interval khi component bị hủy
  },
  methods: {
    checkLoginStatus() {
      this.userEmail = localStorage.getItem('userEmail');
      this.isLoggedIn = !!this.userEmail;
    },
    startWatchingLocalStorage() {
      this.localStorageInterval = setInterval(() => {
        const email = localStorage.getItem('userEmail');
        if (email !== this.userEmail) {
          this.userEmail = email;
          this.isLoggedIn = !!email;
        }
      }, 1000); // Kiểm tra mỗi giây
    },
    logout() {
      localStorage.clear()
      this.isLoggedIn = false;
      this.userEmail = '';
      this.$router.push('/');
    },
  },
};
</script>

<style scoped>
.nav-item{
  font-weight: bold;
}
.navbar {
  margin-bottom: 30px;
}
body {
  font-family: Mulish, sans-serif;
}
.navbar-brand img {
  width: 60px;
  height: auto;
}
a {
  text-decoration: none;
}
</style>