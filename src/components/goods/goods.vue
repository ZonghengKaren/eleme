<template>
  <div>
    <div class="goods">
      <!--左边商品分类栏-->
      <div class="menu-wrapper" ref="menuWrapper">
        <ul>
          <li
            v-for="(item,index) in goods"
            class="menu-item"
            :class="{'current': currentIndex === index}"
            @click="selectMenu(index,$event)">
            <span class="text border-1px">
              <span
                v-show="item.type>0"
                class="icon"
                :class="classMap[item.type]"></span>{{ item.name }}
            </span>
          </li>
        </ul>
      </div>
      <!--右边商品栏-->
      <div class="foods-wrapper" ref="foodsWrapper">
        <ul>
          <li v-for="item in goods" class="food-list food-list-hook">
            <h1 class="title">{{ item.name }}</h1>
            <ul>
              <li
                v-for="food in item.foods"
                class="food-item border-1px"
                @click="selectFood(food,$event)">
                <div class="icon">
                  <img :src="food.icon" width="57px" height="57px">
                </div>
                <div class="content">
                  <h2 class="name">{{ food.name }}</h2>
                  <p class="desc">{{ food.description }}</p>
                  <div class="extra">
                    <span class="count">月售{{ food.sellCount }}份</span>
                    <span>好评率{{ food.rating }}%</span>
                  </div>
                  <!--price组件price-->
                  <price :food="food"></price>
                  <!--添加子组件-->
                  <div class="cartcontrol-wrapper">
                    <cartcontrol :food="food" @event="getEvent"></cartcontrol>
                  </div>
                </div>
              </li>
            </ul>
          </li>
        </ul>
      </div>
      <!--购物车子组件-->
      <shopcart ref="shopcart"
                :select-foods="selectFoods"
                :min-price="seller.minPrice"
                :delivery-price="seller.deliveryPrice"></shopcart>
    </div>
    <!--food组件-->
    <food :food="selectedFood" ref="food"></food>
  </div>
</template>

<script type="text/ecmascript-6">
  // better-scroll插件
  import BScroll from 'better-scroll';
  import shopcart from '../shopcart/shopcart.vue';
  import cartcontrol from '../cartcontrol/cartcontrol.vue';
  import food from '../food/food.vue';
  import price from '../price/price.vue';
  const ERR_OK = 0;
  export default {
    props: {
      seller: {
        type: Object,
        default(){
          return {
            'name': '粥品香坊（回龙观）',
            'description': '蜂鸟专送',
            'deliveryTime': 38,
            'score': 4.2,
            'serviceScore': 4.1,
            'foodScore': 4.3,
            'rankRate': 69.2,
            'minPrice': 20,
            'deliveryPrice': 4,
            'ratingCount': 24,
            'sellCount': 90,
            'bulletin': '粥品香坊其烹饪粥料的秘方源于中国千年古法，在融和现代制作工艺，由世界烹饪大师屈浩先生领衔研发。坚守纯天然、0添加的良心品质深得消费者青睐，发展至今成为粥类的引领品牌。是2008年奥运会和2013年园博会指定餐饮服务商。',
            'supports': [
              {
                'type': 0,
                'description': '在线支付满28减5'
              },
              {
                'type': 1,
                'description': 'VC无限橙果汁全场8折'
              },
              {
                'type': 2,
                'description': '单人精彩套餐'
              },
              {
                'type': 3,
                'description': '该商家支持发票,请下单写好发票抬头'
              },
              {
                'type': 4,
                'description': '已加入“外卖保”计划,食品安全保障'
              }
            ],
            'avatar': 'http://static.galileo.xiaojukeji.com/static/tms/seller_avatar_256px.jpg',
            'pics': [
              'http://fuss10.elemecdn.com/8/71/c5cf5715740998d5040dda6e66abfjpeg.jpeg?imageView2/1/w/180/h/180',
              'http://fuss10.elemecdn.com/b/6c/75bd250e5ba69868f3b1178afbda3jpeg.jpeg?imageView2/1/w/180/h/180',
              'http://fuss10.elemecdn.com/f/96/3d608c5811bc2d902fc9ab9a5baa7jpeg.jpeg?imageView2/1/w/180/h/180',
              'http://fuss10.elemecdn.com/6/ad/779f8620ff49f701cd4c58f6448b6jpeg.jpeg?imageView2/1/w/180/h/180'
            ],
            'infos': [
              '该商家支持发票,请下单写好发票抬头',
              '品类:其他菜系,包子粥店',
              '北京市昌平区回龙观西大街龙观置业大厦底商B座102单元1340',
              '营业时间:10:00-20:30'
            ]
          };
        }
      }
    },
    data() {
      return {
        goods: [],
        listHeight: [], // 存放每个商品列表的高度偏差
        scrollY: 0, //用户滚动的位置
        selectedFood: {} // 点击查看详情的商品
      }
    },
    computed: {
      // 根据滚动的位置判断当前的滚动区间,再取出index去对应左边的商品分类高亮
      currentIndex() {
        for (let i = 0; i < this.listHeight.length; i++) {
          let height1 = this.listHeight[i];
          let height2 = this.listHeight[i + 1];
          // 遍历到最后一个时,listHeight[length+1]返回undefined,所以可以用!height2做判断不是最后一个
          if (!height2 || (this.scrollY >= height1 && this.scrollY < height2)) {
            return i;
          }
        }
        return 0;
      },
      // 把选中的商品价格和数量放入数组中
      selectFoods() {
        // foods会传入购物车组件中，作为购物车的数据来源
        let foods = [];
        this.goods.forEach((good) => {  // good-每个大分类的数据
          good.foods.forEach((food) => {  // food-分类中的每条具体数据
            if (food.count) { // 在cartcontrol组件中设置count
              foods.push(food);
            }
          });
        });
        return foods;
      }
    },
    created() {
      // 活动小图标的类名
      this.classMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee'];
      // 请求goods数据
      this.$http.get('/api/goods').then((response) => {
        response = response.body;
        if (response.errno === ERR_OK) {
          this.goods = response.data;
          // this.$nextTick(() =>{}) 函数
          // 数据更新后,dom真正发生变化是在这个函数后
          // 我们操作dom时,需要在这个函数的回调中去执行
          // 这样就能保证操作的dom一定存在
          this.$nextTick(() => {
            // 获取滚动量
            this._initScroll();
            // 计算每个商品li到父盒子的累加值
            this._calculateHeight();
          });
        }
      });
    },
    methods: {
      // 左边商品分类栏点击事件
      selectMenu(index, $evnt) {
        // 自己开发的event._constructed是为true,pc浏览器没有此事件
        if (!event._constructed) {
          return;
        }
        let foodList = this.$refs.foodsWrapper.getElementsByClassName('food-list-hook');
        let el = foodList[index]; // 当前点击的li
        this.foodsScroll.scrollToElement(el, 300);
      },
      // 点击查看商品详情
      selectFood(food, event) {
        if (!event._constructed) {
          return;
        }
        this.selectedFood = food; // 将点击的food赋值并传入food组件
        this.$refs.food.show(); // 调用food组件的show方法
      },
      // 获取滚动量
      _initScroll() {
        this.menuScroll = new BScroll(this.$refs.menuWrapper, {
          click: true
        });
        this.foodsScroll = new BScroll(this.$refs.foodsWrapper, {
          click: true, // 取消默认阻止事件
          probeType: 3 //监听事件的触发时间，1为即时触发，3为延迟到事件完毕后触发
        });
        // 滚动事件
        this.foodsScroll.on('scroll', (pos) => {
          // 当前滚动的位置
          this.scrollY = Math.abs(Math.round(pos.y));
        })
      },
      _calculateHeight() {
        // 每个商品的盒子
        let foodList = this.$refs.foodsWrapper.getElementsByClassName('food-list-hook');
        let height = 0; // 第一个盒子的高度偏差
        this.listHeight.push(height);
        for (let i = 0; i < foodList.length; i++) {
          let item = foodList[i];
          height += item.clientHeight; // 每个商品盒子到父盒子顶部的距离
          this.listHeight.push(height);
        }
      },
      // 获取子组件cartcontrol的事件源
      getEvent(el){
        // 体验优化,异步执行下落动画
        this.$nextTick(() => {
          // 通过设置ref访问子组件shopcart
          // 调用子组件的drop方法,传入el
          this.$refs.shopcart.drop(el);
        });
      }
    },
    components: {
      shopcart,
      cartcontrol,
      food,
      price
    }
  };
