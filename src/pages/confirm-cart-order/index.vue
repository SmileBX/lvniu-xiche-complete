<template>
  <div class="backgray">
    <!--地址栏-->
    <div class="flex-container clomn orderhead white" @click="goSelectAddress">
      <div class="orderuser">
        <p>收货人：{{address.name}}</p>
        <p>{{address.phone}}</p>
      </div>
      <div class="flex-container addressContent">
        <img src="/static/images/place.png" class="place">
        <text class="placeinfo">收货地址：{{address.address}}</text>
        <img src="/static/images/back.png" class="right">
      </div>
      <div class="ordertips">(温馨提示：请核对收货地址是否正确)</div>
    </div>
    <div class="slide"></div>
    <!--订单详情--卡券-->
    <div v-if="ServiceCard.dt&&ServiceCard.dt.length>0">
      <div class="gray proorder">
        <!-- <div class="protitle white pad">{{ShopData.ShopName}}</div> -->
        <div class="protitle white pad">服务卡券</div>
        <div
          class="proinfo flex-container pad"
          v-for="(ServiceCard,ServiceCardIndex) in ServiceCard.dt"
          :key="ServiceCardIndex"
        >
          <div>
            <img :src="ServiceCard.ProductImg" class="orderimg">
          </div>
          <div class="inforight">
            <div class="infotitle">{{ServiceCard.ProductName}}</div>
            <div class="infospec">{{ServiceCard.ShopName}}</div>
            <div class="infoprice flex-container">
              <p>￥{{ServiceCard.SalePrice}}</p>
              <p>x{{ServiceCard.Number}}</p>
            </div>
          </div>
        </div>
      </div>
      <div class="gray proorder">
        <div class="flex-container infoslide white pad" @click="onShowCoupon('ServiceCard')">
          <div>优惠券</div>
          <div>
            <span v-show="ServiceCardCoupon.offerPrice*1">- ￥{{ServiceCardCoupon.offerPrice}}</span>
            <img src="/static/images/back.png" class="right">
          </div>
        </div>
        <div class="infoslide inputbor flex-container white pad">
          <div>买家留言</div>
          <input
            type="text"
            v-model="ServiceItemContent"
            placeholder="填写内容已和卖家协商确认"
            class="inputmes"
          >
        </div>
        <div class="infoslide slideprice white pad">
          共计{{ServiceCard.AllNumber}}件商品 合计：
          <span>￥{{ServiceCard.AllPrice}}</span>
        </div>
      </div>
    </div>
    <!--服务项目-->
    <div v-if="ServiceItem.dt&&ServiceItem.dt.length>0">
      <div class="gray proorder">
        <!-- <div class="protitle white pad">{{ShopData.ShopName}}</div> -->
        <div class="protitle white pad">服务项目</div>
        <div
          class="proinfo flex-container pad"
          v-for="(ServiceItem,ServiceCardIndex) in ServiceItem.dt"
          :key="ServiceCardIndex"
        >
          <div>
            <img :src="ServiceItem.ProductImg" class="orderimg">
          </div>
          <div class="inforight">
            <div class="infotitle">{{ServiceItem.ProductName}}</div>
            <div class="infospec">{{ServiceItem.ShopName}}</div>
            <div class="infoprice flex-container">
              <p>￥{{ServiceItem.SalePrice}}</p>
              <p>x{{ServiceItem.Number}}</p>
            </div>
          </div>
        </div>
      </div>

      <div class="gray proorder serviceItem">
        <div class="flex-container infoslide white pad" @click="onGetCarInfo">
          <div>选择车辆</div>
          <div>
            <span v-if="carInfo.Id">{{carInfo.CarBrand}} {{carInfo.CarType}} {{carInfo.CarColor}} {{carInfo.CarMumber}}</span>
            <img src="/static/images/back.png" class="right">
          </div>
        </div>
        <div class="flex-container infoslide white pad" @click="onShowCoupon('ServiceItem')">
          <div>优惠券</div>
          <div>
            <span
              v-show="ServiceItem.offerPrice&&ServiceItemCoupon.id"
            >- ￥{{ServiceItem.offerPrice}}</span>
            <img src="/static/images/back.png" class="right">
          </div>
        </div>
        <div class="flex-container infoslide white pad" @click="onShowCard">
          <div>服务卡券</div>
          <div>
            <span v-show="cardId&&ServiceItem.offerPrice">- ￥{{ServiceItem.offerPrice}}</span>
            <img src="/static/images/back.png" class="right">
          </div>
        </div>
        <div class="infoslide inputbor flex-container white pad">
          <div>买家留言</div>
          <input
            type="text"
            v-model="ServiceCardContent"
            placeholder="填写内容已和卖家协商确认"
            class="inputmes"
          >
        </div>
        <div class="infoslide slideprice white pad">
          共计{{ServiceItem.AllNumber}}件商品 合计：
          <span>￥{{ServiceItem.AllPrice}}</span>
        </div>
      </div>
    </div>
      <div class="fixed">
      <!-- 积分抵扣 -->
       <div class="integral">
        <checkbox-group >
          <label class="checkbox flex-container" @click="scoreStatus = !scoreStatus">
              可用{{score}}积分抵扣{{score/100}}元
            <checkbox :value="name" :checked="scoreStatus" color="#ff6325"/>
          </label>
        </checkbox-group>
       </div>
    <!--底部按钮-->
    <div class="botbtn">
      <div class="price white">
        合计:
        <span>￥{{AllPrice}}</span>
      </div>
      <div class="btnconfir" @click="goPay">提交订单</div>
    </div>
    </div>
    <!--选择优惠券-->
    <Coupon
      :showCoupon.sync="showCoupon"
      :couponId.sync="couponId"
      :couponPrice.sync="couponPrice"
      :couponType.sync="couponType"
      :couponList="couponList"
      @success="selectCoupon"
    ></Coupon>
    <!-- 卡券 -->
    <Card
      :showCoupon.sync="showCardStatus"
      :couponList="cardList"
      :CardTicketId.sync="cardId"
      :CardTicketName.sync="cardName"
    ></Card>
    <!-- 支付 -->
    <Pay
      :showPay.sync="showPay"
      :orderNumber="orderNumber"
      :total="AllPrice"
      :successUrl="`/pages/myorder/main?orderNo=${orderNumber}`"
      :closeUrl="`/pages/myorder/main?orderNo=${orderNumber}`"
      balanceRequestUrl="Order/OrderSoldePayment"
    ></Pay>
  </div>
