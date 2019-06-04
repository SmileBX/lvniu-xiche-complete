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
        <div class="ordertips">(温馨提示：请直接至门店进行洗车服务)</div>
      </div>
      <div class="slide"></div>
      <!-- 服务地址 -->
      <div class="flex-container clomn orderhead white" @click="goSelectAddress">
        <div class="orderserve">服务地址</div>
        <div class="orderuser">
          <p>联系人：{{address.name}}</p>
          <p>{{address.phone}}</p>
        </div>
        <div class="flex-container addressContent">
          <img src="/static/images/place.png" class="place">
          <text class="placeinfo">详细地址：{{address.address}}</text>
          <img src="/static/images/back.png" class="right">
        </div>
        <div class="ordertips">(温馨提示：请核对服务地址是否正确)</div>
        <div class="slide"></div>
      </div>
      <!--订单详情-->
      <div class="white proinfo">
        <div class="orderserve">服务项目</div>
        <div class="flex-container ordermain" 
        v-for="(productItem,productIndex) in xicheProductArr" :key="productIndex">
          <img :src="productItem.PicNo" class="ordershopimg">
          <div class="flex-container clomn orderplace">
            <p class="placename detailright">{{productItem.Name}}</p>
            <p>￥{{productItem.Price}}</p>
          </div>
        </div>
        <!-- 上门订单 -->
        <div class="flex-container infoslide white pad" @click="selectItem" >
          <div>增加服务项目</div>
          <div>
            <img src="/static/images/back.png" class="right">
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
          <div>优惠券</div>
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
        <div class="ml-20" >
          <upImg title="添加现场照片" :imgLenght="8" :addImgUrl="addImgUrl" @upImgs="upImgSuccess"></upImg>
        </div>
        <div class="inputbor flex-container white pad bt-eee" >
          <div>备注信息</div>
        </div>
        <div class="infoslide textarea" >
          <textarea
            cols="20"
            rows="10"
            class="infobg"
            placeholder="可以把停车位置写的更详细，可以把想跟洗车员传达的信息都写在这里"
            v-model="textInfo"
            @focus="textareaFocus"
            @blur="textareaBlur"
            :focus="textInfoStatus"
            v-if="textInfoStatus"
          ></textarea>
          <div v-else @click="textareaFocus" class="infobg p10" :class="{'c999':textInfo =='可以把停车位置写的更详细，可以把想跟洗车员传达的信息都写在这里'}">{{textInfo}}</div>
        </div>
      </div>
      <div class="fixed">
      <!-- 积分抵扣 -->
       <div class="integral">
        <checkbox-group >
          <label class="checkbox flex-container" @click="scoreStatus = !scoreStatus">
              可用{{score}}积分抵扣{{score/100}}元
            <checkbox value="1" :checked="scoreStatus"  color="#ff6325"/>
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
          <!--开始时间 结束时间-->
          <!-- <div class="tagflex"> -->
          <!-- <div>开始时间</div> -->
          <!-- <div>结束时间</div> -->
          <!-- </div> -->
          <!--滑动选择时间-->
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
                <!-- <picker-view-column class="pickerColumn">
                <div class="pickerItem" v-for="(item,key) in hourses" :key="key">{{item}}点</div>
              </picker-view-column>
              <picker-view-column class="pickerColumn">
                <div class="pickerItem" v-for="(item,key) in minutes" :key="key">{{item}}分</div>
                </picker-view-column>-->
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
import QQMapWX from "@/utils/qqmap-wx-jssdk"; //腾讯地图，reverseGeocoder逆地址转码
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
      qqmapsdk: null, //实例化地图sdk
      latitude:'',
      longitude:'',
      showPay: false,
      couponList: [], //子组件展示信息
      CardTicketData: [], //卡券列表
      CouponData: [], //优惠券列表
      isshow: true,
      proid: "", //产品id
      CarInfoId: "", //车辆id
      CarInfo: "",
      CouponId: "", //优惠券id
      CouponType: "", //优惠券id
      CardTicketId: "0", //服务卡券id
      couponPrice: "",
      CardTicketPrice: "0.00",
      CardTicketName: "",
      Token: "",
      UserId: "",
      orderNumber: "", //订单编号
      shopId: "",
      // Password: "",
      Remarks: "",
      orderinfo: {},
      ShopData:{},
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
      totalPrice: "", //总价格
      selectTimeStatus: false, //选择时间状态
      datelist: [],
      active: 0,
      hourses: [],
      minutes: [],
      shopTime: "", //商铺的营业时间
      datetip: "", //选中的日期
      dateEnd:'', //选中的结束日期
      time: [0,0], //时间
      nowhour: "", //当前的时间
      addImgUrl, //上传图片按钮图片,
      picListPath:[], //上传图片的数组
      startTime:'',
      endTime:'',
      timeItem:'',
      xicheProductArr: [],
      isXiche: false,
      // 地址
      address: {
        id: "",
        name: "",
        phone: "",
        address: ""
      },
      textInfo:'',//留言
      textInfoStatus:false,
      score:'',//积分数
      scoreStatus:false, // 是否积分抵扣
    };
  },
  onLoad() {
    this.setBarTitle();
    this.initData();
    this.choosedate()
    // 实例化API核心类
    this.qqmapsdk = new QQMapWX({
      key: "3P2BZ-6G4WD-CEX43-PIV5G-3VDYH-N5BGH" // 必填
    });
    //首次获取默认车辆的信息
    // const query = this.$root.$mp.query;
    this.latitude = this.$store.state.latitude;
    this.longitude = this.$store.state.longitude;
    this.textInfo='可以把停车位置写的更详细，可以把想跟洗车员传达的信息都写在这里'
    // 页面传参店铺id
    // if (query.shopId) {
    //   this.shopId = query.shopId;
    // }
    // // 如果洗车的时候，跳转选择项目
    // if (query.type && query.type == "洗车") {
    //   console.log(query.type, this.shopId, "query.type");
    //   wx.navigateTo({
    //     url: "/pages/servince/main?shopId=" + this.shopId
    //   });
    //   return false;
    // }
  },
  onShow() {
    this.Token = wx.getStorageSync("token");
    this.UserId = wx.getStorageSync("userId");
    // 洗车下单
      this.xicheProductArr = [];
      const serItem = wx.getStorageSync("serItem");
      console.log(serItem, "选中的洗车项目");
      
      // 存在选择的项目,不存在项目返回上一页
      if (serItem && serItem.length > 0) {
        let proId =[];
        serItem.map(item=>{
          proId.push(item.Id)
        })
        this.proid =proId.join(',')
      console.log( this.proid, "选中的洗车项目ID");
        this.xicheProductArr = serItem;
        // 获取订单信息
        this.getXicheOrder();
      } else {
        wx.showToast({
          title: "请选择服务项目!",
          icon:'none'
        });
        // wx.navigateBack()
        return false;
      }

    this.getAddress();// 获取收货地址 
    this.showPay = false;
    const carInfo = wx.getStorageSync("CarInfo");
    console.log(carInfo, "carInfo");
    // 车辆信息
    if (carInfo) {
      this.CarInfoId = carInfo.Id;
      this.CarInfo = carInfo.CarBrand +' '+ carInfo.CarType +' '+ carInfo.CarColor+' '+carInfo.CarMumber;

    } else {
      this.getDefaultCar();
    }
  },
  watch: {
    CardTicketId() {
      this.getTotal(); //获取订单总金额
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
      // this.ShopData={}
      // this.CardTicketPrice = "0.00";
      this.CouponId = ""; //优惠券id
      this.couponPrice = "";
      this.scoreStatus = false;//是否积分抵扣
    },
    // 查询默认车辆
    async getDefaultCar() {
      let res = await post("/User/GetCarInfo", {
        UserId: this.UserId,
        Token: this.Token,
        IsDefault: 1
      });
      const _res = res.data[0];
      this.CarInfoId = _res.Id;
      this.CarInfo = _res.CarBrand +' '+ _res.CarType +' '+ _res.CarColor+' '+_res.CarMumber;

      this.getTotal(); //获取订单总金额
    },
    // 获取洗车订单
    async getXicheOrder(){
      this.initData()
      const result = await post("Order/DoorProductsFirmOrder", {
        UserId: this.UserId,
        Token: this.Token,
        ProductList: this.proid
      });
      if (result.code == 0) {
        this.orderinfo = result.data;
        this.ShopData = result.data.Data[0].ShopData[0];
        this.CouponData = result.data.CouponData;
        this.CardTicketData = result.data.CardTicketData;
        this.score = result.data.MemberScore;
        console.log(this.orderinfo.ShopData,this.ShopDadta, "确认页面详情");
      }
      this.getTotal(); //获取订单总金额
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
      if(!this.CarInfoId){
        return false;
      }
      const result = await post("Order/DoorProductsPlaceOrderVerifyAmount", {
        UserId: this.UserId,
        Token: this.Token,
        ProductList: this.proid,
        CarInfoId: this.CarInfoId,
        CouponId: this.CouponId||0,
        CardTicketId: this.CardTicketId||0,
        ScoreId:this.scoreStatus?1:0,
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
      //提交支付
    async goPay() {
      console.log(
        this.UserId,
        this.proid,
        this.CarInfoId,
        this.CouponId,
        this.CardTicketId,
        "提交信息"
      );
      if (this.CarInfoId) {
        const imgArr = await this.upDateImg()
        this.isshow = false;
        const result = await post("Order/SubmitDoorOrders", {
          UserId: this.UserId,
          Token: this.Token,
          ServiceItem: this.proid,
          CarInfoId: this.CarInfoId,
          CouponId: this.CouponId||0,
          CardTicketId: this.CardTicketId||0,
          Remarks: this.textInfo,
          Addr:this.address.address,
          ContactName:this.address.name,
          Tel:this.address.phone,
          LaT:this.latitude,
          Lng:this.longitude,
          PicList:JSON.stringify(imgArr),
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
            console.log("地址转码开始2");
        let address = await this.getCityinfo()
        res.data.addressinfo = address;
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
    //根据经纬度获取城市名称nowPlace 反地理转码
    getCityinfo() {
            console.log("地址转码开始");
      return new Promise((resolved, rejected) => {
        const that = this;
        this.qqmapsdk.reverseGeocoder({
          location: {
            latitude: this.latitude,
            longitude: this.longitude
          },
          success(res) {
            console.log("地址转码成功", res);
            const _res = res.result;
            let address = _res.formatted_addresses.recommend + " - " + _res.address
            resolved(address)
          },
          fail: function(err) {
            console.log(res);
            rejected(err)
          }
        });
      });
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
    // *****************选择图片*****************************************************
    upImgSuccess(base,arr){
      console.log(arr,'图片')
      this.picListPath = arr
    },
    // 根据临时路径返回base64码
    upDateImg() {
      return new Promise((resolved, rejected) => {
        const imgBase = [];
        if (this.picListPath.length < 1) {
          resolved(imgBase);
        }
        for (let i = 0; i < this.picListPath.length; i += 1) {
          wx.getFileSystemManager().readFile({
            filePath: this.picListPath[i], //选择图片返回的相对路径
            encoding: "base64", //编码格式
            success: ress => {
              //成功的回调
              imgBase.push({
                PicUrl: "data:image/png;base64," + ress.data.toString()
              });
              if (i === this.picListPath.length - 1) {
                // console.log("imgBase", imgBase);
                resolved(imgBase);
              }
            },
            fail(err) {
              // console.log("上传失败的图片", err);
              wx.showToast({
                title: "图片上传失败！",
                icon: "none"
              });
              rejected();
            }
          });
        }
      });
    },
    // ****************textarea输入框*************************
    // 聚焦
    textareaFocus(){

      if(this.textInfo =='可以把停车位置写的更详细，可以把想跟洗车员传达的信息都写在这里'){
        this.textInfo =''
      }
      this.textInfoStatus = true;
    },
    // 失去焦点
    textareaBlur(){
      if(!this.textInfo){
        this.textInfo ='可以把停车位置写的更详细，可以把想跟洗车员传达的信息都写在这里'
      }
      this.textInfoStatus = false;
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
