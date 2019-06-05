<template>
  <div>
      <!--顶部图片-->
      <div>
        <img v-show="PicData" :src="PicData.PicUrl" class="bg">
      </div>
      <div class="item">
          <div class="title">{{detailinfo.Name}}</div>
          <div class="complain">{{detailinfo.Synopsis}}</div>
          <div class="flex-container iteminfo">
              <p>已售{{detailinfo.SalesVolume || 0}}件</p>
              <p>今日仅剩<span>{{detailinfo.Stock}}</span>件</p>
          </div>
      </div>
      <div class="hr"></div>
      <!--服务项目-->
      <div class="flex-container item">
          <div >
              <div class="sertitle">服务项目</div>
              <div class="sercomplain">{{detailinfo.Name}}</div>
              <!-- <div class="flex-container iteminfo">
                  <p>
                    <img src="/static/images/gou.png" class="pics">
                    <text>未消费，随时退</text>
                  </p>
                  <p>
                    <img src="/static/images/gou.png" class="pics">
                    <text>过期退</text>
                  </p>
              </div> -->
          </div>
          <div class="price">
            <div>￥{{detailinfo.Price || 0}}</div>
            <div class="vipPrice">VIP ￥{{detailinfo.VipPrice}}</div>
            </div>
      </div>
      <div class="hr"></div>
       <!--优惠-->
      <div class="flex-container coupon">
        <div class="flex-container">
          <div>优惠</div>
          <div>
            <div class="couponmenu">
              <div class="flex-container" v-if="coupon[0]">
                <text class="itemname">优惠券</text>
                <!-- <p class="couponmenuinfo"> -->
                  <!-- <img src="/static/images/deatailbg.png" class="couponbg" style="width:100%;"> -->
                  <!-- <text>满{{coupon[0].meetConditions}}减{{coupon[0].price}}</text> -->
                  <text>{{coupon[0].Name}}</text>
                <!-- </p> -->
              </div>
              <div class="flex-container couptsild">
                <text class="itemname">积分</text>
                <p>购买可得{{detailinfo.Score||0}}积分</p>
              </div>
            </div>
          </div>
        </div>
        <div class="flec-container" @click="getCoupons">
          <text class="getcon">领券</text>
          <img src="/static/images/back.png" class="right">
        </div>
      </div>
      <div class="hr"></div>
      <!--遮罩层-->
      <div class="mask-modal" v-if="showDiscount" @click="showDiscount= false"></div>
      <!--优惠券弹窗-->
      <div class="mask" v-if="showDiscount">
        <div class="masktitle">优惠</div>
        <div class="maskinfo">
          <p class="maskitem">促销</p>
          <div class="maskdetail">
            <p class="maskcoupon">积分</p>
            <text>购买可得{{detailinfo.Score||0}}积分</text>
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
      <!--位置-->
      <div class="flex-container item">
          <div @click="goShopDetail(ShopData.ShopId)">
              <p class="sertitle">{{ShopData.ShopNick}}</p>
              <p class="sercomplain">{{ShopData.Address}}</p>
          </div>
          <div class="flex-container range">
            <p class="location" @click="getMap">
                <img src="/static/images/bg9.png" class="big">
                <img src="/static/images/to.png" class="small">
            </p>
            <p>{{(ShopData.Distance<100)?ShopData.Distance:'100+'}}km</p>
          </div>
      </div>
      <div class="slide"></div>
      <!--评分-->
      <div class="point" v-if="commonlist.length>0">
        <div class="flex-container">
            <p class="left">
              <text>评分</text><text>{{detailinfo.Score}}.0</text>
            </p>
            <p @click="showAllComment">
              <text>查看全部评论</text>
              <img src="/static/images/back.png" class="arrowback">
            </p>
        </div>
        <div class="pointsheet">
            <pointChildpic :commonlist="commonlist" v-if="commonlist.length>0" :showPic="false"></pointChildpic>
        </div>
      </div>
      <div class="slide"></div>
      <!--购买须知-->
      <div class="point" style="margin-bottom:100rpx;">
          <div class="main">购买需知</div>
          <div v-html="detailinfo.ContentDetail">

          </div>
          <!-- <div class="pmenu">
              <img src="/static/images/time.png" class="tippic">
              <div>
                <p class="fsize">适用车型</p>
                <p class="carcor">五座轿车</p>
              </div>
          </div>
          <div class="pmenu">
              <img src="/static/images/time.png" class="tippic">
              <div>
                <p  class="fsize">有效期</p>
                <p class="carcor">付款成功后7天</p>
              </div>
          </div> -->
      </div>
      <!--支付-->
      <div class="btn"> 
          <div class="btnmoney">￥{{detailinfo.Price || 0}}</div>
          <div class="btncharge" @click="toPay(detailinfo.Id)">去结算</div>
      </div>

  </div>
</template>

