<template>
  <div>
    <div>
        <div class="remind">
          <text>{{info.StatusName}}</text>
        </div>
        <!--切换显示-->
        <!--上门订单-->
        <!-- <div v-if="serType==1"> -->
        <div>
            <div class="flex-container orderhead white">
                <img src="/static/images/place.png" class="place">
                <div class="flex-container clomn personinfo">
                    <div class="orderuser">
                        <p>{{info.ContactName}}</p>
                        <p>{{info.TelephoneNumber}}</p>
                    </div>
                    <div class="mysize">{{info.ServiceAddr}}</div>
                </div>
            </div>
            <div class="flex-container apointtime">
                <img src="/static/images/time11.png" class="timelogo">
                <text>{{info.AppointmentStartTime}} - {{info.AppointmentEndTime}}</text>
            </div>
        </div>
        <!--到店订单-->
        <!-- <div class="flex-container shoporder white" v-if="serType==2">
          <img src="/static/images/place.png" class="place">
          <div class="shopservince">
              <div class="shopservinitem">到店服务</div>
              <div class="flex-container">
                  <img src="/static/images/shop5.png" class="shopbg">
                  <div class="flex-container clomn placeinfo">
                      <p class="placename">{{info.ShopName}}</p>
                      <p>{{info.Address}}</p>
                  </div>
              </div>
          </div>
        </div> -->
        
        <div class="flex-container prodetail" v-if="reasonList.length<=0">
            <img src="/static/images/car22.png" class="mycardet">
            <div class="flex-container clomn carright">
                <p>
                   <span>{{info.CarBrand}}</span>
                   <span v-if="info.CarType">- {{info.CarType}}</span>
                   <span v-if="info.CarColor">- {{info.CarColor}}</span>
                </p>
                <p class="carnums">{{info.CarMumber}}</p>
            </div>
        </div>
    </div>
    <div class="slide"></div>
    <div class="serprotitle">
        <text>服务项目</text>
    </div>
    <div class="flex-container prodetail" v-for="(item,index) in info.orderDetails" :key="index">
        <div class="orderpics">
            <img :src="item.ProductImg" class="orderpics">
        </div>
        <div class="flex-container proright">
            <div class="protitle">
                <p>{{item.ProductName}}</p>
            </div>
            <div class="myright">
                <p>￥{{item.UnitPrice}}</p>
                <p class="ordercolor">x{{item.ProductCount}}</p>
            </div>
        </div>
    </div>
    <div class="orderlist">
        <div class="orderitem padtop">
            <!-- <div class="flex-container">
                <p>运费（快递）</p>
                <p>￥{{info.ExpressPrice}}</p>
            </div> -->
            <div class="flex-container">
                <p>订单总价</p>
                <p>￥{{StartPrice}}</p>
            </div>
            <div class="flex-container">
                <p>优惠</p>
                <p>¥ -{{offerPrice}}</p>
            </div>
        </div>
        <div class="flex-container padtop">
              <p>需付款</p>
              <p class="pricecolor">￥{{info.TotalPrice}}</p>
        </div>
    </div>
    <div class="slide"></div>
    <div class="orderinfo">
        <p class="infotitle">订单信息</p>
        <p class="pitem">订单编号：{{info.OrderNumber}}<span class="copy" @click="copy(info.OrderNumber)">复制</span></p>
        <p class="pitem" v-if="info.WarrantyNumber">质保号：{{info.WarrantyNumber}}<span class="copy" @click="copy(info.WarrantyNumber)">复制</span></p>
        <p class="pitem">创建时间：{{info.AddTime}}</p>
        <div>
           <p class="pitem" v-if="info.StatusId=='13' || info.StatusId=='3' || info.StatusId=='1'">支付时间：{{info.PayTime}}</p>
            <p class="pitem" v-if="info.StatusId=='13'">上门时间：{{info.FaHuoTime}}</p>
            <p class="pitem" v-if="info.StatusId=='13'">到店时间：{{info.FaHuoTime}}</p>
            <p class="pitem" v-if="info.StatusId=='13' || info.StatusId=='3' ">成交时间：{{info.CompleteTime}}</p>
        </div>
    </div>
    <p class="flex-container" style="padding:20rpx 50rpx 20rpx 0;justify-content:flex-end" @click="phoneCall">
          <img src="/static/images/phonecall.png" class="tippics1">
          <span>拨打电话</span>
    </p>
    <!-- <div class="flex-container tipmenu" style="border:1px solid red;text-align:right">
        <p class="flex-container tipmenu" style="border:1px solid blue;width:25%;">
            <img src="/static/images/phonecall.png" class="tippics1">
            <span>拨打电话</span>
        </p>
        <p class="flex-container">
            <img src="/static/images/kefu.png" class="tippics2">
            <span>在线客服</span>
        </p>
    </div> -->
    <div class="slide"></div>
    <div class="backgray">
      
        <div class="orderbottom white" v-if="info.StatusId==0">
            <p class="leftbtn" @click="showReasonMak">取消订单</p>
            <p class="rightbtn" @click="toPay">付款</p>
        </div>
        <!-- <div class="orderbottom white" v-if="info.StatusId==16">
            <p class="rightbtn" @click="closeRefund">撤销退款</p>
        </div> -->
      <!--申请退款-->
        <!-- <div class="orderbottom white" v-if="info.StatusId==1">
            <p class="rightbtn" @click="goRefund">申请退款</p>
        </div> -->
        <div class="orderbottom white" v-if="info.StatusId==2">
            <!-- <p class="leftbtn" @click="goRefund">申请退款</p> -->
            <p class="rightbtn" @click="confirmService">已服务</p>
        </div>
        <div class="orderbottom white" v-if="info.StatusId==3">
            <p class="leftbtn" @click="btnDel">删除订单</p>
            <p class="rightbtn" @click="gotoAddComent">去评价</p>
        </div>
        <div class="orderapybottom white "
        v-if="info.StatusId===14 || info.StatusId===17 || info.StatusId===18||info.StatusId===13||info.StatusId===4">
          <p class="leftbtn" @click="btnDel">删除订单</p>
        </div>
        <!--上门订单 已取消的订单 重新预约-->
        <!-- <div class="orderbottom white" v-if="info.StatusId == '14'">
            <p class="leftbtn" @click="btnDel">删除订单</p>
            <p class="rightbtn" @click="appointAgain">重新预约</p>
        </div> -->
        <!--上门、到店交易成功-->
        <!-- <div class="orderapybottom white " v-if="info.StatusId == '13' || info.StatusId == '3'" >
            <p class="rightbtn" @click="addCommont">去评价</p>
        </div> -->
        <!--到店申请退款-->
        <!-- <div class="orderapybottom white " >
            <p class="rightbtn" @click="addCommont">申请退款</p>
        </div> -->
    </div>
    <!--遮罩层是否删除订单-->
    <!-- <div class="mask-modal" v-if="showMask"></div> -->
    <!--删除遮罩层-->
    <!-- <div v-if="showDelete" class="maskdelete">
        <div class="title">
          <text>您的订单尚未付款成功，确 认要取消本订单吗？</text>
        </div>
        <div class="flex-container bottombtn">
            <div>取消</div>
            <div @click="confirmDel">确认</div>
        </div>
    </div> -->
      
    <!-- 取消订单选择原因 -->
    <reasonMask
      title="是否取消订单"
      button="确定"
      v-if="reasonList.length>0"
      :show="reasonShow"
      :data="reasonList"
      @closeReason="reasonShow= false"
      @selectReason="selectReason"
    ></reasonMask>
    <!-- 支付 -->
    <Pay
      :showPay.sync="showPay"
      v-if="showPay"
      :orderNumber="orderNo"
      :total="info.TotalPrice"
      :balanceRequestUrl="balanceRequestUrl"
    ></Pay>
  </div>
