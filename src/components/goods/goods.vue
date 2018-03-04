<template>
  <div class="goods">
    <div class="menu-wrapper" ref="menuWrapper">
      <ul>
        <li class="menu-item" v-for="(item, index) in goods" :class="{'current': currentIndex === index}"
            @click="selectMenu(index, $event)">
          <span class="text border-1px"><span class="icon" :class="classMap[item.type]" v-show="item.type > 0"></span>{{item.name}}</span>
        </li>
      </ul>
    </div>
    <div class="foods-wrapper" ref="foodsWrapper">
      <ul>
        <li class="food-list food-list-hook" v-for="item in goods">
          <h1 class="title">{{item.name}}</h1>
          <ul>
            <li class="food-item border-1px" v-for="food in item.foods" @click="selectFood(food, $event)">
              <div class="icon">
                <img :src="food.icon" width="57" height="57">
              </div>
              <div class="content">
                <div class="name">{{food.name}}</div>
                <div class="desc" v-show="food.description">{{food.description}}</div>
                <div class="extra">
                  <span class="sell-count">月售{{food.sellCount}}份</span><span>好评率{{food.rating}}%</span>
                </div>
                <div class="price">
                  <span class="now-price"><span class="RMB">￥</span>{{food.price}}</span>
                  <span class="old-price" v-show="food.oldPrice"><span class="RMB">￥</span>{{food.oldPrice}}</span>
                </div>
                <div class="cartcontrol-wrapper">
                  <cartcontrol @cartadd="_drop" :food="food"></cartcontrol>
                </div>
              </div>
            </li>
          </ul>
        </li>
      </ul>
    </div>
    <shopcart ref="shopcart" :deliveryPrice="seller.deliveryPrice" :minPrice="seller.minPrice"
              :selectFoods="selectFoods"></shopcart>
    <food :food="selectedFood" ref="food" @cartadd="_drop"></food>
  </div>
</template>

<script type="text/ecmascript-6">
  import BScroll from 'better-scroll';
  import shopcart from '../../components/shopcart/shopcart';
  import cartcontrol from '../../components/cartcontrol/cartcontrol';
  import food from '../../components/food/food';

  const ERR_OK = 0;

  export default {
    props: {
      seller: {
        type: Object
      }
    },

    data() {
      return {
        goods: [],
        listHeight: [],
        scrollY: 0,
        selectedFood: {}
      };
    },

    created() {
      this.classMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee'];
      this.$http.get('/api/goods').then((response) => {
        response = response.body;
        if (response.errno === ERR_OK) {
          this.goods = response.data;
          this.$nextTick(() => {
            this._initScroll();
            this._calculateHeight();
          });
        }
      });
    },

    computed: {
      currentIndex() {
        for (let i = 0; i < this.listHeight.length; i++) {
          let height1 = this.listHeight[i];
          let height2 = this.listHeight[i + 1];
          if ((this.scrollY >= height1 && this.scrollY < height2) || !height2) {
            return i;
          }
        }
        return 0;
      },

      selectFoods() {
        let foods = [];
        this.goods.forEach((good) => {
          good.foods.forEach((food) => {
            if (food.count) {
              foods.push(food);
            }
          });
        });
        return foods;
      }
    },

    methods: {
      _drop(target) {
        // 优化异步执行下落动画
        this.$nextTick(() => {
          this.$refs.shopcart.drop(target);
        });
      },

      _initScroll() {
        this.menuScroll = new BScroll(this.$refs.menuWrapper, {
          click: true
        });

        this.foodsScroll = new BScroll(this.$refs.foodsWrapper, {
          click: true,
          probeType: 3
        });

        this.foodsScroll.on('scroll', (pos) => {
          this.scrollY = Math.abs(Math.round(pos.y));
        });
      },

      _calculateHeight() {
        let foodList = this.$refs.foodsWrapper.getElementsByClassName('food-list-hook');
        let height = 0;
        this.listHeight.push(height);
        for (let i = 0; i < foodList.length; i++) {
          let item = foodList[i];
          height += item.clientHeight;
          this.listHeight.push(height);
        }
      },

      selectMenu(index, event) {
        if (!event._constructed) {
          return;
        }
        let foodList = this.$refs.foodsWrapper.getElementsByClassName('food-list-hook');
        let el = foodList[index];
        this.foodsScroll.scrollToElement(el, 300);
      },

      selectFood(food, event) {
        if (!event._constructed) {
          return;
        }
        this.selectedFood = food;
        this.$refs.food.show();
      }
    },

    components: {
      shopcart,
      cartcontrol,
      food
    }
  };
</script>

<style lang="stylus" rel="stylesheet/stylus">
  @import "../../common/stylus/mixin.styl"

  .goods
    display: flex
    position: absolute
    top: 174px
    bottom: 47px
    width: 100%
    overflow: hidden
    .menu-wrapper
      flex: 0 0 80px
      width: 80px
      background: #f3f5f7
      .menu-item
        display: table
        padding: 0 12px
        width: 56px
        height: 54px
        line-height: 14px
        &.current
          position: relative
          margin-top: -1px
          z-index: 10
          font-weight: 700px
          background-color: #ffffff
          .text
            border-none()
        &:last-child
          .text
            border-none()
        .text
          display: table-cell
          width: 56px
          vertical-align: middle
          font-size: 12px
          border-1px(rgba(7, 17, 27, .1))
          .icon
            display: inline-block
            margin-right: 2px
            width: 12px
            height: 12px
            vertical-align: top
            background-size: 12px 12px
            background-repeat: no-repeat
            &.decrease
              bg-image('decrease_3');
            &.discount
              bg-image('discount_3');
            &.special
              bg-image('special_3');
            &.invoice
              bg-image('invoice_3');
            &.guarantee
              bg-image('guarantee_3');

    .foods-wrapper
      flex: 1
      .title
        padding-left: 14px
        border-left: 2px solid #d9dde1
        height: 26px
        font-size: 12px
        font-weight: 700
        line-height: 26px
        color: rgb(147, 153, 159)
        background-color: #f3f5f7
      .food-item
        display: flex
        margin: 18px
        padding-bottom: 18px
        border-1px(rgba(7, 17, 27, .1))
        &:last-child
          border-none()
          padding-bottom: 0
        .icon
          flex: 0 0 57px
          width: 57px
          height: 57px
        .content
          flex: 1
          position: relative
          margin-left: 10px
          .name
            color: rgb(7, 17, 27)
            margin-top: 2px
            font-size: 14px
          .desc
            margin-top: 8px
            font-size: 10px
            color: rgb(147, 153, 159)
          .extra
            margin-top: 8px
            font-size: 10px
            color: rgb(147, 153, 159)
            .sell-count
              display: inline-block
              margin-right: 12px
          .price
            font-size: 0
            .now-price, .old-price
              display: inline-block
              font-weight: 700
              line-height: 24px
              .RMB
                font-size: 10px
                font-weight: normal
            .now-price
              vertical-align: top
              color: rgb(240, 20, 20)
              font-size: 14px
            .old-price
              font-size: 10px
              color: rgb(147, 153, 159)
              text-decoration: line-through
          .cartcontrol-wrapper
            position: absolute
            right: 0
            bottom: 0
</style>
