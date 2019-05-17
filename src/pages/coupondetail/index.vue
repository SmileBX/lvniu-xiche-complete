<template>
  <div class="coupondetail">
    <div class="top flex-container">
      <div class="tophead">
        <img :src="product.img" class="cardbg">
        <!-- <div>{{清洗卡套餐}}</div> -->
      </div>
      <div class="title">
        <p>{{product.Name}}</p>
        <p>{{product.Synopsis}}</p>
      </div>
      <div class="price">
        <span>￥</span>
        {{product.Price}}
      </div>
    </div>
    <div class="slide"></div>
    <!-- 套餐 -->
    <div v-if="product.BrandId===22">
      <div class="titleinfo">
        <img src="/static/images/fontborder.png" class="itemtitle">
        <!-- <p>{{product.Name}}</p> -->
        <p>卡券套餐</p>
      </div>
      <div class="flex-container iteminfo" v-for="(item,index) in product.MealArr" :key="index">
        <div>{{item.Name}}</div>
        <div class="flex-container inforight">
          <div>{{item.CardTicketsNum}}次</div>
          <div>{{item.MarketPrice}}</div>
        </div>
      </div>
      <!-- <div class="flex-container iteminfo infpborder">
            <div>前挡风玻璃镀膜</div>
            <div class="flex-container inforight">
                <div>1次</div>
                <div>150元</div>
            </div>
      </div>-->
      <div class="infoprice">
        <p>价值¥{{product.MarketPrice}}</p>
        <p>
          购卡仅需¥
          <span>{{product.Price}}</span>
        </p>
      </div>
    </div>
    <!--优惠-->
      <div class="flex-container coupon">
        <div class="flex-container">
          <div>优惠</div>
          <div>
            <div class="couponmenu">
              <div class="flex-container" v-if="coupon[0]">
                <text class="itemname">优惠券</text>
                <!-- <p class="couponmenuinfo"> -->
                  <!-- <img src="/static/images/deatailbg.png" class="couponbg"> -->
                  <!-- <text>满{{coupon[0].meetConditions}}减{{coupon[0].price}}</text> -->
                  <text>{{coupon[0].Name}}</text>
                <!-- </p> -->
              </div>
              <div class="flex-container couptsild">
                <text class="itemname">积分</text>
                <p>购买可得{{shop.score||0}}积分</p>
              </div>
            </div>
          </div>
        </div>
        <div class="flec-container" @click="getCoupons">
          <text class="getcon">领券</text>
          <img src="/static/images/back.png" class="right">
        </div>
      </div>
    <!-- 商户信息 -->
    <div class="ranklist">
      <div class="rankitem flex-container">
        <div class="flex-container shopnum">
          <div class="flex-container rankinfo">
            <img :src="shop.img" class="shopimg">
            <div class="rankcol">
              <p class="shopname">{{shop.name}}</p>
              <p>
                <img src="/static/images/xing.png" v-for="item in shop.score" :key="item" class="xing-point">
                <text>{{shop.score}}.0分</text>
              </p>
            </div>
          </div>

          <div class="ranknums" @click="goShopHome">
            进店逛逛
          </div>
        </div>
      </div>
    </div>

    <div class="linetop">
      <div class="titleinfo">
        <img src="/static/images/border2.png" class="itemtitle">
        <p>购卡须知</p>
      </div>
      <div class="tips" v-html="product.content">
        <!-- <p>1.套餐卡购买成功后，可直接在APP内使用；</p>
            <p>2.使用套餐卡下单时，其他所有促销、优惠券均不可叠加；</p>
            <p>3.套餐卡已经使用后不可退款，请确认后使用；</p>
            <p>4.购卡权益会以服务卡券的形式发放至账户内；</p>
        <p>5.本套餐卡自购买之日起一年内有效，请在有效期内使用；</p>-->
        <!-- {{product.ContentDetail}} -->
      </div>
    </div>
    <div class="slide"></div>
    <div class="fixed">
      <div class="botbtn" @click="confirm">立即购买</div>
    </div>
    
    <!--遮罩层-->
      <div class="mask-modal" v-if="showDiscount" @click="showDiscount= false"></div>
      <!--优惠-->
      <div class="mask" v-if="showDiscount">
        <div class="masktitle">优惠</div>
        <div class="maskinfo">
          <p class="maskitem">促销</p>
          <div class="maskdetail">
            <p class="maskcoupon">积分</p>
            <text>购买可得{{shop.score||0}}积分</text>
          </div>
          <p class="maskitem">领券</p>
          <div class="maskcouponlist" v-for="(coupon,couponIndex) in coupon" :key="couponIndex">
            <img src="/static/images/pinkbg.png" class="pink">
            <div class="maskser flex-container">
              <div>
                <p class="maskprice">
                  {{coupon.DiscountType==1?'¥':''}}
                  <span>{{coupon.price}}</span>
                  {{coupon.DiscountType==2?'折':''}}
                </p>
                <p class="maskask">满{{coupon.meetConditions}}使用</p>
                <p class="maskask">有效期{{coupon.time}}</p>
              </div>
              <div class="maskget" @click="nowGetCoupon(coupon.id)">立即领取</div>
            </div>
          </div>
        </div>
        <div class="btnget" @click="cancleMask">完成</div>
      </div>
  </div>