</template>

<script>
import { post, get } from "../../utils";
import reasonMask from "@/components/reasonMask.vue";
import Pay from "@/components/pay.vue";
import "../../css/common.css";
import "../../css/global.css";
export default {
  onLoad(){
    this.setBarTitle();
  },
  onShow(){
    this.reasonList = [];
    //this.orderNo = "";
    this.userId = wx.getStorageSync("userId");
    this.token = wx.getStorageSync("token");
    this.orderNo = this.$root.$mp.query.orderNo;
    this.serType = this.$root.$mp.query.serType;
    //console.log(this.serType,"预约订单详情页")
    this.getOrderDetails();
  },
  watch:{
    reasonShow(){
      if(this.reasonShow==false){
          this.reasonList=[]
      }
    }
  },
  data () {
    return {
      userId:"",
      token:"",
      orderNo:"",
      serType:"",//显示上门订单还是到店
      info:{},
      orderItemNum:"",//发表评论的订单编号
      title: "申请退款原因",
      button: "确定",
      reasonList: [],
      cancleIndex:"",//订单下标
      reasonShow: false,
      showWait:false,//申请退款
      showHas:false,
      showDelOrder:false,
      showDelete:false,
      showMask:false,
      appraiseType: 0, //0:商品评价；1：上门服务评价；2：到店评价
      orderBigType: 1, //1:商城订单；2：预约订单
      
      
      balanceRequestUrl: "Order/PaymentOrder", //余额支付接口
      showPay: false, //支付弹窗
      offerPrice:0, //优惠的价格
      StartPrice:0, //合计价格
    }
  },
 
  components: {
    reasonMask,Pay
  },
  methods: {
    setBarTitle() {
      wx.setNavigationBarTitle({
        title: "订单详情"
      });
    },
    async getOrderDetails(){
      console.log(this.orderNo)
      let result = await post("Order/OrderDetails",{
        UserId:this.userId,
        Token:this.token,
        OrderNo:this.orderNo
      })
      console.log(result,"预约订单详情")
      if(Object.keys(result.data).length>0){
        this.info = result.data;
        // this.info.AppointmentEndTime = this.info.AppointmentEndTime
        this.orderItemNum=result.data.OrderNumber
        let totalPrice = result.data.TotalPrice
        let StartPrice = 0;
        result.data.orderDetails.map(item => {
              StartPrice+=item.UnitPrice*item.ProductCount
              // totalPrice-=item.UnitPrice*item.ProductCount
        }) 
        this.offerPrice=(StartPrice-result.data.TotalPrice).toFixed(2)  //优惠价格
        this.StartPrice = (StartPrice*1).toFixed(2)
        this.phoneNumber=result.data.ShopMobile
        //console.log(this.orderItemNum,"子单号")
        this.hasData = true;
      }
    },
    // applyMoney(e){
    //     console.log(e.target)
    //      //wx.navigateTo({ url: "/pages/applymoney/main" });
    // },
    //待接单的订单申请退款
    applyMoney(){
      console.log(this.serType)
      if(this.serType==1){
          wx.navigateTo({ url: "/pages/applymoney/main?orderNo="+this.orderNo });
      }
        // wx.navigateTo({url:"/pages/addcomment/main?appraiseType="})
        // this.getCancelReason();
    },
      //申请退款原因
     async getCancelReason() {
      let result = await get("/Order/CancelReason");
      if (result.data.length > 0) {
        this.reasonList = result.data;
        console.log(this.reasonList,"退款的原因枚举")
      }
    },
    closeReason() {
      this.reasonShow = false;
    },
    selectReason(code, codeTxt) {
      this.cancelOrders(codeTxt);
    },
    async cancelOrders(reasonMark) {
      console.log(1111111111111111,"退款的结果")
      let result = await post("Order/CancelOrders", {
        UserId: this.userId,
        Token: this.token,
        OrderNo: this.orderNo,
        ReMarks: reasonMark
      });
      console.log(result,"退款的结果")
      if (result.code === 0) {
        let _this = this;
        wx.showToast({
          title: "退款成功!",
          icon: "none",
          duration: 1500,
          success: function() {
            setTimeout(() => { //去订单列表更改状态
              _this.reasonShow = false;},1500);
          }
        });
      }
    },
    //已成交的订单评价
    addCommont(){
         wx.navigateTo({ url: "/pages/addcomment/main?orderNo="+this.orderItemNum+"&url=addcomment"});
    },

    //已取消的订单删除订单
    btnDel(){
      let _this = this;
      wx.showModal({
        content: "您确定要删除该订单么？",
        confirmColor:'#ff6325',
        success(res) {
          console.log(res,"删除订单")
          if (res.confirm) {
           _this.DeleteOrders();
          } else if (res.cancel) {
          }
        }
      });
    },
     async DeleteOrders() {
      let result = await post("Order/DeleteOrders", {
        UserId: this.userId,
        Token: this.token,
        OrderNo: this.orderNo
      });
      if (result.code === 0) {
        let _this = this;
        wx.showToast({
          title: "删除订单成功!",
          duration: 1500,
          success: function() {
            setTimeout(() => {
              // _this.getOrderDetails()
              wx.navigateBack()
              }, 1500); //订单列表自动删除订单了
          }
        });
      }
    },
    //重新预约
    appointAgain(){
       wx.navigateTo({url:"/pages/index/main"})
    },
    //联系客服
    phoneCall(){
      wx.makePhoneCall({
        phoneNumber: this.phoneNumber // 仅为示例，并非真实的电话号码
      })
    },
    // 取消订单1
    showReasonMak() {
      this.reasonShow = true;
      this.getCancelReason();
    },
    // 取消订单2
    //获取取消订单原因
    async getCancelReason() {
      //获取取消订单原因
      let result = await get("Order/CancelReason");
      if (result.data.length > 0) {
        this.reasonList = result.data;
      }
    },
    selectReason(code, codeTxt) {
      this.cancelOrders(codeTxt);
    },
    async cancelOrders(reasonMark) {
      let result = await post("Order/CancelOrders", {
        UserId: this.userId,
        Token: this.token,
        OrderNo: this.orderNo,
        ReMarks: reasonMark
      });
      if (result.code === 0) {
        let _this = this;
        this.reasonShow = false;
        wx.showToast({
          title: "取消订单成功!",
          duration: 1500,
          success: function() {
            setTimeout(() => {
              _this.getOrderDetails()
            }, 1500);
          }
        });
      }
    },
    // 申请退款
    goRefund() {
      wx.navigateTo({
        url: `/pages/applymoney/main?orderNo=${this.orderNo}&showShop=${true}`
      });
    },
    // 撤销退款
    closeRefund() {
      const that = this;
      wx.showModal({
        title: "请确认进行退款撤销！",
        confirmColor:'#ff6325',
        success(res) {
          if (res.confirm) {
            post("Order/CanelRefund", {
              UserId: that.userId,
              Token: that.token,
              OrderNo: that.orderNo
            }).then(() => {
              wx.showToast({
                title: "撤销成功！"
              });
              setTimeout(() => {
                that.getOrderDetails();
              }, 1000);
            });
          }
        }
      });
    },
    // 已服务
    confirmService(){
      const that = this;
      wx.showModal({
        title: "请确认已完成服务！",
        confirmColor:'#ff6325',
        success(res) {
          if (res.confirm) {
            post("Order/ConfirmService", {
              UserId: that.userId,
              Token: that.token,
              OrderNo: that.orderNo
            }).then(() => {
              wx.showToast({
                title: "已服务！"
              });
              setTimeout(() => {
                that.getOrderDetails();
              }, 1000);
            });
          }
        }
      });
    },
    // 评价
    gotoAddComent() {
       wx.navigateTo({
            url:"/pages/addcomment/main?appraiseType=" +
             ' this.serviceMode' +
              "&orderNo=" +
              this.orderNo
          });

    },
    // 支付
    toPay(){
      this.showPay = true;
    },
    // 复制订单号
    copy(info){
      wx.setClipboardData({
        data:info,
        success(){
          wx.showToast({
            title:'复制成功！'
          })
        }
      })
    }

  },

  // 下拉刷新
  onPullDownRefresh() {
    this.getOrderDetails();
    // 停止下拉刷新
    wx.stopPullDownRefresh();
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
