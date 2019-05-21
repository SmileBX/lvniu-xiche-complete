<template>
  <div class="backgray">
    <div>
      <!---->
      <div class="flex-container clomn ordershophead white">
        <div class="orderserve">服务商家</div>
        <div class="flex-container ordermain" v-if="orderinfo.ShopData">
          <img :src="orderinfo.ShopData.Logo" class="ordershopimg">
          <div class="flex-container clomn orderplace">
            <p class="placename">{{orderinfo.ShopData.ShopNick}}</p>
            <p>{{orderinfo.ShopData.Address}}</p>
          </div>
        </div>
        <div class="ordertips">(温馨提示：请直接至门店进行洗车服务)</div>
      </div>
      <div class="slide"></div>
      <!--订单详情-->
      <div class="white proinfo">
        <div class="orderserve">服务项目</div>
        <div class="flex-container ordermain">
          <img :src="orderinfo.PicNo" class="ordershopimg">
          <div class="flex-container clomn orderplace">
            <p class="placename detailright">{{orderinfo.Name}}</p>
            <p>￥{{orderinfo.Price}}</p>
          </div>
        </div>
        <!-- 上门订单 -->
        <div class="flex-container infoslide white pad" @click="selectItem">
          <div>增加服务项目</div>
          <div>
            <img src="/static/images/back.png" class="right"/>
          </div>
        </div>
        <div class="flex-container infoslide white pad" @click="choseItem(1)">
          <div>上门服务地点</div>
          <div>
            <img src="/static/images/back.png" class="right"/>
          </div>
        </div>

        <div class="flex-container infoslide white pad" @click="choseItem(1)">
          <div>服务车辆</div>
          <div>
            <span v-if="CarInfo">{{CarInfo}}</span>
            <img src="/static/images/back.png" class="right" v-else>
          </div>
        </div>
        <div class="flex-container infoslide white pad" @click="choseItem(3)">
          <div>优惠券{{CouponType}}</div>
          <div>
            <span v-if="couponPrice">{{couponPrice}}</span>
            <img src="/static/images/back.png" class="right" v-else>
          </div>
          <!-- <div>
                  <img src="/static/images/back.png" class="right">
          </div>-->
        </div>
        <div class="flex-container infoslide white pad" @click="choseItem(5)">
          <div>服务卡券</div>
          <div>
            <span v-if="CardTicketName">{{CardTicketName}}</span>
            <img src="/static/images/back.png" class="right" v-else>
          </div>
        </div>
        
        <!-- 上门订单 -->
        <div class="flex-container infoslide white pad inputbor" @click="choseItem(5)">
          <div>联系人</div>
          <input type="text" placeholder="填写联系人" class="inputmes" v-model="Remarks">
        </div>
        <div class="flex-container infoslide white pad inputbor" @click="choseItem(5)">
          <div>联系电话</div>
          <input type="text" placeholder="填写联系电话" class="inputmes" v-model="Remarks">
        </div>
        <div class="flex-container infoslide white pad" @click="choseItem(5)">
          <div>添加现场照片</div>
          <div>
            <span v-if="CardTicketName">{{CardTicketName}}</span>
            <img src="/static/images/back.png" class="right" v-else>
          </div>
        </div>
        <div class="infoslide inputbor flex-container white pad">
          <div>备注信息</div>
          <input type="text" placeholder="填写备注信息" class="inputmes" v-model="Remarks">
        </div>


        <div class="infoslide inputbor flex-container white pad">
          <div>买家留言</div>
          <input type="text" placeholder="填写内容已和卖家协商确认" class="inputmes" v-model="Remarks">
        </div>

        <div class="infoslide slideprice white pad">
          合计：
          <span>￥{{totalPrice}}</span>
        </div>
      </div>
      <!--底部按钮-->
      <div class="botbtn">
        <div class="price white">
          合计:
          <span>￥{{totalPrice}}</span>
        </div>
        <div class="btnconfir" @click="goPay">提交订单</div>
      </div>
      <!--遮罩层-->
      <!-- 优惠券 -->
      <orderCount
        :showCoupon.sync="showCoupon"
        :couponId.sync="CouponId"
        :couponType.sync="CouponType"
        :couponList="couponList"
        :couponPrice.sync="couponPrice"
      ></orderCount>
      <!-- 卡券 -->
      <orderCard
        :showCoupon.sync="showCardTicket"
        :CardTicketId.sync="CardTicketId"
        :couponList="couponList"
        :CardTicketName.sync="CardTicketName"
      ></orderCard>
    </div>
    <Pay
      :showPay.sync="showPay"
      :orderNumber="orderNumber"
      :total="totalPrice"
      :successUrl="`/pages/appointorderdet/main?orderNo=${orderNumber}`"
      :closeUrl="`/pages/appointorderdet/main?orderNo=${orderNumber}`"
    ></Pay>
    <!-- <div>  
        <div class="lasttime white">剩余支付时间  <span>{{minute}}</span> : <span>{{second}}</span></div>
        <div class="flex-container white commonpad">
            <p class="strong">应付金额</p>
            <p class="shouldpay">￥{{totalPrice}}</p>
        </div>
        <div class="slide"></div>
        <div class="white">
            <div class="itempay commonpad">选择第三方支付，剩余支付<span class="shouldpay">¥{{totalPrice}}</span>元</div>
          <radio-group class="radio-group" @change="radioChange">
            <label class="radio" v-for="item in payitems" :key="item.id">
              <div class="flex-container payitem commonpad">
                  <div class="flex-container">
                      <img src="/static/images/wx.png" class="payimg" v-if="item.id==1">
                      <img src="/static/images/rmbbg.png" class="payimg" v-if="item.id==2">
                      <text style="margin-left:20rpx;"> {{item.value}}</text>
                  </div>
                  <radio :value="item.id" :checked="item.checked"/>
              </div>
            </label>
          </radio-group>
        </div>
        <div class="btncharge" @click="payMoney">立即支付</div>
    </div>-->
  </div>
