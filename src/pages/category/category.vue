<template>
  <view class="category">
    <!-- 搜索框 -->
    <view class="search-box">
      <view class="search" @tap="toSearchPage">
        <view class="icon">
          <image src="/static/images/search.png" />
        </view>
        <view class="text">搜索您想购买的商品或者店铺</view>
      </view>
    </view>
    <!-- 内容区 -->
    <view class="main">
      <!-- 左侧内容 -->
      <view class="first-menu">
        <block v-for="(firstCategory, index) in firstCategoryList" :key="index">
          <view class="item" :class="{active:selectedIndex===index}" :data-index="index" @tap="switchCategory">{{ firstCategory.name }}</view>
        </block>
      </view>
      <!-- 右侧内容 -->
      <view class="subclass">
        <view v-if="firstCategoryList.length" class="adv-img">
          <image :src="firstCategoryList[selectedIndex].imgUrl" />
        </view>
        <block v-for="(secondCategory, index) in secondCategoryList" :key="index">
          <view class="sub-con">
            <view class="second-menu">{{ secondCategory.name }}</view>
            <view class="third-menu">
              <block v-for="(thirdCategory, subIndex) in secondCategory.categories" :key="subIndex">
                <view class="item" :data-categoryid="thirdCategory.categoryId" :data-name="thirdCategory.name" @tap="toSearchListPage">
                  <view class="img">
                    <image :src="thirdCategory.imgUrl" />
                  </view>
                  <view class="text">{{ thirdCategory.name }}</view>
                </view>
              </block>
            </view>
          </view>
        </block>
      </view>
    </view>
  </view>
</template>

<script>
const http = require('../../utils/http.js')
const cartCount = require('../../utils/cart-count.js')
const config = require('../../utils/config.js')

export default {
  computed: {
    config() {
      return config
    }
  },
  data() {
    return {
      selectedIndex: 0, // 选中的分类项(默认为第一个分类)
      firstCategoryList: [], // 一级分类内容(一级分类下的categories为二级分类)
      secondCategoryList: [] // 二级分类内容(二级分类下的categories为三级分类)
    }
  },

  /**
   * 生命周期函数--监听页面加载
   */
  onLoad: function(options) {
    this.getPlatformCategoryList()
  },

  /**
   * 生命周期函数--监听页面
   */
  onShow: function() {
    // 获取购物车数字
    cartCount.getCartCount()
  },

  methods: {
    /**
     * 获取平台分类(直接得到1、2、3级的分类)
     */
    getPlatformCategoryList() {
      uni.showLoading()
      const params = {
        url: '/xxw_shop_goods/ua/category/shop_category_list',
        method: 'GET',
        data: {
          shopId: 0,
          parentId: 0
        },
        callBack: res => {
          if (!res) {
            return uni.hideLoading()
          }
          this.firstCategoryList = res
          // 默认选择: 第一个一级分类
          this.secondCategoryList = this.firstCategoryList[0].categories
          uni.hideLoading()
        }
      }
      http.request(params)
    },

    /**
     * 切换分类，改变二级分类内容
     */
    switchCategory(e) {
      const index = parseInt(e.currentTarget.dataset.index)
      this.selectedIndex = index
      this.secondCategoryList = this.firstCategoryList[index].categories
    },

    /**
     * 跳转到搜索页
     */
    toSearchPage() {
      uni.navigateTo({
        url: '/pages/search-page/search-page'
      })
    },

    /**
     * 跳转到商品列表页
     */
    toSearchListPage(e) {
      const categoryId = parseInt(e.currentTarget.dataset.categoryid)
      const keyword = e.currentTarget.dataset.name
      uni.navigateTo({
        url: '/pages/search-list/search-list?categoryId=' + categoryId + '&keyword=' + keyword
      })
    }
  }
}
</script>

<style>
@import "./category.css";
</style>