<script>
import { get, myget, mypost, post, toLogin } from "../../utils";
import pointChildpic from "@/components/pointChildpic"; //公用评论带图片组件
import "../../css/common.css";
import "../../css/global.css";
export default {
  onLoad(){
    this.setBarTitle();
    this.proid=this.$root.$mp.query.proid;
    this.shopLat=this.$root.$mp.query.shopLat;
    this.shopLng=this.$root.$mp.query.shopLng;
    this.lat=wx.getStorageSync('latitude');
    this.lng=wx.getStorageSync('longitude');
    //console.log(this.shopLng,this.shopLat,"商铺经纬度")
  },
  onShow(){
       this.detailinfo={
       };
       this.PicData={}
       this.commonlist={};
      this.ShopData={};
    this.Token=wx.getStorageSync('token');
    this.UserId=wx.getStorageSync('userId');
    
    this.getSerdetail()
  },
  data () {
    return {
      shopLat:"",  //商铺经纬度 
      shopLng:"",
       name:"",
       address:"",
       proid:"",
       lat:"",
       lng:"",
       Token:"",
       UserId:"",
       detailinfo:{
         },
       commonlist:{},
      ShopData:{},
      PicData:[],
      // 领取优惠券
      coupon:[],
      isshow:false,
      showDiscount:false,

    }
  },
 
  components: {
    pointChildpic
  },
  methods: {
    setBarTitle() {
      wx.setNavigationBarTitle({
        title: "服务详情"
      });
    },
    async getSerdetail(){
        var result=await post("/Server/ServiceProductsDetails",{
            UserId:this.UserId,
            Token:this.Token,
            ProductId:this.proid,
            // Lat:this.lat,
            // Lng:this.lng
            Lat:0,
            Lng:0
            // UserId:"91DC2AD9ED2B2C1C",
            // Token:"6CB02D88A51F637467FC0CAD02681D71",
            // ProductId:"341",
            // Lat:22.548456637984177,
            // Lng:114.06455183658751
        })
        if(result.code==0){
            this.detailinfo=result.data[0];
            this.address=result.data[0].ShopData[0].ShopNick
            this.ShopData = result.data[0].ShopData[0]
            this.PicData = result.data[0].PicData[0]
            this.name=result.data[0].ShopData[0].Address
            this.commonlist=result.data[0].CommentData;
            this.$set(result.data[0].ShopData[0],"Distance",parseFloat(result.data[0].ShopData[0].Distance).toFixed(2))
            console.log(this.detailinfo,"服务详情");
            this.address=result.data[0].ShopData[0].Address
            this.name=result.data[0].ShopData[0].ShopNick 
            const tt=result.data[0].ContentDetail.split("><")
            console.log(tt.indexOf("</p>"))
            //console.log("数据长度："+this.commentlist.length);
        }
        // 获取优惠券信息
        this.getCoupon()
    },
    showAllComment(){
        wx.navigateTo({ url: "/pages/shopcommentlist/main?proid="+this.proid });
    },//获取所有评价
    getMap(){
       wx.openLocation({
        latitude:this.shopLat*1,
        longitude:this.shopLng*1,
        address:this.address,
        name:this.name,
        scale: 18
      })
    },
    toPay(e){//跳转到确认订单页面
      // 上门
      if(this.detailinfo.ServiceMode==1){
        const detail = this.detailinfo;
        const params =[{
          Id:detail.Id,
          MarketPrice:detail.MarketPrice,
          Name:detail.Name,
          PicNo:detail.PicData[0].PicUrl,
          Price:detail.Price,
          SalesVolume:detail.ServiceMode,
          Synopsis:detail.Synopsis,
          VipPrice:detail.VipPrice
        }]
         wx.setStorageSync("serItem",params);
        wx.navigateTo({ url: `/pages/xicheConfirmOrder/main?shopId=${this.ShopData.ShopId}` });
      }
      // 到店
      else{
      this.$store.commit("setVisitConfirmOrder",{
          ProductId:e
      })
      wx.navigateTo({ url: "/pages/visitconfirmorder/main" });
      }
    },
    // 获取优惠券列表
    async getCoupon() {
      if (wx.getStorageSync("token") && wx.getStorageSync("userId")) {
        const params = {
          UserId: wx.getStorageSync("userId"),
          Token: wx.getStorageSync("token"),
          ShopId:this.ShopData.ShopId,
          page: 1
        };
        const res = await post("Coupon/CouponCenter", params);
        this.coupon = [];
        for (let i = 0; i < res.data.length; i += 1) {
          const _res = res.data[i];
          this.coupon.push({
            price: _res.Denomination,
            time: _res.StartEndTimeStr,
            Name:_res.Name,
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
    // 跳转到店铺详情
    goShopDetail(id){
      wx.navigateTo({url:'/pages/shopdetail/main?shopid='+id})
    }
  },

  created () {
    // let app = getApp()
  }
}
</script>

<style lang="scss" scoped>
  @import "./style";
  @import "../../css/common.css";
</style>
