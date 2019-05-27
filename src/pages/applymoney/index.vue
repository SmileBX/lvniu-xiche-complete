<template>
  <div>
    <div>
      <!--商城订单申请退款-->
      <div>
        <div class="flex-container pad" v-for="(item,index) in info.orderDetails" :key="index">
          <div>
            <img :src="item.ProductImg" class="applyimg">
          </div>
          <div class="flex-container clomn applypro">
            <p class="protitle">{{item.ProductName}}</p>
            <p class="prospec">{{info.ShopName}}</p>
            <p class="proprice">¥{{item.UnitPrice}}</p>
          </div>
          
        </div>
      </div>
      
          <div class="flex-container payPrice">
            付款总额：
            <p class="proprice">¥{{info.TotalPrice}}</p>
          </div>
      <!--预约订单申请退款-->
      <!-- <div class="flex-container pad" v-else>
        <div>
          <img src="/static/images/shop5.png" class="applyimg">
        </div>
        <div class="flex-container clomn applypro">
          <p class="protitle">普教洗车</p>
          <p class="proprice">¥ 30</p>
        </div>
      </div> -->
    </div>
    <div class="promopt">
      <p>· 本品支持订单签收后的7天内退换货，商城将在2天内审核申请</p>
      <p>· 申请成功后奖励的积分将相应扣除，货款原支付方式退回</p>
      <p>· 如未与买家协商一致，请勿使用到付或平邮</p>
    </div>
    <div class="slide"></div>
    <!--退款选项-->
    <div>
      <div class="applyitem arrowposit" @click="choseReason">
        <p>退款原因:</p>
        <!-- <input type="text" placeholder="商品少件/破损"> -->
        <div>{{content}}</div>
        <img src="/static/images/back.png" class="right arrowpic">
      </div>
      <div class="flex-container applyitem">
        <p>退款备注:</p>
        <input type="text" v-model="text" placeholder="选填">
      </div>
      <div class="applyitem lastap flex-container money w70">
        <p>退款金额:</p>
        <div>￥{{info.TotalPrice}}</div>
        <!-- <input type="number" v-model="money" :placeholder="info.TotalPrice"> -->
      </div>
    </div>
    <div class="tipspromot">可修改退款金额，最多￥{{info.TotalPrice}}，不含配送费用</div>
    <!-- <div class="addpics">
      <p>上传凭证</p>
      <div>
        <img :src="item" v-for="(item,index) in imgs" :key="index" class="pics">
        <img src="/static/images/camera.png" @click="upImg" class="pics">
      </div>
    </div> -->
    <!-- <upImg :imgLangth="4" @upImgs="upImgs" title="上传凭证" addImgUrl="/static/images/bg20.png"></upImg> -->
    <div class="slide"></div>
    <div class="applycon" @click="submit">确认提交</div>
    <!--弹框遮罩-->
    <div class="mask-modal" v-if="isshow" @click="closeMsk"></div>
    <div class="reasonmask" v-if="isshow">
      <div class="reasontitle commonpaditem">
        <img src="/static/images/close.png" class="close" @click="closeMsk">
        <text>选择退款原因</text>
      </div>
      <div>
        <radio-group @change="selectReason">
          <label
            class="commonpaditem flex-container reasonitem"
            v-for="item in reasonlist"
            :key="item.id"
          >
            <p>{{item.name}}</p>
            <input type="radio" :value="item.id" :checked="item.status" class="checkbox-cart">
          </label>
        </radio-group>
      </div>

      <div class="reasonclose" @click="closeMsk">关闭</div>
    </div>
  </div>
</template>

<script>
import { post} from "../../utils";
// import upImg from "@/components/upImg.vue"
import "../../css/common.css";
import "../../css/global.css";
export default {
  components: {
    // upImg
  },
  data () {
    return {
      userId: "",
      token: "",
      isshow:false,
      showShop:false,//选择是商城订单还是预约订单:true--商城订单，false--预约订单
      reasonlist:[
        {id:1,name:"不想买了",status:true},
        {id:2,name:"更优惠的活动",status:false},
        {id:3,name:"信息填写错误重新拍",status:false},
        {id:4,name:"卖家缺货",status:false},
        {id:5,name:"同城交易",status:false},
        {id:6,name:"其他原因(请备注说明)",status:false},
      ],
      orderNo: '',
      info:{},
      money:'',
      content:'不想买了',
      contentId:1,
      text:'',
      imgs:[]
    }
  },
 
  onShow(){
    this.setBarTitle();
    this.userId = wx.getStorageSync("userId");
    this.token = wx.getStorageSync("token");
    if(this.$root.$mp.query.showShop){
      this.showShop = this.$root.$mp.query.showShop;
    } if(this.$root.$mp.query.orderNo){
      this.orderNo = this.$root.$mp.query.orderNo;
    }
    this.init();
    this.getOrderDetails();
  },
  methods: {
    init(){
      
      this.info={};
      this.money='';
      this.content='不想买了';
      this.contentId=1;
      this.text='';
      // this.imgs=[];
    },
    setBarTitle() {
      wx.setNavigationBarTitle({
        title: "申请退款"
      });
    },
    async getOrderDetails(){
      let result = await post("Order/OrderDetails",{
        UserId:this.userId,
        Token:this.token,
        OrderNo:this.orderNo
      })
        this.info = result.data;
        this.money = result.data.TotalPrice
    },
    // 选择退款原因
    selectReason(id){
      this.reasonlist.map((item,index)=>{
        if(item.id == id.target.value){
          this.content = item.name
          this.contentId = item.id
          // this.reasonlist[index].status = true;
          item.status = true;
        }else{
          item.status = false;
        }
      })
      this.closeMsk();
    },
    // 上传图片
    // upImgs(imgs){
    //   console.log(imgs,'imges')
    //   this.imgs = imgs;
    // },
    closeMsk(){
      this.isshow=false
    },
    choseReason(){
      this.isshow=true
    },
    // 提交
    async submit(){
      const params={
        UserId:this.userId,
        Token:this.token,
        OrderNo:this.orderNo,
        RefundContent:this.text,
        RefundReasonId:this.contentId
      }
      await post('Order/SubmitReturn',params)
      wx.showToast({
        title:'提交成功！'
      })
      setTimeout(()=>{
      wx.navigateBack();
      },1500)
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
.w70{
  div{
  width:70%;
  }
  align-items:center;
  justify-content:flex-start;
}
.payPrice{
  padding:20rpx;
}
</style>
