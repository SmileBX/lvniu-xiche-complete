<template>
  <div v-if="hasData">
    <div class="remind">
      <text>{{info.StatusName}}</text>
    </div>
    <div class="flex-container orderhead white">
        <img src="/static/images/place.png" class="place">
        <div class="clomn personinfo">
            <div class="orderuser">
                <p>{{info.ContactName}}</p>
                <p>{{info.TelephoneNumber}}</p>
            </div>
            <div>{{info.ServiceAddr}}</div>
        </div>
    </div>
    <div class="slide"></div>
    <div class="flex-container prodetail" v-for="(item,index) in info.orderDetails" :key="index">
        <div class="orderpics">
            <img :src="item.ProductImg" class="orderpics">
        </div>
        <div class="proright">
            <div class="clomn protitle">
              <p>{{item.ProductName}}</p>
            </div>
            <div class="flex-container">
              <p class="ordercolor flex1"><span v-if="item.ProductSkuName">{{item.ProductSkuName}}</span><span v-else>无规格属性</span></p>
              <p class="ordercolor">x{{item.ProductCount}}</p>
            </div>
            <p>￥{{item.UnitPrice}}</p>
        </div>
    </div>
    <div class="orderlist">
        <div class="orderitem padtop">
            <!-- <div class="flex-container">
                <p>运费（快递）</p>
                <p>￥{{info.ExpressPrice}}</p>
            </div> -->
            <div class="flex-container">
                <p>订单价格</p>
                <p>￥{{info.TotalPrice}}</p>
            </div>
            <div class="flex-container">
                <p>优惠</p>
                <p>-¥0.00</p>
            </div>
        </div>
        <div class="flex-container padtop">
              <p>总价</p>
              <p class="pricecolor">￥{{info.TotalPrice}}</p>
        </div>
    </div>
    <div class="slide"></div>
    <div class="orderinfo">
        <p class="infotitle">订单信息</p>
        <p class="pitem">订单编号：{{info.OrderNumber}} <span class="copy" @click="copy(info.OrderNumber)">复制</span></p>
        <p class="pitem">创建时间：{{info.AddTime}}</p>
        <p class="pitem" v-if="info.StatusId !==0&&info.PayTime">支付时间：{{info.PayTime}}</p>
        <!-- <div v-if="shwoSuccess">
            <p class="pitem">到店时间：2018-04-20 17:20:59</p>
            <p class="pitem">成交时间：2018-04-20 18:20:59</p>
        </div> -->
    </div>
    <div class="flex-container tipmenu">
        <p></p>
        <p class="flex-container" @click="callService">
            <img src="/static/images/phonecall.png" class="tippics1">
            <span>拨打电话</span>
        </p>
        <!-- <p class="flex-container">
            <img src="/static/images/kefu.png" class="tippics2">
            <span>在线客服</span>
        </p> -->
    </div>
    <div class="slide"></div>
    <div class="backgray">
        <!-- 未付款 -->
        <div class="orderbottom white" v-if="info.StatusId==0">
            <p @click="showReasonMak">取消订单</p>
            <p @click="showPay=true">付款</p>
        </div>
        <!--已付款、已完成-->
        <div class="orderbottom white" v-if="info.StatusId==13">
            <p @click="goRefund">申请退款</p>
            <p @click="btnDel">删除订单</p>
        </div>
        <!--申请退款-->
        <div class="orderapybottom white " v-if="info.StatusId==16">
            <p @click="closeRefund">撤销退款</p>
        </div>
        <!--已退款 、已经取消订单删除-->
        <div class="orderbottom white" v-if="info.StatusId==14 || info.StatusId==17|| info.StatusId===18">
            <p @click="btnDel">删除订单</p>
        </div>
        <!--买家已付款-->
        <!-- <div class="orderapybottom white " v-if="showPay">
            <p>再次购买</p>
        </div> -->
        <!--交易成功-->
        <!-- <div class="orderapybottom white " v-if="shwoSuccess">
            <p @click="addCommont">去评价</p>
        </div> -->
    </div>
    <!--遮罩层是否删除订单-->
    <!-- <div class="mask-modal" v-if="showMask"></div> -->
    <!--删除遮罩层-->
    <!-- <div v-if="showDelete" class="maskdelete">
        <div class="title">
          <text>您的订单尚未付款成功，确认要取消本订单吗？</text>
        </div>
        <div class="flex-container bottombtn">
            <div>取消</div>
            <div @click="confirmDel">确认</div>
        </div>
    </div> -->
    <!--拨打电话遮罩层-->
    <div v-if="showCall" class="maskcall">
        <div class="title call">
          <text>400-118-5222</text>
        </div>
        <div class="flex-container bottombtn">
            <div>取消</div>
            <div @click="confirmDel">呼叫</div>
        </div>
    </div>
      
    <!-- 取消订单选择原因 -->
    <reasonMask
      title="取消订单原因"
      button="确定"
      :show="reasonShow"
      :data="reasonList"
      @closeReason="closeReason"
      @selectReason="selectReason"
    ></reasonMask>
    <!-- 支付 -->
    <Pay :showPay.sync="showPay" :orderNumber="orderNo" 
      :total="info.TotalPrice" 
      balanceRequestUrl="Order/OrderSoldePayment"
    ></Pay>
  </div>
