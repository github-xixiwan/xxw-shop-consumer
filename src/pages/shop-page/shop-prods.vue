<template>
  <view class="shop-page">
    <view class="shop-prods">
      <!-- 头部 -->
      <view class="header">
        <view class="bg">
          <image src="/static/img/banner3.png" />
        </view>
        <view class="bg-mask" />
        <view class="shop-info">
          <view class="logo">
            <image v-if="shopInfo.shopLogo" :src="shopInfo.shopLogo" />
          </view>
          <view class="text-box">
            <view class="name">{{ shopInfo.shopName }}</view>
            <view class="focus-box">
              <view v-if="shopInfo.type === 1" class="self">自营</view>
            </view>
          </view>
        </view>
        <view class="sortbar">
          <view class="item active">默认</view>
          <view class="list-style" @tap="changeListStyle">
            <image v-if="isLineProds" src="/static/images/list-row-w.png" />
            <image v-else src="/static/images/list-line-w.png" />
          </view>
        </view>
      </view>

      <!-- 商品列表 -->
      <view v-if="prodList.length" class="prods-box" :class="{'add-bg' : !isLineProds}">
        <view :class="[isLineProds ? 'line-prods' : 'prods']">
          <block v-for="(item,index) in prodList" :key="index">
            <view class="item" @tap="detail(item.spuId)">
              <view class="img">
                <image :src="item.mainImgUrl" />
              </view>
              <view class="text-box">
                <view class="name">{{ item.spuName }}</view>
                <view class="price-box">
                  <view class="price">
                    <view class="symbol">￥</view>
                    <view class="big">{{ item.priceFee | price }}</view>
                  </view>
                </view>
              </view>
            </view>
          </block>
        </view>
        <view class="nomore">没有更多了，看看别的吧</view>
      </view>
      <view v-else class="nomore">暂无商品，看看别的吧</view>

      <!-- 店铺tabbar -->
      <view class="shop-tabbar">
        <view class="item" @tap="goShopIndex">
          <view class="icon">
            <!-- <image src="/static/images/shop-index-r.png" /> -->
            <image src="/static/images/shop-index.png" />
          </view>
          <view class="text">首页</view>
        </view>
        <view class="item active" @tap="goShopProds">
          <view class="icon">
            <image src="/static/images/shop-prods-r.png" />
            <!-- <image src="/static/images/shop-prods.png" /> -->
          </view>
          <view class="text">商品</view>
        </view>
        <view class="item" @tap="goShopCategory">
          <view class="icon">
            <!-- <image src="/static/images/shop-category-r.png" /> -->
            <image src="/static/images/shop-category.png" />
          </view>
          <view class="text">分类</view>
        </view>
      </view>
    </view>
  </view>
</template>

<script>
const config = require('../../utils/config.js')
const http = require('../../utils/http.js')
export default {
  computed: {
    config() {
      return config
    }
  },
  filters: {
    price(value) {
      return (value / 100).toFixed(2)
    }
  },
  data() {
    return {
      shopId: 0,
      shopInfo: {},
      shopSecondaryCategoryId: '',
      keyword: '',
      pageNum: 1,
      pageSize: 8,
      total: 1,
      pages: 1,
      prodList: [],
      isLineProds: true // 列表格式，默认横排展示
    }
  },
  onReachBottom() {
    if (this.pageNum < this.totalPage) {
      this.pageNum++
      this.getProd()
    }
  },
  onLoad(options) {
    this.shopId = options.shopId
    this.shopSecondaryCategoryId = options.shopSecondaryCategoryId
    this.keyword = options.keyword
    this.shopInfo = uni.getStorageSync('shopInfo')
    this.getProd()
    // this.getShopInfo()
  },
  methods: {
    // 去商品详情
    detail(spuId) {
      uni.navigateTo({
        url: '/pages/detail/detail?spuId=' + spuId
      })
    },
    // 收藏店铺
    collectShop() {
      this.isCollect = !this.isCollect
    },
    /**
     * 获取商品列表
     */
    getProd() {
      const params = {
        url: '/xxw_shop_search/ua/search/simple_page',
        method: 'GET',
        data: {
          shopId: this.shopId,
          pageNum: this.pageNum,
          pageSize: this.pageSize,
          shopSecondaryCategoryId: this.shopSecondaryCategoryId
        },
        callBack: res => {
          if (this.pageNum !== 1) {
            this.prodList = this.prodList.concat(res.records[0].spus)
          } else {
            this.prodList = res.records[0].spus
          }
          this.totalRow = res.totalRow
          this.totalPage = res.totalPage
        },
        errCallBack: errMsg => {
          console.log(errMsg)
        }
      }
      http.request(params)
    },

    // 切换tabbar
    goShopIndex() {
      uni.navigateTo({
        url: `/pages/shop-page/shop-index?shopId=${this.shopId}`
      })
    },
    goShopProds() {
      uni.navigateTo({
        url: `/pages/shop-page/shop-prods?shopId=${this.shopId}`
      })
    },
    goShopCategory() {
      uni.navigateTo({
        url: `/pages/shop-page/shop-category?shopId=${this.shopId}`
      })
    },

    // 切换列表样式
    changeListStyle() {
      this.isLineProds = !this.isLineProds
    }

  }
}
</script>
<style>
@import "./shop-page.css";
</style>
