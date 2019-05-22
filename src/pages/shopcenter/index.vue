<template>
  <div class="flex-container bigcontainer">
    <!--左菜单-->
    <div>
      <scroll-view scroll-y class="ser-menu">
        <div
          v-for="(item,index) in menulist"
          :key="index"
          :class="{'active':activeClassifyId==item.id}"
          @click="activeClassify(item.id)"
          class="nemeitem"
        >
          <text class="title">{{item.name}}</text>
        </div>
      </scroll-view>
    </div>
    <!--右列表-->
    <scroll-view scroll-y class="list" :scroll-into-view="'item'+activeClassifyId" scroll-with-animation>
      
      <div class="listBox" :id="'item'+product.classifyId" v-for="(product,productIndex) in productlist" :key="productIndex">
        <div class="list_title">{{product.classifyName}}</div>
        <div class="null" v-if="product.list.length<1"></div>
        <div class="flex-container intitem" v-for="(item,index) in product.list"  :key="item.id">
          <div class="flex-container cartdetal">
            <div>
              <img
                :src="item.img"
                class="carpic"
                v-if="ispop"
                @click="goDetail(item.brandId,item.id)"
              >
              <!-- <img src="/static/images/comblo.png" class="carpic" v-else-if="comb"  @click="goDetail(2)">
              <img src="/static/images/cardshop.png" class="carpic" v-else  @click="goDetail(2)">-->
            </div>
            <div class="flex-container citeminfo">
              <p class="itemtitle" @click="goDetail(item.brandId,item.id)">{{item.title}}</p>
              <!-- <p class="progress" @click="goDetail(item.brandId,item.id)" 
              v-if="item.brandId!=21">
                <text v-for="(tab,tabIndex) in item.tab" :key="tabIndex">{{tab}}</text>
              </p> -->
              <!-- <p class="sales" v-if="item.brandId!=21&&item.brandId!=24" @click="goDetail(item.brandId,item.id)">销量{{item.sale}}</p> -->
              <!-- <p class="sales" v-if="item.brandId==24" @click="goDetail(item.brandId,item.id)">销量{{item.sale}}</p> -->
                <!-- 店铺名称 -->
              <!-- <p class="sales" v-if="item.brandId==21" @click="goDetail(item.brandId,item.id)">{{item.shopName}}</p> -->
              <p class="sales" @click="goDetail(item.brandId,item.id)">{{item.shopName}}</p>
                <!-- 距离 -->
              <!-- <p class="sales" v-if="item.brandId==21" @click="goDetail(item.brandId,item.id)">距离：{{item.Distance}}km</p> -->
              <p class="sales" v-if="item.brandId!=24" @click="goDetail(item.brandId,item.id)">距离：{{item.Distance}}km</p>
              <div class="flex-container around">
                <p class="price">￥{{item.price}}</p>
                <!-- <div v-show="item.isAttr"> -->
                <div v-if="item.ServiceMode!=1">
                  <img src="/static/images/s1.png" @click="lessNumber(index)" class="tippic">
                  <text class="nums">{{item.num}}</text>
                  <img src="/static/images/addcart.png" @click="addNumber(index)" class="tippic">
                </div>
                <div class="pay" v-else @click="goDetail(item.brandId,item.id)">立即购买</div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <p
        class="ovedMsg"
        v-if="isOved"
        style="text-align:center;padding:20rpx;font-size:26rpx;color:#666;"
      >没有数据了哦！</p>
    </scroll-view>

    <!--底部按钮-->
    <div class="btn">
      <div class="btninfo">
        <div class="btn-shop">
          <div class="cartshopbg" @click="goUrl('/pages/shopcart/main')">
            <img src="/static/images/shopcart.png" class="cartshop">
            <div class="numright">{{carNum>99?99+'+':carNum}}</div>
          </div>
          <div class="shopprice">￥{{carPrice}}</div>
        </div>
        <div class="btn-confirm" @click="goUrl('/pages/shopcart/main')">确定</div>
      </div>
    </div>
  </div>
</template>