</template>

<script>
import { post,get } from "../../utils";
import reasonMask from "@/components/reasonMask";
import Pay from '@/components/pay.vue'
import "../../css/common.css";
import "../../css/global.css";
export default {
  components: {
    reasonMask,
    Pay
  },
  onLoad(){
    this.setBarTitle();
  },
  onShow(){
    this.userId = wx.getStorageSync("userId");
    this.token = wx.getStorageSync("token");
    this.orderNo = this.$root.$mp.query.orderNo;
    this.getOrderDetails();
  },
  data () {
    return {
      // orderItemNum:"",//发表评论的订单编号
      hasData:false,
      showCall:false,

      
      reasonShow: false,
      reasonList: [],
      userId:"",
      token:"",
      orderNo:"",
      info:{},
      showPay:false,  //支付弹窗
      totalPrice:""  //需要支付的价格
    }
  },
  methods: {
    setBarTitle() {
      wx.setNavigationBarTitle({
        title: "订单详情"
      });
    },
    cancleOrder(){
      // this.showMask=true
    },
    goToPay(){
      this.showPay=true
    },
    confirmDel(){
      // this.showMask=false
    },
    addCommont(){
      wx.navigateTo({ url: "/pages/addcomment/main?orderNo="+this.orderItemNum+"&url=addcomment"});
    },
    async getOrderDetails(){
      let result = await post("Order/OrderDetails",{
        UserId:this.userId,
        Token:this.token,
        OrderNo:this.orderNo
      })
      console.log(result,"订单详情")
      if(Object.keys(result.data).length>0){
        this.info = result.data;
        // this.orderItemNum=result.data.OrderNumber
        // console.log(this.orderItemNum,"子单号")
        this.hasData = true;
      }
    },
    // 关闭取消订单原因弹窗
    closeReason() {
      this.reasonShow = false;
    },
    selectReason(code, codeTxt) {
      this.cancelOrders(codeTxt);
    },
    // 删除订单
    btnDel() {
      let _this = this;
      wx.showModal({
        content: "您确定要删除该订单么？",
        confirmColor:'#ff6325',
        success(res) {
          if (res.confirm) {
            _this.DeleteOrders();
          } else if (res.cancel) {
          }
        }
      });
    },
    // 删除订单
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
          icon: "none",
          duration: 1500,
          success: function() {
            setTimeout(() => {
              wx.navigateBack()
            }, 1500);
          }
        });
      }
    },
    // 申请退款
    goRefund(){
      wx.navigateTo({
        url:`/pages/applymoney/main?orderNo=${this.orderNo}&showShop=${true}`
      })
    },
    // 撤销退款
    closeRefund(){
     const that = this;
      wx.showModal({
        title:'请确认进行退款撤销！',
        confirmColor:'#ff6325',
        success(res){
          if(res.confirm){
              post('Order/CanelRefund',{
                UserId: that.userId,
                Token: that.token,
                OrderNo: that.orderNo,
              }).then(()=>{
              wx.showToast({
                title:'撤销成功！'
              })
              setTimeout(()=>{
                 that.getOrderDetails();
              },1000)
              })
          }
        }
      })

    },
      //获取取消订单原因
    async showReasonMak() {
      let result = await get("Order/CancelReason");
      if (result.data.length > 0) {
        this.reasonList = result.data;
      }
      console.log(this.reasonList,'aaa')
      this.reasonShow = true;
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
        wx.showToast({
          title: "取消订单成功!",
          icon: "none",
          duration: 1500,
          success: function() {
            setTimeout(() => {
              _this.reasonShow = false;
              
              //做改变订单状态
                _this.$set(_this.info,"StatusId",14);
                _this.$set(_this.info,"StatusName","交易关闭");
            }, 1500);
          }
        });
      }
    },
    // 联系客服，打电话
    callService(){
      wx.makePhoneCall({
        phoneNumber:this.info.ShopMobile
      })
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

  created () {
    // let app = getApp()
  }
}
</script>

<style lang="scss" scoped>
  @import "./style";
  @import "../../css/common.css";
</style>
