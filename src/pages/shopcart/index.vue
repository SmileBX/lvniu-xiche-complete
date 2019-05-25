<template>
  <div class="backgray">
    <!--全选-->
    <div class="btn flex-container white">
      <div class="btn-close flex-container">
        <input
          type="checkbox"
          class="checkbox-cart"
          @click="selectProduct()"
          :checked="isSelectAll"
        >
        <text>全选商品</text>
      </div>
      <div class="btn-confirm" @click="editPro" v-if="isShow">编辑</div>
      <div class="btn-confirm" @click="editPro" v-else>完成</div>
    </div>
    <div class="slider"></div>
    <!--右列表-->
    <div class="list">
      <div class="flex-container shopitem white" v-for="(item,pindex) in carData" :key="item.id">
        <div class="left">
          <input
            type="checkbox"
            class="checkbox-cart"
            :checked="item.isSelect"
            @click="oneClick(pindex)"
          >
        </div>
        <div class="shopItem">
          <orderChild :showButton="!isShow" :data="item" @editCar="editCar" :gotoDetail="true"></orderChild>
        </div>
        </div>
    </div>
    <!--底部按钮-->
    <div>
      <div class="botbtn" v-if="isShow">
        <div class="price white">
          合计:
          <span>￥{{total}}</span>
        </div>
        <div class="btnconfir" @click="submit">结算</div>
      </div>
      <div class="delete botbtn" v-else>
        <div class="price"></div>
        <div class="btnconfir" @click="deleteCar">删除</div>
      </div>
    </div>
    <!--参数遮罩层-->
    <specChild :showSpecs="showSpecs"></specChild>
  </div>
</template>

