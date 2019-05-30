<template>
  <div class="backgray">
    <div>
      <!--服务商家-->
      <div class="flex-container clomn ordershophead white">
        <div class="orderserve">服务商家</div>
        <div class="flex-container" v-if="ShopData">
          <img :src="ShopData.Logo" class="ordershopimg">
          <div class="flex-container clomn orderplace">
            <p class="placename">{{ShopData.ShopNick}}</p>
            <p>{{ShopData.Address}}</p>
          </div>
        </div>
        <div class="ordertips" >(温馨提示：请直接至门店进行洗车服务)</div>
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
        
        <div class="flex-container infoslide white pad" @click="selectTimeStatus = true;">
          <div>预约服务时间</div>
          <div>
             <span v-if="datetip&&startTime">{{datetip}} {{startTime}} - {{dateEnd}} {{endTime}}</span>
           <img v-else src="/static/images/back.png" class="right">
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
        </div>
        <div class="flex-container infoslide white pad" @click="choseItem(5)">
          <div>服务卡券</div>
          <div>
            <span v-if="CardTicketName">{{CardTicketName}}</span>
            <img src="/static/images/back.png" class="right" v-else>
          </div>
        </div>
        <div class="infoslide inputbor flex-container white pad">
          <div>联系人</div>
          <input type="text" placeholder="填写内容已和卖家协商确认" class="inputmes" v-model="address.name">
        </div>
        <div class="infoslide inputbor flex-container white pad">
          <div>联系电话</div>
          <input type="text" placeholder="填写内容已和卖家协商确认" class="inputmes" v-model="address.phone">
        </div>


        <div class="infoslide inputbor flex-container white pad">
          <div>买家留言</div>
          <input type="text" placeholder="填写内容已和卖家协商确认" class="inputmes" v-model="Remarks">
        </div>
      </div>
      <div class="fixed">
      <!-- 积分抵扣 -->
       <div class="integral">
        <checkbox-group>
          <label class="checkbox flex-container" @click="scoreStatus = !scoreStatus">
              可用{{score}}积分抵扣{{score/100}}元
            <checkbox value="1" :checked="scoreStatus" color="#ff6325"/>
          </label>
        </checkbox-group>
       </div>
        <!--底部按钮-->
        <div class="botbtn flex-container">
          <div class="price white">
            合计:
            <span>￥{{totalPrice}}</span>
          </div>
          <div class="btnconfir" @click="goPay">提交订单</div>
        </div>
      </div>
      <!-- 选择时间弹窗 -->
      <div class="selectTime" v-show="selectTimeStatus">
        <div class="mask" @click="selectTimeStatus= false"></div>
        <div class="pad">
          <!--预约时间-->
          <div class="appoint">预约时间</div>
          <!--时间-->
          <div class="timeflex">
            <div
              v-for="(item,index) in datelist"
              :key="index"
              :class="{active:active==index}"
              @click="changebg(index)"
            >{{item}}</div>
          </div>
          <div class="freeRoom">
            <div>
              <!--<div class="timeText">{{year}}年{{month}}月{{day}}日</div>-->
              <picker-view class="pickerView" :value="time" @change="bindDateChangeStart">
                <picker-view-column class="pickerColumn">
                  <div class="pickerItem" v-for="(item,key) in hourses" :key="key">{{item}}点</div>
                </picker-view-column>
                <picker-view-column class="pickerColumn">
                  <div class="pickerItem" v-for="(item,key) in minutes" :key="key">{{item}}分</div>
                </picker-view-column>
              </picker-view>
            </div>
          </div>
        </div>
        <!--底部按钮组-->
        <div class="tagflex bottomstyle">
          <div class="btn-cancle" @click="selectTimeStatus = false">取消</div>
          <div class="btn-order" @click="choseTimeSbumit">确认</div>
        </div>
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
  </div>
</template>

