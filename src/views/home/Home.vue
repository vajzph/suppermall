<template>
  <div id="home">
    <nav-bar class="nav-home"><div slot="center">购物街</div></nav-bar>
    <tab-control :titles="['流行','新款','经典']" @tabclick="tabclick" ref="tabControl1" class="tab-control" v-show="isTabFixed"/>
    
    <scroll class="content" ref="scroll" :probe-type="3" @scroll="contentScroll" :pull-up-load='true' @pullingUp="loadMore">
      <home-swiper :banners="banners" @swiperImageLoad="swiperImageLoad"/>
      <home-recommend-view :recommends="recommends"/>
      <home-feature/>
      <tab-control :titles="['流行','新款','经典']" @tabclick="tabclick" ref="tabControl2"/>
      <goods-list :goods="showGoods"/>
    </scroll>

    <back-top @click.native="backClick" v-show="isShowBackTop"/>
  </div>
</template>

<script type="text/ecmascript-6">
import HomeSwiper from './childComps/HomeSwiper'
import HomeRecommendView from './childComps/HomeRecommendView'
import HomeFeature from './childComps/HomeFeature'

import NavBar from 'components/common/navbar/NavBar'
import TabControl from 'components/content/tabControl/TabControl'
import GoodsList from 'components/content/goods/GoodsList'
import Scroll from 'components/common/scroll/Scroll'
import BackTop from 'components/content/backTop/BackTop'

import {getHomeMultidata, getHomeGoods} from "network/home"
import {debounce} from "common/utils"
import {itemListenerMixin, backTopMixin} from "common/mixin"

export default {
  name: "Home",
  components: {
    HomeSwiper,
    HomeRecommendView,
    HomeFeature,
    NavBar,
    TabControl,
    GoodsList,
    Scroll,
    BackTop
  },  
  mixins: [itemListenerMixin, backTopMixin],
  data() {
    return {
      banners: [],
      recommends: [],
      goods: {
        'pop': {page: 0, list: []},
        'new': {page: 0, list: []},
        'sell': {page: 0, list: []}
      },
      currentType: 'pop',
      tabOffsetTop: 0,
      isTabFixed: false,
      saveY: 0
    }
  },
  computed: {
    showGoods() {
      return this.goods[this.currentType].list
    }
  },
  created() {
    //1.请求多个数据
    this.getHomeMultidata()

    this.getHomeGoods('new')
    this.getHomeGoods('pop')
    this.getHomeGoods('sell')
    
  },
  mounted() {  
  },
  activated() {
    this.$refs.scroll.scrollTo(0, this.saveY, 0 )
    this.$refs.scroll.refresh()
  },
  deactivated() {
    //保存Y值
    this.saveY = this.$refs.scroll.getScrollY()
    //取消全局事件监听
    this.$bus.$off('itemImageLoad', this.itemImgListener)
  },
  methods: {
    //事件监听
    tabclick(index) {
      switch (index) {
        case 0:
          this.currentType = 'pop'
          break
        case 1:
          this.currentType = 'new'
          break
        case 3:
          this.currentType = 'sell'
          break
      }
      this.$refs.tabControl1.currentIndex = index
      this.$refs.tabControl2.currentIndex = index
    },
    contentScroll(position) {
      //判断backTap是否显示
      this.listenShowBackTop(position)
      //决定tabControl是否吸顶
      this.isTabFixed = (-position.y) > this.tabOffsetTop
    },
    loadMore() {
      this.getHomeGoods(this.currentType)
    },
    swiperImageLoad() {
      //获取tabControl的offsetTop,,所有的组件都有$el属性：用于获取组件元素
      this.tabOffsetTop = this.$refs.tabControl2.$el.offsetTop
    },

    //数据请求
    getHomeMultidata(){
      getHomeMultidata().then(res =>{
        this.banners = res.data.banner.list;
        this.recommends = res.data.recommend.list;
      })
    },
    getHomeGoods(type){
      const page = this.goods[type].page + 1
      getHomeGoods(type, page).then(res => {
        this.goods[type].list.push(...res.data.list)
        this.goods[type].page += 1

        this.$refs.scroll.finishPullUp()
      })
    },
  }
}
</script>

<style scoped>
 #home {
    /* padding-top: 44px; */
    height: 100vh;

    position: relative;
  }

  .nav-home {
    background-color: var(--color-tint);
    color: white;

    /* 在使用浏览器原生滚动时，为了让导航不跟随一起滚动: */
    /* position: fixed;
    left: 0;
    right: 0;
    top: 0;
    z-index: 9; */
  }

  .content {
    /* height: 300px; */
    overflow: hidden;

    position: absolute;
    top: 44px;
    bottom: 49px;
    left: 0;
    right: 0;
  }

  .tab-control {
    position: relative;
    z-index: 9;
  }

  /* .content {
    height: calc(100% - 93px);
    overflow: hidden;
    margin-top: 44px;
  } */
</style>
