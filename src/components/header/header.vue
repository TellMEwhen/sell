<template>
  <div class="header">
    <div class="content-wrapper">
      <div class="avatar">
        <img :src="seller.avatar" alt="seller avatar" width="64" height="64">
      </div>
      <div class="content">
        <div class="title">
          <span class="brand"></span>
          <span class="name">{{seller.name}}</span>
        </div>
        <div class="description">
          {{seller.description}}/{{seller.deliveryTime}}分钟送达
        </div>
        <div v-if="seller.supports" class="supports">
          <span class="icon" :class="classMap[seller.supports[0].type]"></span>
          <span class="text">{{seller.supports[0].description}}</span>
        </div>
      </div>
      <div v-if="seller.supports" class="support-count" @click="showDetail">
        <span class="count">{{seller.supports.length}}个</span>
        <span class="icon-keyboard_arrow_right"></span>
      </div>
    </div>
    <div class="bulletin-wrapper" @click="showDetail">
      <span class="bulletin-title"></span>
      <span class="bulletin-text">{{seller.bulletin}}</span>
      <span class="icon-keyboard_arrow_right"></span>
    </div>
    <div class="background">
      <img :src="seller.avatar" width="100%" height="100%">
    </div>
    <transition name="fade">
      <div class="detail" v-show="detailShow">
        <div class="detail-wrapper clearfix">
          <div class="detail-main">
            <h1 class="name">{{seller.name}}</h1>
            <div class="star-wrapper">
              <star :score="seller.score" :size="48"></star>
            </div>
            <div class="title">
              <div class="line"></div>
              <div class="text">优惠信息</div>
              <div class="line"></div>
            </div>
            <ul class="supports" v-if="seller.supports">
              <li class="support-item" v-for="(item,index) in seller.supports">
                <span class="icon" :class="classMap[seller.supports[index].type]"></span>
                <span class="text">{{seller.supports[index].description}}</span>
              </li>
            </ul>
            <div class="title">
              <div class="line"></div>
              <div class="text">商家公告</div>
              <div class="line"></div>
            </div>
            <div class="bulletin">
              <p class="content">{{seller.bulletin}}</p>
            </div>
          </div>
        </div>
        <div class="detail-close" @click="showDetail">
          <span class="icon-close"></span>
        </div>
      </div>
    </transition>
  </div>
</template>

<script type="text/ecmascript-6">
  import star from '../../components/star/star';

  export default {
    props: {
      seller: {
        type: Object
      }
    },

    data() {
      return {
        detailShow: false
      };
    },

    created() {
      this.classMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee'];
    },

    methods: {
      showDetail() {
        this.detailShow = !this.detailShow;
      }
    },

    components: {
      star
    }
  };
</script>

<style lang="stylus" rel="stylesheet/stylus">
  @import "../../common/stylus/mixin"

  .header
    position: relative
    color: rgb(255, 255, 255)
    overflow: hidden
    background-color: rgba(0, 0, 0, .5)
    .content-wrapper
      position: relative
      padding: 24px 12px 18px 24px
      font-size: 0
      .avatar
        display: inline-block
        width: 64px
        height: 64px
        vertical-align: top
        border-radius: 2px
      .content
        display: inline-block
        margin-left: 16px
        padding: 2px 0
        .title
          margin-bottom: 8px
          font-size: 0
          .brand
            display: inline-block
            width: 30px
            height: 18px
            vertical-align: top
            bg-image('brand')
            background-size: 30px 18px
            background-repeat: no-repeat
          .name
            display: inline-block
            margin-left: 6px
            font-size: 16px
            font-weight: bold
            line-height: 18px
        .description
          margin-top: 8px
          font-size: 12px
        .supports
          margin-top: 10px
          .icon
            display: inline-block
            margin-right: 4px
            width: 12px
            height: 12px
            vertical-align: top
            background-size: 12px 12px
            background-repeat: no-repeat
            // 定义不同的class来达到显示不同的图标的目的
            &.decrease
              bg-image('decrease_1')
            &.discount
              bg-image('discount_1')
            &.guarantee
              bg-image('guarantee_1')
            &.invoice
              bg-image('invoice_1')
            &.special
              bg-image('special_1')
          .text
            display: inline-block
            font-size: 12px
      .support-count
        position: absolute
        right: 12px
        bottom: 15px
        box-sizing: border-box
        padding: 7px 8px
        height: 24px
        text-align: center
        background-color: rgba(0, 0, 0, .1)
        border-radius: 12px
        .count, .icon-keyboard_arrow_right
          display: inline-block
          font-size: 10px
        .count
          margin-right: 2px
        .icon-keyboard_arrow_right
          vertical-align: top
    .bulletin-wrapper
      position: relative
      height: 28px
      box-sizing: border-box
      padding: 0 26px 0 12px
      line-height: 28px
      white-space: nowrap
      overflow: hidden
      text-overflow: ellipsis
      background-color: rgba(7, 17, 27, .2)
      .bulletin-title
        display: inline-block
        margin-top: 8px
        width: 22px
        height: 12px
        bg-image('bulletin')
        background-size: 22px 12px
        background-repeat: no-repeat
      .bulletin-text
        margin: 0 4px
        vertical-align: top
        font-size: 10px
      .icon-keyboard_arrow_right
        position: absolute
        right: 12px
        bottom: 8px
        font-size: 10px
    .background
      position: absolute
      top: 0
      left: 0
      z-index: -1
      width: 100%
      height: 100%
      filter: blur(10px)
    .detail
      position: fixed
      top: 0
      left: 0
      z-index: 100
      width: 100%
      height: 100%
      background-color: rgba(7, 17, 27, .8)
      overflow: auto
      opacity: 1
      &.fade-enter-active, &.fade-leave-active
        transition: all .5s
      &.fade-enter, &.fade-leave
        opacity: 0
        background-color: rgba(7, 17, 27, 0)
      .detail-wrapper
        position: relative
        width: 100%
        min-height: 100%
        .detail-main
          margin-top: 64px
          padding-bottom: 64px
          .name
            font-size: 16px
            font-weight: 700
            text-align: center
          .star-wrapper
            margin-top: 16px
            padding: 2px 0
            text-align: center
          .title
            display: flex
            margin: 28px auto 24px
            width: 80%
            .line
              flex: 1
              position: relative
              top: -11px
              border-bottom: 1px solid rgba(255, 255, 255, .2)
            .text
              margin: 0 12px
              font-size: 24px
              font-weight: 700
          .supports
            margin: 0 auto
            width: 80%
            .support-item
              margin-bottom: 12px
              padding: 0 12px
              font-size: 0
              &:last-child
                margin-bottom: 0
              .icon
                display: inline-block
                margin-right: 6px
                width: 16px
                height: 16px
                vertical-align: top
                background-size: 16px 16px
                background-repeat: no-repeat
                &.decrease
                  bg-image('decrease_2')
                &.discount
                  bg-image('discount_2')
                &.special
                  bg-image('special_2')
                &.invoice
                  bg-image('invoice_2')
                &.guarantee
                  bg-image('guarantee_2')
              .text
                line-height: 16px
                font-size: 12px

          .bulletin
            margin: 0 auto
            width: 80%
            .content
              padding: 0 12px
              font-size: 12px
              line-height: 24px
      .detail-close
        position: relative
        width: 32px
        height: 32px
        margin: -64px auto 0 auto
        clear: both
        font-size: 32px


</style>
