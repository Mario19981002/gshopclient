<template>
  <section class="order">
    <HeaderTop title="订单列表" />
    <section class="order_no_login" v-show="!username">
      <img src="./images/person.png" />
      <h3>登录后查看外卖订单</h3>
      <button @click="$router.push('/login')">立即登陆</button>
    </section>
    <section class="order_in_login" v-show="username">
      <div class="wrapper">
        <div class="order_wrapper">
          <div class="order_list" v-for="(cart,index) in carts" :key="index">
            <div class="order_item">
              <div class="order_header">{{cart.shop}}</div>
              <div class="order_body clearfix" v-for="(food,index) in cart.cartFood" :key="index">
                <div class="order_body_shop">{{food.name}}</div>
                <div class="order_body_num">×{{food.count}}</div>
                <div class="order_body_tot">{{food.price}}元</div>
              </div>
              <div class="order_footer clearfix">
                <div class="order_footer_date">{{cart.createdAt}}</div>
                <div class="order_footer_total">{{cart.totalPrice}}元</div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>
  </section>
</template>

<script>
import HeaderTop from "../../components/HeaderTop/HeaderTop.vue";
import BScroll from "better-scroll";
import { mapState } from "vuex";
export default {
  data() {
    return {
      carts: [],
      scroll: null,
      username: window.sessionStorage.getItem("username")
    };
  },
  components: {
    HeaderTop
  },
  mounted() {
    const { username } = this;
    const query = Bmob.Query("cart");
    const query1 = query.equalTo("userInfo", "==", username);
    // const query1 = query.equalTo("userInfo", "==", "abc");
    query.or(query1);
    query.find().then(res => {
      this.carts = res;
    });
    new BScroll(".wrapper", {
      click: true
    });
  }
};
</script>

<style lang="stylus" rel="stylesheet/stylus">
@import '../../common/stylus/mixins.styl';

/* 清除浮动 */
.clearfix::before, .clearfix::after {
  content: '';
  display: table;
}

.clearfix::after {
  clear: both;
}

.order { // 订单
  width: 100%;

  .header {
    background-color: rgb(0, 141, 225);
    position: fixed;
    z-index: 100;
    left: 0;
    top: 0;
    width: 100%;
    height: 45px;

    .header_search {
      position: absolute;
      left: 15px;
      top: 50%;
      transform: translateY(-50%);
      width: 10%;
      height: 50%;

      .icon-sousuo {
        font-size: 25px;
        color: #fff;
      }
    }

    .header_title {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      width: 50%;
      color: #fff;
      text-align: center;

      .header_title_text {
        font-size: 20px;
        color: #fff;
        display: block;
      }
    }

    .header_login {
      font-size: 14px;
      color: #fff;
      position: absolute;
      right: 15px;
      top: 50%;
      transform: translateY(-50%);

      .header_login_text {
        color: #fff;
      }
    }
  }

  .order_no_login {
    padding-top: 140px;
    width: 60%;
    margin: 0 auto;
    text-align: center;

    >img {
      display: block;
      width: 100%;
      height: 30%;
    }

    >h3 {
      padding: 10px 0;
      font-size: 17px;
      color: #6a6a6a;
    }

    >button {
      display: inline-block;
      background: rgb(0, 141, 225);
      font-size: 14px;
      color: #fff;
      border: 0;
      outline: none;
      border-radius: 5px;
      padding: 10px 20px;
    }
  }

  .order_in_login {
    padding-top: 45px;
    margin-bottom: 60px;

    .wrapper {
      height: 525px;
    }

    .order_list {
      border: 1px solid #eee;
      border-radius: 4px;
      background-color: #eee;
      margin: 10px 10px;
    }

    .order_item {
      padding: 10px;

      .order_header {
        height: 30px;
        line-height: 30px;
        text-align: center;
        margin-bottom: 10px;
      }

      .order_body {
        padding: 10px 0;

        .order_body_shop {
          float: left;
        }

        .order_body_num {
          float: left;
          font-size: 14px;
          padding-left: 10px;
        }

        .order_body_tot {
          float: right;
        }
      }

      .order_footer {
        padding: 10px 0;
        margin-top: 10px;

        .order_footer_date {
          float: left;
        }

        .order_footer_total {
          float: right;
        }
      }
    }
  }
}
</style>