</template>

<script>
import { post } from "@/utils/index";
import Coupon from "@/components/couponCart.vue";
import Card from "@/components/orderCard.vue";
import Pay from "@/components/pay.vue";
import "../../css/common.css";
import "../../css/global.css";
export default {
  components: {
    Coupon,
    Card,
    Pay
  },
  data() {
    return {
      UserId: "",
      Token: "",
      showPay: false,
      showway: false,
      productList: [],
      freight: 0,
      address: {
        id: "",
        name: "",
        phone: "",
        address: ""
      },
      // 优惠券价格
      couponPrice: 0.0,
      couponType: 0,
      showCoupon: false,
      // 使用的优惠券id，0--不使用
      couponId: 0,
      // 优惠券列表
      couponList: [],
      // 订单编号
      orderNumber: "",
      cartIds: [],
      latitude: 0,
      longitude: 0,

      selectCouponType: "",
      // 服务项目
      ServiceItem: {},
      ServiceItemCoupon: {
        id: "",
        price: ""
      },
      cardId: 0,
      cardName: "不使用服务卡券",
      cardList: [],
      ServiceItemContent: "",
      // 卡券
      showCardStatus: false,
      ServiceCard: {},
      ServiceCardCoupon: {
        id: "",
        price: ""
      },
      ServiceCardContent: "",
      // 选择车辆信息
      carInfo: {
        CarMumber:''
      },

      ShopData: {},
      AllNumber: 0,
      AllPrice: 0,
      score:0, //用户可用积分
      scoreStatus:false,
    };
  },

  computed: {
    total() {
      let totals = 0;
      this.productList.map(arr => {
        arr.list.map(_list => {
          totals += _list.SalePrice * _list.Number;
        });
      });
      totals = totals - this.couponPrice - this.freight;
      return totals.toFixed(2);
      console.log(this.couponPrice, this.freight);
      // 计算购物车价格
      //   const res = await post("Order/BuyCartShopMoney", {
      //     UserId: wx.getStorageSync("userId"),
      //     Token: wx.getStorageSync("token"),
      //     ShopId:0,
      //     CartIds: this.cartIds,
      //     AddressId: this.address.id,
      //     // 优惠券id
      //     MemberCouponId: this.couponId * 1,
      //     // 服务卡券
      //     MemberCardId: 0
      //   });
      //  return  res.data.toFixed(2)
    }
  },
  watch: {
    // 更新id时更新地址
    "$store.state.addressId"() {
      if (this.$store.state.addressId) {
        // this.getAddress();
      }
    },
    // 更新卡券时
    cardId() {
      // if(this.cardId){
      this.comfirmOrder();
      // }
    },
    // 是否使用积分
      scoreStatus(){
        this.comfirmOrder()
      },
    "carInfo.Id"() {
      if (this.carInfo.Id) {
        this.comfirmOrder();
      }
    }
  },
  onShow() {
    this.UserId = wx.getStorageSync("userId");
    this.Token = wx.getStorageSync("token");
    this.setBarTitle();
    // 获取定位
    wx.getLocation({
      type: "gcj02",
      success: res => {
        this.latitude = res.latitude;
        this.longitude = res.longitude;
        this.init();
      }
    });
  },
  // 下拉刷新
  onPullDownRefresh() {
    this.init();
    // 停止下拉刷新
    wx.stopPullDownRefresh();
  },
  methods: {
    setBarTitle() {
      wx.setNavigationBarTitle({
        title: "确认订单"
      });
    },
    init() {
      this.showPay = false;
      this.showCoupon = false;
      this.showCardStatus = false;
      this.ServiceItemCoupon = {};
      this.ServiceCardCoupon = {};
      this.cardId = 0;
      this.cardName = "不使用服务卡券";
      const _carIds = JSON.stringify(this.$store.state.cartIds);
      this.cartIds = _carIds.substring(1, _carIds.length - 1);
      this.carInfo = wx.getStorageSync("CarInfo");
      this.getCarInfo();
      this.getData();
    },
    async getData() {
      // 购物车列表请求
      // const res = await post("Cart/GetConfirmOrderGoods", {
      const res = await post("Cart/GetConfirmOrderXQ", {
        UserId: wx.getStorageSync("userId"),
        Token: wx.getStorageSync("token"),
        CartIdList: this.cartIds,
        Lat: this.latitude,
        Lng: this.longitude
      });
      const data = res.data;
      this.ServiceItem = data.ServiceItem;
      this.ServiceCard = data.ServiceCard;
      this.ShopData = data.ShopData;
      this.AllNumber = data.AllNumber;
      this.AllPrice = data.AllPrice;
      this.score = data.MemberScore;
      console.log("productList", this.productList);

      this.getAddress();
    },
    // 获取默认车辆
    async getCarInfo() {
      if (!this.carInfo.Id) {
        const res = await post("User/GetCarInfo", {
          IsDefault: 1,
          UserId: wx.getStorageSync("userId"),
          Token: wx.getStorageSync("token")
        });
        this.carInfo = res.data[0];
      }
      console.log(this.carInfo);
    },
    // 获取收货地址
    async getAddress() {
      this.address.id = this.$store.state.addressId;
      // 重置store选择地址
      this.$store.commit("setSelectAddress", {
        url: "",
        status: false
      });
      let res = {};
      // 还没选中地址，拿默认地址
      if (!this.address.id) {
        res = await post("Address/defaultaddress_New", {
          UserId: wx.getStorageSync("userId"),
          Token: wx.getStorageSync("token")
        });
      } else {
        // 选择地址获取选择的信息
        res = await post("Address/GetInfo", {
          UserId: wx.getStorageSync("userId"),
          Token: wx.getStorageSync("token"),
          Id: this.address.id
        });
      }
      const _res = res.data;
      this.address = {
        id: _res.id,
        name: _res.name,
        phone: _res.tel,
        address: _res.addressinfo
      };
      this.comfirmOrder();
    },

    // 跳转选择收货地址
    goSelectAddress() {
      this.$store.commit("setSelectAddress", {
        url: "/pages/confirm-cart-order/main",
        status: true
      });
      wx.navigateTo({ url: "/pages/sitemanage/main" });
    },
    // 开始选择优惠券
    onShowCoupon(type) {
      let list = [];
      // 优惠券
      if (type === "ServiceCard") {
        list = this.ServiceCard.CouponData;
        this.couponId = this.ServiceCardCoupon.id
      }
      // 服务卡券
      else if (type === "ServiceItem") {
        list = this.ServiceItem.CouponData;
        this.couponId = this.ServiceItemCoupon.id
      }
      this.couponList = [];
      list.map(item => {
        this.couponList.push({
          name: item.Title,
          id: item.CouponId,
          price: item.Denomination,
          type: item.DiscountType
        });
      });
      this.selectCouponType = type;
      this.showCoupon = true;
    },

    // 选择优惠券成功
    selectCoupon() {
      console.log(this.couponType, "couponType");
      if (this.selectCouponType) {
        const type = this.selectCouponType;
        let price = this.couponPrice;
        if (this.couponType == 1) {
          price = "-" + price;
        } else if (this.couponType == 2) {
          price = price + "折";
        }

        // 优惠券
        if (type === "ServiceCard") {
          this.ServiceCardCoupon = {
            id: this.couponId,
            price,
            type: this.couponType
          };
          // this.ServiceCardCoupon.price=price
        }
        // 服务卡券
        else if (type === "ServiceItem") {
          this.ServiceItemCoupon = {
            id: this.couponId,
            price,
            type: this.couponType
          };
          // this.$set(this.ServiceItemCoupon,'price',price)
        }
        console.log(this.ServiceItemCoupon, "price");
      }
      this.comfirmOrder();
    },
    // 选择卡券
    onShowCard() {
      const list = this.ServiceItem.CardTicketData;
      this.cardList = [];
      list.map(item => {
        this.cardList.push({
          Title: item.Title,
          CardTicketId: item.CardTicketId
        });
      });
      this.showCardStatus = true;
      this.comfirmOrder();
    },
    closeMask() {
      (this.showPay = false), (this.showway = false), (this.showCoupon = false);
    },
    // 确认订单
    async comfirmOrder() {
      if (!this.address.name) {
        return false;
      }
      const params = {
        UserId: this.UserId,
        Token: this.Token,
        CartIds: this.cartIds,
        ContactName: this.address.name,
        Tel: this.address.phone,
        Score:this.scoreStatus?1:0,
        ServiceItem: {
          CarInfoId: this.carInfo.Id,
          CardTicketId: this.cardId || 0,
          CouponId: this.ServiceItemCoupon.id || 0,
          Remark: this.ServiceItemContent
        },
        ServiceCard: {
          CouponId: this.ServiceCardCoupon.id || 0,
          Remark: this.ServiceCardContent
        }
      };
      const res = await post("Order/CheckBuyCartOrder", params);
      this.AllPrice = res.data.allPayMoney;
      this.ServiceCard.AllPrice = res.data.ServiceCard.allPayMoney;
      this.$set(
        this.ServiceCard,
        "offerPrice",
        res.data.ServiceCard.AllyhPrice
      );
      this.ServiceItem.AllPrice = res.data.ServiceItem.allPayMoney;
      this.$set(
        this.ServiceItem,
        "offerPrice",
        res.data.ServiceItem.AllyhPrice
      );
      // this.ServiceItem.offerPrice = res.data.ServiceItem.AllyhPrice
      console.log(res.data.ServiceItem, this.ServiceItem);
    },
    // 提交订单
    async goPay() {
      const params = {
        UserId: this.UserId,
        Token: this.Token,
        CartIds: this.cartIds,
        ContactName: this.address.name,
        Tel: this.address.phone,
        Score:this.scoreStatus?1:0,
        ServiceItem: {
          CarInfoId: this.carInfo.Id,
          CardTicketId: this.cardId || 0,
          CouponId: this.ServiceItemCoupon.id || 0,
          Remark: this.ServiceItemContent
        },
        ServiceCard: {
          CouponId: this.ServiceCardCoupon.id || 0,
          Remark: this.ServiceCardContent
        }
      };
      const res = await post("Order/BuyCartOrder", params);
      wx.showToast({ title: "提交成功！" });

      console.log("typeof", typeof res.data);
      let order = "";
      if (typeof res.data === "string") {
        this.orderNumber = res.data;

        // 查询订单价格
        // order = await post("Order/OrderDetails", {
        //   UserId: wx.getStorageSync("userId"),
        //   Token: wx.getStorageSync("token"),
        //   OrderNo: res.data
        // });
      } else {
        const str = JSON.stringify(res.data);
        const _str = _carIds.substring(1, str.length - 1);
        this.orderNumber = _str;
      }
      // 1.5秒后弹出支付窗口
      setTimeout(() => {
        this.showPay = true;
        console.log("前台计算订单价格", this.total);
        console.log("后台计算订单价格", this.showPay);
      }, 500);
    },
    // 跳转选择车辆
    onGetCarInfo() {
      wx.navigateTo({
        url: "/pages/mycar/main?url=123"
      });
    }
  },

  created() {
    // let app = getApp()
  }
};
</script>

<style lang="scss" scoped>
@import "./style";
@import "../../css/common.css";
.serviceItem {
  padding-bottom: 150rpx;
}
.infoslide img {
  margin-left: 10rpx;
}
.proorder {
  border-bottom: 20rpx #f4f7fc solid;
}
.proinfo {
  width: 100%;
  border-bottom: #fff 10rpx solid;
  justify-content: flex-start;
  .infoprice,
  .inforight {
    width: 100%;
  }
}
</style>
