<template>
  <div>
    <transition name="move">
      <div class="food" v-show="showFlag" ref="food">
        <div class="food-content">
          <div class="image-header">
            <img :src="food.image">
            <div class="back" @click="hide">
              <span class="icon-arrow_lift"></span>
            </div>
          </div>
          <div class="content">
            <h1 class="title">{{food.name}}</h1>
            <div class="detail">
              <span class="sell-count">月售{{food.sellCount}}</span><span class="rating">好评率{{food.rating}}%</span>
            </div>
            <div class="price">
              <div class="now-price">
                <span class="rmb">￥</span>{{food.price}}
              </div>
              <div class="old-price" v-show="food.oldPrice">￥{{food.oldPrice}}</div>
            </div>
            <div class="cartcontrol-wrapper">
              <cartcontrol @cartadd="addFood" :food="food"></cartcontrol>
            </div>
            <transition name="fade">
              <div class="buy" v-show="!food.count ||food.count === 0" @click.stop="addFirst">加入购物车</div>
            </transition>
          </div>
          <split v-show="food.info"></split>
          <div class="info" v-show="food.info">
            <h1 class="title" v-show="food.info">商品介绍</h1>
            <div class="text">{{food.info}}</div>
          </div>
          <split></split>
          <div class="food-rating">
            <h1 class="title">商品评价</h1>
            <ratingselect :select-type="selectType" :only-content="onlyContent" :desc="desc" :ratings="food.ratings"
                          @select="selectRating" @toggle="toggleContent"></ratingselect>
            <div class="rating-wrapper">
              <ul v-show="food.ratings && food.ratings.length">
                <li class="rating-item border-1px" v-for="rating in food.ratings"
                    v-show="needShow(rating.rateType, rating.text)">
                  <div class="user">
                    <span class="name">{{rating.username}}</span>
                    <img class="avatar" width="12" height="12" :src="rating.avatar">
                  </div>
                  <div class="time">{{rating.rateTime | formatDate}}</div>
                  <p class="text">
                    <span
                      :class="{'icon-thumb_up': rating.rateType===0, 'icon-thumb_down': rating.rateType===1}"></span>{{rating.text}}
                  </p>
                </li>
              </ul>
              <div class="no-rating" v-show="!food.ratings || !food.ratings.length">暂无评价</div>
            </div>
          </div>
        </div>
      </div>
    </transition>
  </div>
</template>

<script type="text/ecmascript-6">
  import BScroll from 'better-scroll';
  import Vue from 'vue';
  import cartcontrol from '../../components/cartcontrol/cartcontrol';
  import split from '../../components/split/split';
  import ratingselect from '../../components/ratingselect/ratingselect';
  import {formatDate} from '../../common/js/date';

  // const POSITIVE = 0;
  // const NEGATIVE = 1;
  const ALL = 2;

  export default {
    props: {
      food: {
        type: Object
      }
    },

    data() {
      return {
        showFlag: false,
        selectType: ALL,
        onlyContent: true,
        desc: {
          all: '全部',
          positive: '推荐',
          negative: '吐槽'
        }
      };
    },

    methods: {
      show() {
        this.showFlag = true;
        this.selectType = ALL;
        this.onlyContent = true;
        this.$nextTick(() => {
          if (!this.scroll) {
            this.scroll = new BScroll(this.$refs.food, {
              click: true
            });
          } else {
            this.scroll.refresh();
          }
        });
      },

      hide() {
        this.showFlag = false;
      },

      addFirst(event) {
        if (!event._constructed) {
          return;
        }
        this.$emit('cartadd', event.target);
        Vue.set(this.food, 'count', 1);
      },

      addFood(target) {
        this.$emit('cartadd', target);
      },

      selectRating(type) {
        this.selectType = type;
        this.$nextTick(() => {
          this.scroll.refresh();
        });
      },

      toggleContent() {
        this.onlyContent = !this.onlyContent;
        this.$nextTick(() => {
          this.scroll.refresh();
        });
      },

      needShow(type, text) {
        if (this.onlyContent && !text) {
          return false;
        }
        if (this.selectType === ALL) {
          return true;
        } else {
          return type === this.selectType;
        }
      }
    },

    filters: {
      formatDate(time) {
        let date = new Date(time);
        return formatDate(date, 'yyyy-MM-dd hh:mm');
      }
    },

    components: {
      cartcontrol,
      split,
      ratingselect
    }
  };