</script>

<style lang="stylus" rel="stylesheet/stylus">
  @import '../../common/stylus/mixin.styl';

  .goods
    display: flex
    position: absolute
    top: 174px
    bottom: 46px
    width: 100%
    overflow: hidden
    .menu-wrapper
      flex: 0 0 80px
      width: 80px
      background: #f3f5f7
      .menu-item
        display: table
        height: 54px
        width: 56px
        padding: 0 12px
        line-height: 14px
        &.current
          position: relative
          z-index: 10
          margin-top: -1px
          background: #fff
          font-weight: 700
          .text
            border-none()
        .icon
          display: inline-block
          vertical-align: top
          width: 12px
          height: 12px
          margin-right: 2px
          background-size: 12px 12px
          background-repeat: no-repeat
          &.decrease
            bg-image('decrease_3')
          &.discount
            bg-image('discount_3')
          &.guarantee
            bg-image('guarantee_3')
          &.invoice
            bg-image('invoice_3')
          &.special
            bg-image('special_3')
        .text
          display: table-cell
          width: 56px
          vertical-align: middle
          border-1px(rgba(7, 17, 27, 0.1))
          font-size: 12px
    .foods-wrapper
      flex: 1
      .title
        padding-left: 14px
        height: 26px
        line-height: 26px
        border-left: 2px solid #d9dde1
        font-size: 12px
        color: rgb(147, 153, 159)
        background: #f3f5f7
      .food-item
        display: flex
        margin: 18px
        padding-bottom: 18px
        border-1px(rgba(7, 17, 27, 0.1))
        &:last-child
          border-none()
          margin-bottom: 0
        .icon
          flex: 0 0 57px
          margin-right: 10px
        .content
          flex: 1
          .name
            margin: 2px 0 8px 0
            height: 14px
            line-height: 14px
            font-size: 14px
            color: rgb(7, 17, 27)
          .desc, .extra
            line-height: 10px
            font-size: 10px
            color: rgb(147, 153, 159)
          .desc
            line-height: 12px
            margin-bottom: 8px
          .extra
            .count
              margin-right: 12px
          .cartcontrol-wrapper
            position: absolute
            right: 0
            bottom: 12px

</style>
