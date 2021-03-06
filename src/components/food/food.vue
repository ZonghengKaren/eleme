<template>
  <transition name="move">
    <div class="food" v-show="showFlag" ref="food">
      <div class="food-content">
        <div class="image-header">
          <img :src="food.image" alt="">
          <div class="back" @click="hide">
            <i class="icon-arrow_lift"></i>
          </div>
        </div>
        <div class="content">
          <div class="title">{{food.name}}</div>
          <div class="detail">
            <span class="sell-count">月售{{food.sellCount}}份</span><span class="rating">好评率{{food.rating}}</span>
          </div>
          <!--price组件price-->
          <price :food="food"></price>
          <div class="cartcontrol-wrapper">
            <cartcontrol :food="food"></cartcontrol>
          </div>
          <transition name="fade">
            <div
              class="buy"
              v-show="!food.count || food.count === 0"
              @click.stop.prevent="addFirst">
              加入购物车
            </div>
          </transition>
        </div>
        <!--split组件-->
        <split v-show="food.info"></split>
        <div class="info" v-show="food.info">
          <h1 class="title">商品信息</h1>
          <p class="text">
            {{food.info}}
          </p>
        </div>
        <!--split组件-->
        <split v-show="food.info"></split>
        <!--评论模块-->
        <div class="rating">
          <h1 class="title">商品评价</h1>
          <ratingselect
            :select-type="selectType"
            :only-content="onlyContent"
            :desc="desc"
            :ratings="food.ratings"></ratingselect>
        </div>
      </div>
    </div>
  </transition>
</template>

<script type="text/ecmascript-6">
  import price from "../price/price.vue";
  import split from '../split/split.vue';
  import cartcontrol from "../cartcontrol/cartcontrol.vue";
  import ratingselect from '../ratingselect/ratingselect.vue';
  import BScroll from 'better-scroll';
  import Vue from 'vue';

  const ALL = 2;
  export default {
    props: {
      food: {
        type: Object
      }
    },
    data() {
      return {
        showFlag: false, // 是否显示详情
        selectType: ALL,  // 显示评论的类别
        onlyContent: true, // 只显示文字评论
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
        // 添加溢出可滚动效果
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
      hide() {  // 返回键
        this.showFlag = false;
      },
      addFirst(event) { // 事件函数不传参数默认是event
        if (!event._constructed) {
          return;
        }
        // 把点击+号的事件源传入父组件
        this.$emit('event', event.target);
        // 第一次food对象没有count属性
        Vue.set(this.food, 'count', 1)
      }
    },
    components: {
      price,
      cartcontrol,
      split,
      ratingselect
    }
  };
</script>

<style lang="stylus" rel="stylesheet/stylus">
  .food
    position: fixed
    left: 0
    top: 0
    bottom: 48px
    z-index: 30
    width: 100%
    background: #fff
    &.move-enter-active, &.move-leave-active
      transition: all 0.2s linear
    &.move-enter, &.move-leave-active
      transform: translate3d(100%, 0, 0)
    .image-header
      position: relative
      width: 100%
      height: 0
      padding-top: 100% // 构建出高度和宽度相等的正方型
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
      .title
        line-height: 14px
        margin-bottom: 8px
        font-size: 14px
        font-weight: 700
        color: rgb(7, 17, 27)
      .detail
        margin-bottom: 18px
        line-height: 10px
        height: 10px
        font-size: 0
        .sell-count, .rating
          font-size: 10px
          color: rgb(147, 152, 159)
        .sell-count
          margin-right: 12px
      .cartcontrol-wrapper
        position: absolute
        right: 12px
        bottom: 12px
      .buy
        position: absolute
        right: 18px
        bottom: 18px
        z-index: 10
        height: 24px
        line-height: 24px
        padding: 0 12px
        box-sizing: border-box
        border-radius: 12px
        font-size: 10px
        color: #fff
        background: rgb(0, 160, 220)
        &.fade-enter-active, &.fade-leave-active
          transition: all 0.2s
        &.fade-enter, &.fade-leave-active
          opacity: 0
    .info
      padding: 18px
      .title
        line-height: 14px
        margin-bottom: 6px
        font-size: 14px
        color: rgb(7, 17, 27)
      .text
        line-height: 24px
        padding: 0 8px
        font-size: 12px
        color: rgb(77, 85, 93)

    .rating
      padding-top: 18px
</style>