<script>
import { get, myget, mypost, post, toLogin } from "../../utils";
import orderCount from "@/components/orderCount.vue";
import Pay from "@/components/pay.vue";
import upImg from "@/components/upImg.vue";
import orderCard from "@/components/orderCard.vue";
import addImgUrl from "../../../static/images/bg20.png";
import "../../css/common.css";
import "../../css/global.css";
export default {
  components: {
    orderCount,
    orderCard,
    Pay,
    upImg
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
      ShopData:{},//商户详情
      CouponId: "0", //优惠券id
      CouponType: "", //优惠券id
      CardTicketId: "0", //服务卡券id
      couponPrice: "0.00",
      CardTicketPrice: "0.00",
      CardTicketName: "",
      Token: "",
      UserId: "",
      orderNumber: "", //订单编号
      shopId: "",
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
      selectTimeStatus: false, //选择时间状态
      datelist: [],
      active: 0,
      hourses: [],
      minutes: [],
          AppointmentStartTime:'',
          AppointmentEndTime:'',
      shopTime: "", //商铺的营业时间
      totalPrice: "", //总价格
      shopTime: "", //商铺的营业时间
      datetip: "", //选中的日期
      dateEnd:'',
      time: [0,0], //时间
      nowhour: "", //当前的时间
      startTime:'',
      endTime:'',
      timeItem:'',
      // 地址
      address: {
        id: "",
        name: "",
        phone: "",
        address: ""
      },
      score:'',//积分数
      scoreStatus:false, // 是否积分抵扣

    };
  },
  onLoad() {
    this.setBarTitle();
    this.initData();
    // 初始化选择时间
    this.choosedate()
    //首次获取默认车辆的信息
    const query = this.$root.$mp.query;
    // 页面传参店铺id
    if (query.shopId) {
      this.shopId = query.shopId;
    }
  },
  onShow() {
    this.Token = wx.getStorageSync("token");
    this.UserId = wx.getStorageSync("userId");

    this.getAddress();// 获取收货地址 
    this.showPay = false;
    //获取vuex商品信息
    this.proid = this.$store.state.visitconfirmorder.ProductId;
    // 获取订单信息
    this.getOrderInfo();
    console.log(this.proid, "proid");
    // this.lat = wx.getStorageSync("latitude");
    // this.lng = wx.getStorageSync("longitude");
    // this.Password = wx.getStorageSync("password");
    const carInfo = wx.getStorageSync("CarInfo");
    // 车辆信息
    if (carInfo) {
      this.CarInfoId = carInfo.Id;
      this.CarInfo = carInfo.CarBrand +' '+ carInfo.CarType +' '+ carInfo.CarColor+' '+carInfo.CarMumber;

      this.getTotal(); //获取订单总金额
    } else {
      // this.CarInfoId=wx.getStorageSync("carId")
      this.getDefaultCar();
    }
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
    // 是否积分抵扣
    scoreStatus(){
      this.getTotal(); //获取订单总金额
    },
    CouponId() {
      if(this.couponPrice){
      let price = this.couponPrice;
      if (this.CouponType == 1) {
        price = "-" + price;
      } else if (this.CouponType == 2) {
        price = price + "折";
      }
      console.log(this.CouponType, "this.CouponType");
      this.couponPrice = price;
      }
      this.getTotal(); //获取订单总金额
    }
  },
  methods: {
    setBarTitle() {
      wx.setNavigationBarTitle({
        title: "确认订单"
      });
    },
    initData() {
      this.CardTicketId = ""; //服务卡券id
      this.CardTicketName = "";
      // this.CardTicketPrice = "0.00";
      this.CouponId = ""; //优惠券id
      this.couponPrice = "";
      this.scoreStatus = false;//是否积分抵扣
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
      this.CarInfo = _res.CarBrand +' '+ _res.CarType +' '+ _res.CarColor+' '+_res.CarMumber;
      this.getTotal(); //获取订单总金额
    },
    //获取到店订单信息
    async getOrderInfo() {
      const result = await post("/Order/ServiceProductsFirmOrder", {
        UserId: this.UserId,
        Token: this.Token,
        ProductId: this.proid
      });
      if (result.code == 0) {
        this.orderinfo = result.data[0];
        this.ShopData = result.data[0].ShopData[0];

        this.CouponData = result.data[0].CouponData;
        this.CardTicketData = result.data[0].CardTicketData;
        this.totalPrice = result.data[0].Price;
        this.score = result.data[0].Score;
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
      if(!this.CarInfoId){
        return false;
      }
      const result = await post("/Order/ServiceProductsPlaceOrderVerifyAmount", {
        UserId: this.UserId,
        Token: this.Token,
        ProductId: this.proid,
        CarInfoId: this.CarInfoId,
        CouponId: this.CouponId,
        CardTicketId: this.CardTicketId,
        Remarks: this.Remarks,
        ScoreId:this.scoreStatus?1:0
      }).catch(() => {
        setTimeout(() => {
          this.initData()
        }, 1500);
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
        const result = await post("/Order/ServiceProductsPlaceOrder", {
          UserId: this.UserId,
          Token: this.Token,
          ProductId: this.proid,
          CarInfoId: this.CarInfoId,
          CouponId: this.CouponId,
          CardTicketId: this.CardTicketId,
          Remarks: this.Remarks,
          ContactName:this.address.name,
          Tel: this.address.phone,
          AppointmentStartTime:this.AppointmentStartTime,
          AppointmentEndTime:this.AppointmentEndTime,
          ScoreId:this.scoreStatus?1:0
        });
        console.log(result, "发起支付请求");
        this.orderNumber = result.data;
        if (result.code * 1 == 200) {
          wx.redirectTo({
            url: "/pages/appointorderdet/main?orderNo=" + result.data
          });
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
    selectItem() {
      //  + this.shopId
      wx.navigateTo({
        url: "/pages/servince/main?shopId=DECACB358531C437"
      });
    },
    // **********收货地址****************************************************
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
    },
    // 跳转选择收货地址
    goSelectAddress() {
      this.$store.commit("setSelectAddress", {
        url: "/pages/visitconfirmorder/main",
        status: true
      });
      wx.navigateTo({ url: "/pages/sitemanage/main" });
    },
    // ********选择时间弹窗函数********************************************************
    changebg(index) {
      this.active = index;
      this.datetip = this.datelist[index];
    },
    // 初始化选择时间
    choosedate() {
      this.datelist = [];
      const ddd = new Date();
      console.log('ddd.getHours()',ddd.getHours())
      this.time[0] = this.nowhour+1 //最近预约时间
      const year = ddd.getFullYear();
      const month = ddd.getMonth() + 1;
      const date = ddd.getDate();
      this.nowhour = ddd.getHours();
      this.nowminute = ddd.getMinutes();
      for (let n = 0; n < 5; n++) {
        let nowMonth = new Date(year, month - 1, date * 1 + n).getMonth() + 1;
        let nowDate = new Date(year, month - 1, date * 1 + n).getDate();
        let i = `${nowMonth}` + "月" + `${nowDate}` + "日";
        this.datelist.push(i);
        // date++;
      }
      this.datetip = this.datelist[0];
      this.gethous();
      this.getMinutes();
    },
    gethous() {
      for (let i = 0; i < 24; i++) {
        if (i.toString().length < 2) {
          i = "0" + i;
        }
        this.hourses.push(i);
      }
      this.time[0] = this.nowhour+1 //最近预约时间
    },
    getMinutes() {
      for (let i = 0; i < 60; i++) {
        if (i.toString().length < 2) {
          i = "0" + i;
        }
      if(i%10==0){
        this.minutes.push(i);
      }
      }
    },
    // 更改时间
    bindDateChangeStart(e) {
      // valIndex 是获取到的年月日在各自集合中的下标
      this.time = e.mp.detail.value;
      // this.time[1] = this.minutes[e.mp.detail.value[1]]
      // console.log(this.time,'e.mp.detail')
      // console.log(JSON.stringify(e.mp.detail.value))
      // let hourses = this.hourses[valIndex[0]]
      // let minutes = this.minutes[valIndex[1]]
      // 滚动时再动态 通过年和月获取 这个月下对应有多少天
    },
    // 确认选择时间
    choseTimeSbumit() {
      console.log(this.datetip, "日期");
      console.log(this.time, "时间");
      // 未修复营业时间
      // const timeStart = this.shopTime.split(" - ")[0].split(":")[0];
      // const timeEnd = this.shopTime.split("-")[1].split(":")[0];
      // console.log(timeStart, timeEnd, this.time, "营业时间");
      // console.log(this.datelist, this.datetip.replace(/月/g, "-"), "日期");
      let datetip = this.datetip;
      let dateArr =[]
      dateArr = datetip.split('月')
      dateArr[1] = dateArr[1].replace(/日/g,'')
      // datetip = datetip.replace(/月/g, "-");
      // datetip = datetip.replace(/日/g, " ");
      const year = new Date().getFullYear();
      let time = [];
      let timeArr = this.time
      timeArr[1] = this.minutes[this.time[1]]
      timeArr.map(t => {
        let ts = "";
        if (String(t).length === 1) {
          ts = "0" + t;
        } else {
          ts = t;
        }
      console.log('ts',ts)
        time.push(ts||'00');
      });
      // 默认预约2小时
      let timeEnd = (time[0]*1)+2
      let dateEnd = dateArr[1]
      console.log(time,this.time, "datetip[datetip.length-2]");
      if(time[0]>21){
        dateEnd =(dateEnd*1)+1 
        }
        if(timeEnd===24){
          timeEnd ='00'
        }else if(timeEnd===25){
          timeEnd ='01'
        }
      const timeItem = year + "-" + dateArr[0] +'-'+dateArr[1]+' ' + time[0] + ":" + time[1] + ":00";
      this.timeItem = timeItem;
      const timeItem2 = year + "-" + dateArr[0] +'-'+dateEnd+' ' + timeEnd + ":" + time[1] + ":00";
      console.log("time", timeItem, timeItem2);
      post("Order/CheckMakeTime", {
        WorkWeek: this.ShopData.WorkWeek,
        WorkTime: this.ShopData.WorkTime,
        AppointmentStartTime: timeItem,
        AppointmentEndTime: timeItem2
      })
        .then(res => {
          // console.log(res, "校验时间");
          this.selectTimeStatus = false;
          this.AppointmentStartTime = timeItem;
          this.AppointmentEndTime = timeItem2;
          this.startTime = time[0] + ":" + time[1];
          this.endTime = timeEnd + ":" + time[1];
          this.dateEnd = dateArr[0] +'月'+dateEnd+'日'
          this.orderInfoStatus = 0;
          // wx.setStorageSync("timearr",timeItem)
          // wx.setStorageSync("datearr",'')
          // this.changeTime();
          // wx.navigateBack({ url: "/pages/location/main" });
        })
        .catch(err => {
          console.log("时间请求失败");
          wx.showToast({
            title: err.data.msg + "!",
            icon: "none",
            duration: 2000
          });
        });
      return false;
    },
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
