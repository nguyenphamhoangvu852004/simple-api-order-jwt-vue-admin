<template>
  <div class="container">
    <div class="cart-label">
      <div class="label-product">Sản Phẩm</div>
      <div class="label-price">Đơn Giá</div>
      <div class="label-quantity">Số Lượng</div>
      <div class="label-total">Số Tiền</div>
      <div class="label-action">Thao Tác</div>
    </div>
    <section class="cart-item" v-for="(item, index) in cartItems" :key="index">
      <div class="cart-info">
        <img class="cart-img" :src="getImageUrl(item.image)" alt="Product Image" />
        <div class="cart-name">{{ item.name }}</div>
      </div>
      <div class="cart-price">
        <span class="">{{ formatCurrency(item.price) }}</span>
      </div>
      <div class="cart-quantity">
        <div class="cart-quantity-counter">
          <button class="cart-quantity-btn" @click="decreaseQuantity(index)">-</button>
          <span class="cart-quantity-value">{{ item.quantity }}</span>
          <button class="cart-quantity-btn" @click="increaseQuantity(index)">+</button>
        </div>
      </div>
      <div class="cart-total">
        {{ formatCurrency(item.price * item.quantity) }}
      </div>
      <div class="cart-action">
        <a @click="removeItem(index)">Xóa</a>
      </div>
    </section>
    <p style="text-align: right">Thanh toán khi nhận hàng</p>
    <section class="cart-order">
      <div class="total">Tổng tiền: <span class="total-price">{{ formatCurrency(totalPrice) }}</span></div>
      <button class="cart-button" @click="checkout">
        <span>Mua hàng</span>
      </button>
    </section>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      cartItems: JSON.parse(localStorage.getItem('cartItems')) || [],
    };
  },
  computed: {
    totalPrice() {
      return this.cartItems.reduce((total, item) => total + item.price * item.quantity, 0);
    },
  },
  methods: {
    formatCurrency(value) {
      return new Intl.NumberFormat('vi-VN', {
        style: 'currency',
        currency: 'VND',
      }).format(value);
    },
    getImageUrl(imageName) {
     // Kiểm tra nếu imageName là null hoặc không có, thì trả về ảnh mặc định
      if (!imageName) {
        return require('@/assets/image/default-product-image.jpg'); // Hình ảnh mặc định
      }
      // Nếu có ảnh, thì trả về đường dẫn ảnh
      return require(`@/assets/image/${imageName}`);
    },
    increaseQuantity(index) {
      this.cartItems[index].quantity++;
      this.updateLocalStorage();
    },
    decreaseQuantity(index) {
      if (this.cartItems[index].quantity > 1) {
        this.cartItems[index].quantity--;
        this.updateLocalStorage();
      }
    },
    removeItem(index) {
      this.cartItems.splice(index, 1);
      this.updateLocalStorage();
    },
    updateLocalStorage() {
      localStorage.setItem('cartItems', JSON.stringify(this.cartItems));
    },
    async checkout() {
      if (!await this.checkUserDetail()) {
        alert("Cập nhật thông tin người dùng trước khi đặt hàng!");
        await this.$router.push('/user');
      } else {
        try {
          const token = localStorage.getItem('accessToken');
          const orderData = {
            items: this.cartItems.map(item => ({
              productId: item.id,
              productSizeId: item.size,
              quantity: item.quantity,
            })),
          };

          console.log(orderData);
          await axios.post('http://localhost:2999/api/orders', orderData, {
            headers: {
              Authorization: `Bearer ${token}`,
            },
          });

          alert('Đặt hàng thành công!');
          this.cartItems = [];
          this.updateLocalStorage();
        } catch (error) {
          console.error('Đặt hàng thất bại:', error);
          alert('Có lỗi xảy ra khi đặt hàng. Vui lòng thử lại.');
        }
      }
    },
    async checkUserDetail() {
      try {
        const token = localStorage.getItem('accessToken');
        const response = await axios.get(`http://localhost:2999/api/users`, {
          headers: {
            Authorization: `Bearer ${token}`,
          }
        });

        if (!response.data) {
          console.error("User  not found");
          return false;
        }

        const { fullname, address, phoneNumber } = response.data;
        console.log(response.data)
        return !(!fullname || !address || !phoneNumber);
      } catch (error) {
        console.error('Error check user:', error);
        return false;
      }
    },
  },
};
</script>


<style scoped>

.total-price{
  color: #EE4D2D;
  font-size: 20px;
}

.cart-order{
  display: flex;
  justify-content: right;
  align-content: center;
}

.total{
  margin-right: 15px;
  align-content: center;
}

.cart-button{
  background: #EE4D2D;
  width: 170px;
  height: 40px;
  border: none;
  color: white;
  border-radius: 1px;
}

body{
  box-sizing: border-box;
}

.container{
  width: 80%;
  margin: 0 auto;
}

.cart-label{
  align-items: center;
  background: #fff;
  border-radius: .125rem;
  border-radius: 3px;
  box-shadow: 0 1px 1px 0 rgba(0,0,0,.05);
  color: #888;
  display: flex;
  font-size: 14px;
  height: 55px;
  margin-bottom: 12px;
  overflow: hidden;
  padding: 0 20px;
}

input[type=checkbox], input[type=radio]{
  box-sizing: border-box;
  padding: 0;
}

.label-product{
  width: 46.27949%;
}

.label-price{
  width: 16.66667%;
}

.label-quantity{
  width: 16.66667%;
}

.label-total{
  width: 16.66667%;
}

.label-action{
  width: 16.66667%;
}

.cart-item{
  margin: 20px 0;
}

.cart-info{
  width: 46.27949%;
  display: flex;
  align-content: center;
  justify-content: left;
}

.cart-item{
  align-items: center;
  background: #fff;
  border-radius: .125rem;
  border-radius: 3px;
  box-shadow: 0 1px 1px 0 rgba(0,0,0,.05);
  color: #888;
  display: flex;
  font-size: 14px;
  height: 55px;
  margin-bottom: 12px;
  overflow: hidden;
  padding: 0 20px;
}

.cart-img{
  width:50px;
}

.cart-name{
  padding-left: 8px;
  display: -webkit-box;
  height: 90%;
  margin: auto 0;
}

.cart-price{
  width: 16.66667%;
}

.cart-quantity{
  width: 16.66667%;
}

.cart-total{
  width: 16.66667%;
}

.cart-action{
  width: 16.66667%;
  cursor: pointer;
}

.cart-action:hover{
  color: red;
}

.cart-quantity-counter {
  display: inline-flex;
  align-items: center;
  width: 104px;
  border: 1px solid #ddd;
  border-radius: 4px;
  overflow: hidden;
}

.cart-quantity-btn {
  width: 30px;
  height: 30px;
  border: none;
  background: none;
  cursor: pointer;
  font-size: 20px;
  color: #888;
  outline: none;
}

.cart-quantity-value {
  width: 40px;
  text-align: center;
  font-size: 16px;
  color: #333;
  border-left: 1px solid #ddd;
  border-right: 1px solid #ddd;
}
</style>
