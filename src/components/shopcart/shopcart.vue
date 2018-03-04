<template>
  <div>
    <div class="shopcart">
      <div class="content" @click="toggleList">
        <div class="content-left">
          <div class="logo-wrapper">
            <div class="count" v-show="totalCount">{{totalCount}}</div>
            <div class="logo" :class="{'heighlight': totalCount}">
              <span class="icon-shopping_cart" :class="{'heighlight': totalCount}"></span>
            </div>
          </div>
          <div class="price" :class="{'heighlight': totalPrice}">￥{{totalPrice}}</div>
          <div class="desc">另需配送费￥{{deliveryPrice}}</div>
        </div>
        <div class="content-right" @click.stop="pay">
          <div class="pay" :class="payClass">{{payDesc}}</div>
        </div>
      </div>
      <div class="ball-container">
        <div v-for="ball in balls">
          <transition name="drop" @before-enter="beforeEnter" @enter="enter" @after-enter="afterEnter">
            <div class="ball" v-show="ball.show">
              <div class="inner inner-hook"></div>
            </div>
          </transition>
        </div>
      </div>
      <transition name="fold">
        <div class="shopcart-list" v-show="listShow">
          <div class="list-header">
            <h1 class="title">购物车</h1>
            <span class="empty" @click="empty">清空</span>
          </div>
          <div class="list-content" ref="listContent">
            <ul>
              <li class="food border-1px" v-for="food in selectFoods">
                <div class="left">
                  <span class="food-name">{{food.name}}</span>
                </div>
                <div class="right">
                  <div class="price">
                    <span class="rmb">￥</span>
                    <span class="total-price">{{food.price * food.count}}</span>
                  </div>
                  <div class="cartcontrol-wrapper">
                    <cartcontrol @cartadd="drop" :food="food"></cartcontrol>
                  </div>
                </div>
              </li>
            </ul>
          </div>
        </div>
      </transition>
    </div>
    <transition name="fade">
      <div class="list-mask" v-show="listShow" @click="toggleList"></div>
    </transition>
  </div>
</template>

<script type="text/ecmascript-6">
  import BScroll from 'better-scroll';
  import cartcontrol from '../../components/cartcontrol/cartcontrol';

  export default {
    props: {
      selectFoods: {
        type: Array,
        default() {
          return [
            {
              price: 10,
              count: 1
            }
          ];
        }
      },

      deliveryPrice: {
        type: Number,
        default: 0
      },

      minPrice: {
        type: Number,
        default: 0
      }
    },

    data() {
      return {
        balls: [
          {
            show: false
          },
          {
            show: false
          },
          {
            show: false
          },
          {
            show: false
          },
          {
            show: false
          }
        ],
        dropBalls: [],
        fold: true
      };
    },

    computed: {
      totalPrice() {
        let total = 0;
        this.selectFoods.forEach((food) => {
          total += food.price * food.count;
        });
        return total;
      },

      totalCount() {
        let count = 0;
        this.selectFoods.forEach((food) => {
          count += food.count;
        });
        return count;
      },

      payDesc() {
        if (this.totalPrice === 0) {
          return `￥${this.minPrice}元起送`;
        } else if (this.totalPrice < this.minPrice) {
          let diff = this.minPrice - this.totalPrice;
          return `还差￥${diff}元起送`;
        } else {
          return `去结算`;
        }
      },

      payClass() {
        if (this.totalPrice < this.minPrice) {
          return 'not-enough';
        } else {
          return 'enough';
        }
      },

      listShow() {
        if (!this.totalCount) {
          this.fold = true;
          return false;
        }
        let show = !this.fold;
        if (show) {
          this.$nextTick(() => {
            if (!this.scroll) {
              this.scroll = new BScroll(this.$refs.listContent, {
                click: true
              });
            } else {
              this.scroll.refresh();
            }
          });
        }
        return show;
      }
    },

    methods: {
      drop(el) {
        for (let i = 0; i < this.balls.length; i++) {
          let ball = this.balls[i];
          if (!ball.show) {
            ball.show = true;
            ball.el = el;
            this.dropBalls.push(ball);
            return;
          }
        }
      },

      beforeEnter(el) {
        let count = this.balls.length;
        while (count--) {
          let ball = this.balls[count];
          if (ball.show) {
            let rect = ball.el.getBoundingClientRect();
            let x = rect.left - 32;
            let y = -(window.innerHeight - rect.top - 22);
            el.style.display = '';
            el.style.webkitTransform = `translate3d(0, ${y}px, 0)`;
            el.style.transform = `translate3d(0, ${y}px, 0)`;
            let inner = el.getElementsByClassName('inner-hook')[0];
            inner.style.webkitTransform = `translate3d(${x}px, 0, 0)`;
            inner.style.transform = `translate3d(${x}px, 0, 0)`;
          }
        }
      },

      enter(el) {
        /* eslint-disable no-unused-vars */
        let rf = el.offsetHeight;
        this.$nextTick(() => {
          el.style.webkitTransform = 'translate3d(0, 0, 0)';
          el.style.transform = 'translate3d(0, 0, 0)';
          let inner = el.getElementsByClassName('inner-hook')[0];
          inner.style.webkitTransform = 'translate3d(0, 0, 0)';
          inner.style.transform = 'translate3d(0, 0, 0)';
        });
      },

      afterEnter(el) {
        let ball = this.dropBalls.shift();
        if (ball) {
          ball.show = false;
          el.style.display = 'none';
        }
      },

      toggleList() {
        if (!this.totalCount) {
          return;
        }
        this.fold = !this.fold;
      },

      empty() {
        this.selectFoods.forEach((food) => {
          food.count = 0;
        });
      },

      pay () {
        if (this.totalPrice < this.minPrice) {
          return;
        }
        window.alert(`支付${this.totalPrice}元`);
      }
    },

    components: {
      cartcontrol
    }
  };