</template>

<script>
import { get, myget, mypost, post, toLogin } from "../../utils";
import orderCount from "@/components/orderCount.vue";
import Pay from "@/components/pay.vue";
import orderCard from "@/components/orderCard.vue";
import "../../css/common.css";
import "../../css/global.css";
export default {
  onLoad() {
    this.setBarTitle();
    //首次获取默认车辆的信息
  },
  onShow() {
    this.initData();
    this.Token = wx.getStorageSync("token");
    this.UserId = wx.getStorageSync("userId");
    this.showPay = false;
    //获取vuex商品信息
    this.proid = this.$store.state.visitconfirmorder.ProductId;
    console.log(this.proid,'proid')
    this.lat = wx.getStorageSync("latitude");
    this.lng = wx.getStorageSync("longitude");
    // this.Password = wx.getStorageSync("password");
    const carInfo = wx.getStorageSync("CarInfo");
    if (carInfo) {
      this.CarInfoId = carInfo.Id;
      this.CarInfo = carInfo.CarBrand + carInfo.CarType + carInfo.CarColor;

      this.getTotal(); //获取订单总金额
    } else {
      // this.CarInfoId=wx.getStorageSync("carId")
      this.getDefaultCar();
    }
    this.getOrderInfo();
    console.log(this.proid, this.Token, this.UserId, this.CarInfoId);
    //this.changeTime()
    // if(this.CarInfoId){
    //   this.getTotal()//获取订单总金额
    // }
  },
  watch: {
    CardTicketId() {
      // if(this.CardTicketId!=='0'){
       this.getTotal(); //获取订单总金额
      // }
    },
    CouponId(){
      let price = this.couponPrice;
        if (this.CouponType == 1) {
          price = "-" + price;
        } else if (this.CouponType == 2) {
          price = price + "折";
        }
        console.log(this.CouponType,'this.CouponType')
      this.couponPrice = price
       this.getTotal(); //获取订单总金额
    }
  },
  data() {
    return {
      showPay: false,
      couponList: [], //子组件展示信息
      CardTicketData: [], //卡券列表
      CouponData: [], //优惠券列表
      isshow: true,
      proid: "", //产品id
      CarInfoId: "", //车辆id
      CarInfo: "",
      CouponId: "0", //优惠券id
      CouponType: "", //优惠券id
      CardTicketId: "0", //服务卡券id
      couponPrice: "0.00",
      CardTicketPrice: "0.00",
      CardTicketName: "",
      Token: "",
      UserId: "",
      orderNumber: "", //订单编号
      // Password: "",
      Remarks: "",
      orderinfo: {},
      orderParmar: "", ////////////
      minute: 10,
      second: "00",
      timer: "",
      a: 4,
      payitems: [
        { id: 1, value: "微信支付", checked: "true" },
        { id: 2, value: "余额支付" }
      ],
      showCoupon: false, //组件
      showCardTicket: false,
      totalPrice: "" //总价格
    };
  },
  computed: {
    //计算之后的总价格
    // total() {
    //   let totals = 0;
    //   totals =
    //     this.totalPrice -
    //     this.couponPrice -
    //     this.totalPrice * this.CardTicketPrice;
    //   return totals.toFixed(2);
    // }
  },
  components: {
    orderCount,
    orderCard,
    Pay
  },
  methods: {
    setBarTitle() {
      wx.setNavigationBarTitle({
        title: "确认订单"
      });
    },
    initData() {
      this.CardTicketId = "0"; //服务卡券id
      this.CardTicketName = "";
      // this.CardTicketPrice = "0.00";
      this.CouponId= "0"; //优惠券id
      this.couponPrice= "0.00";
    },
    // 查询默认车辆
    async getDefaultCar() {
      // console.log(this.CarInfoId,"默认车辆id")
      let res = await post("/User/GetCarInfo", {
        UserId: this.UserId,
        Token: this.Token,
        // CarId:this.CarInfoId
        IsDefault: 1
      });
      console.log(res, "获取默认车辆");
      const _res = res.data[0];
      this.CarInfoId = _res.Id;
      this.CarInfo = _res.CarBrand + _res.CarType + _res.CarMumber;
      this.getTotal(); //获取订单总金额
    },
    //获取订单信息
    async getOrderInfo() {
      var result = await post("/Order/ServiceProductsFirmOrder", {
        UserId: this.UserId,
        Token: this.Token,
        ProductId: this.proid
      });
      if (result.code == 0) {
        this.orderinfo = result.data[0];
        this.orderinfo.ShopData = result.data[0].ShopData[0];
        this.CouponData = result.data[0].CouponData;
        this.CardTicketData = result.data[0].CardTicketData;
        this.totalPrice = result.data[0].Price
        console.log(this.orderinfo.ShopData, "确认页面详情");
      }
    },
    choseItem(e) {
      if (e == 1) {
        //我的车辆
        wx.navigateTo({ url: "/pages/mycar/main?url=visitconfirmorder" });
      } else if (e == 3) {
        //优惠券
        this.showCoupon = true;
        this.title = "请选择优惠券";
        this.couponList = this.CouponData;
      } else {
        //服务卡券
        this.showCardTicket = true;
        this.title = "请选择服务卡券";
        this.couponList = this.CardTicketData;
      }
      console.log(this.couponList);
    },
    radioChange(e) {
      for (const x in this.payitems) {
        if (this.payitems[x].id == e.target.value) {
          this.payitems[x].checked = true;
        } else {
          this.payitems[x].checked = false;
        }
      }
    },
    //获取订单总金额
    async getTotal() {
      console.log(this.CardTicketId, "获取卡券信息");
      wx.setStorageSync("CarInfo", "");
      var result = await post("/Order/ServiceProductsPlaceOrderVerifyAmount", {
        UserId: this.UserId,
        Token: this.Token,
        ProductId: this.proid,
        CarInfoId: this.CarInfoId,
        CouponId: this.CouponId,
        CardTicketId: this.CardTicketId,
        Remarks: this.Remarks
      }).catch(()=>{
        setTimeout(()=>{
        // 请求失败，情况服务卡券
        this.CardTicketName=''
        this.CardTicketId=''
        },1500)
      });
      console.log(result, "获取订单总金额");
      if (result.code == 0) {
        this.totalPrice = result.data.allPayMoney;
      }
    },
    async goPay() {
      //提交支付
      console.log(
        this.UserId,
        this.proid,
        this.CarInfoId,
        this.CouponId,
        this.CardTicketId,
        "提交信息"
      );
      if (this.CarInfoId) {
        this.isshow = false;
        var result = await post("/Order/ServiceProductsPlaceOrder", {
          UserId: this.UserId,
          Token: this.Token,
          ProductId: this.proid,
          CarInfoId: this.CarInfoId,
          CouponId: this.CouponId,
          CardTicketId: this.CardTicketId,
          Remarks: this.Remarks
        });
        console.log(result, "发起支付请求");
          this.orderNumber = result.data;
          if(result.code*1==200){
            wx.redirectTo({
              url:'/pages/appointorderdet/main?orderNo='+result.data
            })
            return false;
          }
          setTimeout(() => {
            this.showPay = true;
          }, 500);
      } else {
        wx.showToast({
          title: "请选择服务车辆",
          icon: "none",
          duration: 2000
        });
        return false;
      }
    },
    // 上门选择服务项目
    selectItem(){
      //  + this.shopId
      wx.navigateTo({
        url:'/pages/servince/main?shopId=DECACB358531C437'
      })
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
</style>