<script>
import { post } from "@/utils/index";
import orderChild from "@/components/orderChild";
import specChild from "@/components/specChild";
import "../../css/common.css";
import "../../css/global.css";
export default {
  data() {
    return {
      userId: "",
      token: "",
      carData: [],

      // showMask: false,
      isShow: true,
      showDelete: false,
      showSpecs: false,
      menulist: [
        { name: "洗车" },
        { name: "内饰" },
        { name: "除甲醛" },
        { name: "镀金" }
      ],
      servicelist: [
        {
          id: 1,
          title: "外观简单清洗",
          sale: "198",
          price: "30.00",
          isSelect: true
        },
        {
          id: 2,
          title: "精细清洗漆面上光打蜡",
          sale: "98",
          price: "198.00",
          isSelect: false
        },
        {
          id: 3,
          title: "普通清洗漆面去污",
          sale: "88",
          price: "60.00",
          isSelect: false
        },
        {
          id: 4,
          title: "蒸汽深度清洗",
          sale: "38",
          price: "80.00",
          isSelect: false
        }
      ],
      active: "洗车",
      isSelectAll: false,
      latitude:0,
      longitude:0,
    };
  },

  components: {
    orderChild,
    specChild
  },
  computed: {
    total() {
      let total = 0;
      for (var i = 0; i < this.carData.length; i++) {
        const _carData = this.carData[i];
        if (_carData.isSelect) {
          total += _carData.price * _carData.num;
        }
      }
      return total.toFixed(2);
    }
  },
  onShow() {
    // 初始化全选
    this.userId = wx.getStorageSync("userId");
    this.token = wx.getStorageSync("token");
    this.isSelectAll = false;
    this.isShow = true;
    this.setBarTitle();
    
      // 获取定位
      wx.getLocation({
        type: "wgs84",
        success: res => {
          this.latitude = res.latitude;
          this.longitude = res.longitude;
          this.getCarData();
        }
          });
  },
  methods: {
    setBarTitle() {
      wx.setNavigationBarTitle({
        title: "购物车"
      });
    },

    // 获取购物车信息
    async getCarData() {
      const params = {
        UserId: this.userId,
        Token: this.token,
            Lat: this.latitude,
            Lng: this.longitude
      };
      const res = await post("Cart/CartList", params);
      this.carData = [];
      for (let i = 0; i < res.data.length; i += 1) {
        const datas = res.data[i];
        this.carData.push({
          id: datas.Id,
          img: datas.ProductImg,
          title: datas.ProductName,
          sku: datas.SpecText.replace(/_/g, " "),
          skuSubmit: datas.SpecText,
          shopName: datas.ShopName,
          shopId: datas.ShopId,
          productId:datas.ProductId,
          price: datas.SalePrice,
          maxBuyNum: datas.MaxBuyNum,
          num: datas.Number,
          stock: datas.Stock,
          isSelect: false,
          type:datas.BrandId //商品分类0--全部分类，21--商品，22--套餐，23--卡券
        });
      }
    },
    selectProduct: function() {
      //遍历list，全部取反
      this.isSelectAll = !this.isSelectAll;
      for (var i = 0, len = this.carData.length; i < len; i++) {
        this.carData[i].isSelect = this.isSelectAll;
        console.log(this.carData[i].isSelect);
      }
    },
    oneClick(i) {
      this.carData[i].isSelect = !this.carData[i].isSelect;
      for (var i = 0; i < this.carData.length; i++) {
        if (!this.carData[i].isSelect) {
          this.isSelectAll = false;
          return;
        }
      }
      this.isSelectAll = true;
    },
    // 添加数量
    addNum(id) {
      this.editCar(id, "add");
    },
    // 减少数量
    lessNum(id) {
      this.editCar(id, "less");
    },
    // 编辑购物车
    async editCar(id, num) {
      console.log(id, num);
      num = num * 1;
      let index = "";
      for (let i = 0; i < this.carData.length; i += 1) {
        const _carData = this.carData[i];
        if (_carData.id === id) {
          index = i;
        }
      }
      const data = [
        {
          CartId: id,
          Total: num,
          SpecText: this.carData[index].skuSubmit
        }
      ];
      const params = {
        UserId: this.userId,
        Token: this.token,
        data
      };
      // 数量大于0，编辑购物车
      // 数量等于0，删除购物车
      if (num < 1) {
        const that = this;
        wx.showModal({
        content:'您确定要删除所选商品吗？',
        confirmColor:'#ff6325',
        success(res){
          if(res.confirm){
            console.log('id', num);
            const data=[{
                CartId: id,
                Total: 1,
                SpecText: ''
              }];
            that.details(data)
          }else if(res.cancel){

          }
        }
        })
        return false;
        // wx.showToast({
        //   title: "受不了了，宝贝不能再减少了哦！",
        //   icon: "none"
        // });
        // this.carData[index].num = 1;
      }
      if (num > this.carData[index].stock) {
        wx.showToast({
          title: "超出库存数量了哦！",
          icon: "none"
        });
        this.carData[index].num = this.carData[index].stock;
        return false;
      }
      await post("Cart/EditCart", params);
      this.carData[index].num = num;
      return true;
    },
    // 删除购物车
    deleteCar() {
       const that = this;
        wx.showModal({
        content:'您确定要删除所选商品吗？',
        confirmColor:'#ff6325',
        success(res){
          if(res.confirm){
              let data = [];
              for (let i = 0; i < that.carData.length; i += 1) {
                const _carData = that.carData[i];
                if (_carData.isSelect) {
                  data.push({
                    CartId: _carData.id,
                    Total: _carData.num,
                    SpecText: _carData.skuSubmit
                  });
                }
              }
              that.details(data)
          }else if(res.cancel){

          }
        }
        })
    },
    async details(data){
      this.showMask = false;
      this.showDelete = false;
      const params = {
        UserId: this.userId,
        Token: this.token,
        data
      };
      await post("Cart/DelCart", params);
      wx.showToast({
        title: "删除成功！",
        icon: "success",
        duration: 2000
      });
      this.getCarData();
    },
    // 更改编辑状态
    editPro() {
      this.isShow = !this.isShow;
    },
    submit() {
      const cartIds = [];
      let status = false;
      this.carData.map(data => {
        if (data.isSelect) {
          // 检查是否超出购买最大值数量
          // if(!this.checked(data)){
          //   status = true;
          //    return false
          // } 
          cartIds.push(data.id);
        }
      });
      if(status||cartIds.length<1){
        return false;
      }
      console.log("a", cartIds);
      this.$store.commit("update", { cartIds: cartIds });
      wx.navigateTo({
        url: "/pages/confirm-cart-order/main"
      });
    },
    // 检查库存和最大购买数量
    checked(data){
      if(data.num>data.stock||data.num>data.maxBuyNum){
        wx.showToast({
          title:'超出购买数量！',
          icon:'none'
        })
        return false
      }
      return true
    },
    
  },

  // 下拉刷新
  onPullDownRefresh() {
    this.getCarData();
    // 停止下拉刷新
    wx.stopPullDownRefresh();
  },
  created() {
    // let app = getApp()
  }
};
</script>

<style lang="scss" scoped>
@import "./style";
@import "../../css/common.css";
.list{
  padding-bottom:90rpx;
}
</style>