</script>

<style lang="stylus" rel="stylesheet/stylus">
  @import "../../common/stylus/mixin.styl"

  .shopcart
    position: fixed
    bottom: 0
    left: 0
    width: 100%
    height: 48px
    z-index: 50
    .content
      display: flex
      color: rgb(128, 133, 138)
      background-color: #141d27
      .content-left
        flex: 1
        .count
          display: inline-block
          position: absolute
          top: 0
          right: 0
          width: 24px
          height: 16px
          text-align: center
          font-size: 9px
          color: rgb(255, 255, 255)
          line-height: 16px
          background-color: rgb(240, 20, 20)
          border-radius: 16px
          box-shadow: 0 4px 8px 0 rgba(0, 0, 0, .4)
        .logo-wrapper
          display: inline-block
          position: relative
          top: -10px
          margin: 0 12px
          padding: 6px
          width: 56px
          height: 56px
          background-color: #141d27
          box-sizing: border-box
          vertical-align: top
          border-radius: 50%
          .logo
            width: 100%
            height: 100%
            text-align: center
            background-color: rgb(43, 52, 60)
            border-radius: 50%
            &.heighlight
              background-color: rgb(0, 160, 220)
            .icon-shopping_cart
              font-size: 24px
              line-height: 44px
              &.heighlight
                color: #fff
        .price
          display: inline-block
          margin-top: 12px
          padding-right: 12px
          line-height: 24px
          font-size: 18px
          font-weight: 700
          border-right: 1px solid rgba(255, 255, 255, .1)
          &.heighlight
            color: #fff
        .desc
          display: inline-block
          vertical-align: top
          margin: 12px 0 0 12px
          line-height: 24px
          font-size: 12px
      .content-right
        flex: 0 0 105px
        width: 105px
        .pay
          height: 48px
          line-height: 48px
          font-size: 12px
          font-weight: 700
          text-align: center
          &.not-enough
            background-color: rgb(43, 51, 59)
          &.enough
            background-color: #00b43c
            color: #fff
    .ball-container
      .ball
        position: fixed
        left: 32px
        bottom: 22px
        z-index: 200
        transition: all .4s cubic-bezier(.49, -.29, .75, .41)
        .inner
          width: 16px
          height: 16px
          border-radius: 50%
          background-color: rgb(0, 160, 220)
          transition: all .4s linear
    .shopcart-list
      position: absolute
      top: 0
      left: 0
      z-index: -1
      width: 100%
      color: rgb(7, 17, 27)
      transition: all .4s
      transform: translate3d(0, -100%, 0)
      &.fold-enter-active, &.fold-leave-active
        transition: all .4s
      &.fold-enter, &.fold-leave-to
        transform: translate3d(0, 0, 0);
      .list-header
        padding: 0 18px
        height: 40px
        line-height: 40px
        background-color: #f3f5f7
        border-bottom: 1px solid #dbdee1
        .title
          float: left
          font-size: 14px
          line-height: 40px
        .empty
          float: right
          font-size: 12px
          color: rgb(0, 160, 220)
      .list-content
        padding: 0 18px
        max-height: 217px
        overflow: hidden
        font-size: 0
        background-color: #fff
        .food
          height: 48px
          border-1px(rgba(7, 17, 27, .1))
          &:last-child
            border-none()
          .left
            display: inline-block
            .food-name
              font-size: 14px
              line-height: 48px
              color: rgb(7, 17, 27)
          .right
            display: inline-block
            float: right
            .price
              display: inline-block
              margin: 17px 12px 0 0
              color: rgb(160, 20, 20)
              .rmb
                font-size: 10px
                font-weight: normal
              .total-price
                font-size: 14px
                font-weight: 700
            .cartcontrol-wrapper
              display: inline-block
              margin-top: 7px
              vertical-align: top

  .list-mask
    position: fixed
    top: 0
    left: 0
    width: 100%
    height: 100%
    z-index: 40
    background-color: rgba(7, 17, 27, .6)
    backdrop-filter: blur(10px)
    opacity: 1
    &.fade-enter-active, &.fade-leave-active
      transition: all .4s
    &.fade-enter, &.fade-leave-to
      opacity: 0
      background-color: rgba(7, 17, 27, 0)
</style>