<script>
// BrandId 21:服务项目 22:服务套餐23:服务卡券 24:镀晶版本
import { post } from "@/utils/index";
import "../../css/common.css";
import "../../css/global.css";
export default {
  data() {
    return {
      // servicelist:'',
      ispop: true,
      comb: false,
      // 分类列表
      menulist: [],
      //普通商品
      productlist: [],
      //卡券
      cardlist: [],
      //套餐
      combolist: [],
      activeId: Number,
      isSelectAll: "true",
      carData: [],
      carNum: 0,
      carPrice: 0,
      userId: "",
      token: "",
      page: 1,
      pageSize: 30,
      isOved: false,
      notData: false,
      latitude:0,
      longitude:0,
      activeClassifyId:'a', //活动的分类id
    };
  },
  watch: {
    // productlist:{
    //     handler: function () {
    //       const list =[]
    //       for(let i=0;i<this.productlist.length;i+=1){
    //         const data = this.productlist[i]
    //         if(data.num){
    //           list.push(data)
    //         }
    //       }
    //       this.carData = list;
    //       console.log(this.carData)
    //     },
    //     deep: true
    // }
  },
  onLoad() {
    this.setBarTitle();
    this.initData();
  },
  onShow() {
      this.userId = wx.getStorageSync("userId")
        this.token = wx.getStorageSync("token")
  },
  methods: {
    setBarTitle() {
      wx.setNavigationBarTitle({
        title: "商城"
      });
    },
    initData() {
      this.page = 1;
      this.cardlist = [];
      this.combolist = [];
      this.carData = [];
      this.isOved = false;
      this.notData = false;
        // 分类列表
      // 获取定位
      wx.getLocation({
        type: "wgs84",
        success: res => {
          this.latitude = res.latitude;
          this.longitude = res.longitude;
          this.getClassify();
        }
          });
    },
    // 获取分类列表
    getClassify() {
      const that = this;
      post("Server/GetShopCardSort", { BrandList: 0 }).then(res => {
        this.menulist = [];
        for (let i = 0; i < res.data.length; i += 1) {
          if(i===0&&res.data[0]){
            this.activeClassifyId = res.data[0].Id
          }
          const datas = res.data[i];
          that.menulist.push({
            id: datas.Id,
            name: datas.TypeName,
            BrandList:datas.BrandList
          });
        }
        this.getClassifyLisit()
        // 获取第一个分类产品
        // that.getClassifyLisit(that.menulist[0].id,that.menulist[0].BrandList);
      });
    },
    // 分类产品列表 type分类类型 // 镀晶产品--24
    async getClassifyLisit(id,type) {
      const that = this;
      this.productlist =[]
          console.log(this.activeClassifyId,'activeClassifyId')
      this.menulist.map(async (item)=>{
        
          const params = {
            page: this.page,
            pageSize: this.pageSize,
            TypeId: item.id,
            Lat: this.latitude,
            Lng: this.longitude
          };
         let res = await post("Goods/GoodsList", params)
         let list =[]
            for (let i = 0; i < res.data.length; i += 1) {
              const datas = res.data[i];
              list.push({
                brandId: datas.BrandId, //商品分类0--全部分类，21--商品，22--套餐，23--卡券
                id: datas.Id,
                title: datas.Name,
                price: datas.Price,
                img: datas.ProductImg,
                sale: datas.SalesVolume,
                shopName:datas.ShopName,
                Distance:datas.Distance.toFixed(2),
                ServiceMode:datas.ServiceMode, //是否上门产品
                num: 0,
                stock: datas.Stock,
                tab: datas.KeywordName ? JSON.parse(datas.KeywordName) : [],
                // isAttr:datas.SpecificationValue&&datas.BrandId===21
                isAttr: datas.SpecificationValue
              });
            }
            that.productlist.push({
                classifyId:item.id,
                classifyName:item.name,
                list
            })
      })
      this.getCarData();
            
    },
    // 活动分类
    activeClassify(id){
            this.activeClassifyId = id
    },
    // 删除购物车
    async lessNumber(index) {
      // if (this.productlist[index].num > 0) {
      //   this.productlist[index].num -= 1;
      // }
      const product = this.productlist[index];
      let carId = "";
      let num = 0;
      let status = true;
      for (let i = 0; i < this.carData.length; i += 1) {
        if (this.carData[i].ProductId == product.id) {
          carId = this.carData[i].Id;
          num = this.carData[i].Number - 1;
          status = true;
          break;
        } else {
          status = false;
        }
      }
      // 如果数量等于0，跳出
      if (!status) {
        return false;
      }
      const data = [
        {
          CartId: carId,
          Total: num,
          SpecText: ""
        }
      ];
      const params = {
        UserId: this.userId,
        Token: this.token,
        data: data
      };
      // 数量大于0，编辑购物车
      // 数量等于0，删除购物车
      if (num !== 0) {
        await post("Cart/EditCart", params);
      } else if (num === 0) {
        const res = await post("Cart/DelCart", params);
        if (res.code * 1 === 0) {
          this.productlist[index].num = 0;
        }
      }
      this.getCarData();
    },
    // 添加购物车
    async addNumber(index) {
      // this.productlist[index].num += 1;
      const product = this.productlist[index];
      const params = {
        UserId: this.userId,
        Token: this.token,
        ProId: product.id,
        Count: 1,
        SpecText: ""
      };
      // 大于购物车库存，设置购物车数量为库存
      if (product.stock <= product.num) {
        let carId = "";
        // let num = "";
        for (let i = 0; i < this.carData.length; i += 1) {
          if (this.carData[i].ProductId === product.id) {
            carId = this.carData[i].Id;
            // num = this.carData[i].Number - 1;
          }
        }
        params.Count = product.stock;
        wx.showToast({
          title: "大于库存了哦！",
          icon: "none"
        });
        setTimeout(() => {
          post("Cart/EditCart", {
            UserId: this.userId,
            Token: this.token,
            data: [
              {
                CartId: carId,
                Total: product.stock,
                SpecText: ""
              }
            ]
          }).then(() => {
            this.getCarData();
          });
        }, 1000);
      } else {
        await post("Cart/AddCart", params);
        this.getCarData();
      }
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
      this.carNum = res.data.length;
      let carPrice = 0;
      for (let i = 0; i < res.data.length; i += 1) {
        const datas = res.data[i];
        // this.carNum += datas.Number
        carPrice += datas.SalePrice * datas.Number;
        this.carData.push(datas);
        const productlist = this.productlist;
        for (let j = 0; j < productlist.length; j += 1) {
          const _productlist = productlist[j];
          if (datas.ProductId === _productlist.id) {
            this.productlist[j].num = datas.Number;
          }
        }
      }
      this.carPrice = carPrice.toFixed(2);
    },
    change(e) {
      console.log(e);
      this.active = e;
      if (e == "服务卡券") {
        this.servicelist = this.cardlist;
        this.ispop = false;
        this.comb = false;
      } else if (e == "营销套餐") {
        this.servicelist = this.combolist;
        this.comb = true;
        this.ispop = false;
      } else {
        this.servicelist = this.productlist;
        this.ispop = true;
      }
    },
    // 跳转
    goDetail(type, id) {
      //type商品分类0--全部分类，21--服务卡券商品，22--套餐，23--卡券,24镀晶产品
      console.log(type, "type-id", id);
      // wx.navigateTo({ url: "/pages/detail/main?id=" + id });
      // return false;
            this.$store.commit('update',{
              latitude: this.latitude,
              longitude: this.longitude
            });
      var a = type * 1;
      if (a === 21) {
        wx.navigateTo({ url: "/pages/serdetail/main?proid=" + id });
      }
      if (a === 22 || a === 23) {
        wx.navigateTo({ url: "/pages/coupondetail/main?id=" + id });
      }
      // if (a === 24) {
      //   wx.navigateTo({ url: "/pages/djdetail/main?id=" + id });
      // }
    },
    toPAy() {
      wx.navigateTo({ url: "/pages/confirmorder/main" });
    },
    goUrl(url) {
      wx.navigateTo({ url });
    }
  },
  // 下拉刷新
  onPullDownRefresh() {
    this.initData();
    wx.stopPullDownRefresh();
  },
  // 上拉加载
  onReachBottom() {
    if (!this.isOved && !this.notData) {
      this.page += 1;
      this.getClassifyLisit();
    }
  }
};
</script>

<style lang="scss" scoped>
@import "./style";
@import "../../css/common.css";
.pay{
  width:140rpx;
  line-height:50rpx;
  background:#ff6325;
  border-radius:30rpx;
  color:#fff;
  padding:0 10rpx;
  text-align:center;
}
</style>