</script>

<style lang="stylus" rel="stylesheet/stylus">
  @import "../../common/stylus/mixin.styl"

  .food
    position: fixed
    left: 0
    top: 0
    bottom: 48px
    z-index: 30
    width: 100%
    background-color: #fff
    transform: translate3d(0, 0, 0)
    &.move-enter-active, &.move-leave-active
      transition: all .4s
    &.move-enter, &.move-leave-to
      transform: translate3d(100%, 0, 0)
    .image-header
      position: relative
      width: 100%
      height: 0
      padding-top: 100%
      img
        position: absolute
        top: 0
        left: 0
        width: 100%
        height: 100%
      .back
        position: absolute
        top: 10px
        left: 0
        .icon-arrow_lift
          display: block
          padding: 10px
          font-size: 20px
          color: #fff
    .content
      position: relative
      padding: 18px
      background-color: #fff
      .title
        margin-bottom: 8px
        font-size: 14px
        font-weight: 700
        color: rgb(7, 17, 27)
        line-height: 14px
      .detail
        margin-bottom: 18px
        font-size: 10px
        color: rgb(147, 153, 159)
        line-height: 10px
        .sell-count
          display: inline-block
          margin-right: 12px
        .rating
          display: inline-block
      .price
        font-size: 0
        .now-price, .old-price
          display: inline-block
          font-weight: 700
          line-height: 24px
        .now-price
          margin-right: 10px
          font-size: 14px
          color: rgb(240, 20, 20)
          .rmb
            font-size: 10px
            font-weight: normal
        .old-price
          font-size: 10px
          color: rgb(147, 153, 159)
          text-decoration: line-through
      .cartcontrol-wrapper
        position: absolute
        right: 12px
        bottom: 12px
      .buy
        position: absolute
        right: 18px
        bottom: 18px
        z-index: 10
        padding: 6px 12px
        height: 24px
        line-height: 12px
        font-size: 10px
        color: #fff
        background-color: rgb(0, 160, 220)
        border-radius: 12px
        box-sizing: border-box
        opacity: 1
        &.fade-enter-active, &.fade-leave-active
          transition: all .4s
        &.fade-enter, &.fade-leave-to
          opacity: 0
    .info
      padding: 18px
      .title
        margin-bottom: 6px
        font-size: 14px
        color: rgb(7, 17, 27)
        line-height: 14px
      .text
        padding: 0 8px
        font-size: 12px
        color: rgb(77, 85, 93)
        line-height: 24px
    .food-rating
      padding: 18px 0 0
      .title
        margin-left: 18px
        font-size: 14px
        color: rgb(7, 17, 27)
        line-height: 14px
      .rating-wrapper
        padding: 0 18px
        .rating-item
          position: relative
          padding: 16px 0
          border-1px(rgba(7, 17, 27, .1))
          &:last-child
            border-none()
          .user
            position: absolute
            top: 16px
            right: 0
            font-size: 0
            .name
              display: inline-block
              margin-right: 6px
              vertical-align: top
              font-size: 10px
              color: rgb(147, 153, 159)
              line-height: 12px
            .avatar
              vertical-align: top
              line-height: 12px
              background-image: 12px 12px
              background-repeat: no-repeat
              border-radius: 50%
          .time
            font-size: 10px
            color: rgb(147, 153, 159)
            line-height: 12px
          .text
            margin-top: 4px
            font-size: 12px
            color: rgb(7, 17, 27)
            line-height: 16px
            .icon-thumb_up, .icon-thumb_down
              margin-right: 4px
              font-size: 12px
              line-height: 16px
            .icon-thumb_up
              color: rgb(0, 160, 220)
            .icon-thumb_down
              color: rgb(147, 153, 159)

        .no-rating
          padding: 18px 0
          font-size: 12px
          color: rgb(147, 153, 159)
</style>