</template>

<script>
import { post } from "@/utils/index";
import "../../css/common.css";
import "../../css/global.css";
export default {
  onShow() {
    (this.userId = wx.getStorageSync("userId")),
      (this.token = wx.getStorageSync("token")),
      (this.showDiscount=false)
      this.setBarTitle();
    this.id = this.$root.$mp.query.id;
    this.getData();
  },
  data() {
    return {
      id: "",
      userId: "",
      token: "",
      product: {},
      shop:{
        id:'',
        name:'',
        img:'',
        pf:0
      },
      // 优惠券
      coupon:[],
      showDiscount:false,

    };
  },

  components: {},
  methods: {
    setBarTitle() {
      wx.setNavigationBarTitle({
        title: "详情"
      });
    },
    async getData() {
      const res = await post("Server/ServiceMealProductsDetails", {
        // ProductId: 309 * 1,
        ProductId: this.id * 1,
        UserId: this.userId,
        Token: this.token,
        Lat: 0,
        Lng: 0
      });
      // debugger;
      this.product = res.data[0];
      // 把每个图片加上样式
      // console.log(res.data[0].ContentDetail)
      // console.log(res.data[0].ContentDetail.replace(/\<img/gi, '<img style="max-width:100%;height:auto" '))
      this.product.content = res.data[0].ContentDetail.replace(/\<img/gi, '<img style="max-width:100%;height:auto" ');
      let imgs = [];
      this.product.PicData.map(img => {
        imgs.push(img.PicUrl);
      });
      this.product.img = imgs[0];
      const shop = res.data[0].ShopData[0]
      this.shop={
        shopId:shop.ShopId,
        name:shop.ShopNick,
        img:shop.Logo,
        score:res.data[0].Score
      }
      // 获取优惠券信息
      this.getCoupon();
    },
    
    // 获取优惠券列表
    async getCoupon() {
      if (wx.getStorageSync("token") && wx.getStorageSync("userId")) {
        const params = {
          UserId: wx.getStorageSync("userId"),
          Token: wx.getStorageSync("token"),
          ShopId:this.shop.shopId,
          page: 1
        };
        const res = await post("Coupon/CouponCenter", params);
        this.coupon = [];
        for (let i = 0; i < res.data.length; i += 1) {
          const _res = res.data[i];
          this.coupon.push({
            price: _res.Denomination,
            Name:_res.Name,
            time: _res.StartEndTimeStr,
            meetConditions: _res.MeetConditions,
            DiscountType:_res.DiscountType,
            id: _res.Id
          });
        }
        console.log(this.coupon, "coupon");
      }
    },
    // 展示优惠券
    getCoupons() {
      this.isshow = true;
      this.showDiscount = true;
    },
    // 领取优惠券
    async nowGetCoupon(id) {
      const params = {
        UserId: wx.getStorageSync("userId"),
        Token: wx.getStorageSync("token"),
        UseType: 0,
        CouponId: id
      };
      const res = await post("Coupon/ReceiveCoupon", params);
      wx.showToast({
        title: "领取成功!"
      });
      // setTimeout(() => {
      //   this.isshow = false;
      //   this.showDiscount = false;
      // }, 1500);
    },
    // 关闭优惠券弹窗
    cancleMask() {
      this.isshow = false;
      this.showDiscount = false;
    },
    // 购买
    confirm() {
      // 判断库存
      // if(!this.stockCheck()){
      //   return false;
      // }
      // this.showNums = false;
      // this.isshow = false;

      this.$store.commit("setConfirmOrder", {
        addressId: "",
        productId: this.product.Id,
        sku: "",
        // buyNum:this.selectSkuNum,
        buyNum: 1,
        couponId: ""
      });
      wx.navigateTo({ url: `/pages/confirmorder/main?isCard=${true}` });
    },
    // 进店逛逛
    goShopHome(){
      wx.navigateTo({
        url:`/pages/shopdetail/main?shopid=${this.shop.shopId}`
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
