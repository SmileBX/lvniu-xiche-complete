<template>
  <div>
    <!--填写订单-->
    <div class="glo-relative">
      <!-- <img src="/static/images/cartbg5.png" class="shopbg"> -->
      <map
        id="map"
        :longitude="longitude"
        :latitude="latitude"
        scale="15"
        :controls="controls"
        :markers="markers"
        @markertap="markertap"
        @regionchange="regionchange"
        @controltap="controltap"
        style="width: 100%; height:480rpx;"
      ></map>
      <!-- show-location -->
    </div>
    <!--填写订单信息列表弹窗-->
    <div class="list" v-show="orderInfoStatus*1===0">
      <div class="p20">
        该商家服务时间为：
        <div>{{shopInfo.WorkWeekSTR}} / {{shopInfo.WorkTime}}</div>
      </div>
      <!-- <div class="item" @click="chosePlace"> -->
      <div class="item">
        <img src="/static/images/yellow.png" class="diandian">
        <span class="location-self">{{nowPlace}}</span>
        <!-- <img src="/static/images/back.png" class="back"> -->
      </div>
      <p class="hr"></p>
      <div class="item" @click="choseItem">
        <img src="/static/images/yellow.png" class="diandian">
        <span class="location-self" style="width:90%;">{{serTip}}</span>
        <img src="/static/images/back.png" class="back">
      </div>
      <p class="hr"></p>
      <div class="item" @click="choseTime">
        <img src="/static/images/yellow.png" class="diandian">
        <!-- <span class="location-self">{{timetip}}</span> -->
        <span class="location-self" v-if="!startTime">请选择服务时间</span>
        <span class="location-self" v-else>{{datetip}} {{startTime}}-{{endTime}}</span>
        <img src="/static/images/back.png" class="back">
      </div>
      <p class="hr"></p>
      <div class="item" @click="choseCar">
        <img src="/static/images/yellow.png" class="diandian">
        <span class="location-self">{{cartip}}</span>
        <img src="/static/images/back.png" class="back">
      </div>
      <p class="hr"></p>
      <div class="item" @click="choseMoney">
        <img src="/static/images/yellow.png" class="diandian">
        <p class="location-self">价格 : {{total}}</p>
        <div style="width:40%; margin-left:230rpx;text-align:right">
          <p class="taginfo">
            <img src="/static/images/tag.png" class="tag">
            <span class="goods">充值有优惠</span>
          </p>
          <img src="/static/images/back.png" class="back">
        </div>
      </div>
      <p class="hr"></p>
      <div class="item">
        <img src="/static/images/yellow.png" class="diandian">
        <p class="location-self">
          姓名
          <input placeholder="请输入您的姓名" v-model="personName">
        </p>
      </div>
      <p class="hr"></p>
      <div class="item">
        <img src="/static/images/yellow.png" class="diandian">
        <p class="location-self">
          手机
          <input placeholder="请输入您的手机号码" v-model="personPhone">
        </p>
      </div>
      <p class="hr"></p>
      <div class="item sign">
        <span @click="addpics">添加现场照片 ({{PicList.length}})</span>
        <span @click="addinfo">备注信息: {{Remarks}}</span>
      </div>
      <div class="chase" @click="toPay">立即下单</div>
    </div>
    <!-- 选择时间弹窗 -->
    <div class="selectTime" v-show="orderInfoStatus*1===1">
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
            <picker-view class="pickerView" :value="value" @change="bindDateChangeStart">
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
              </picker-view-column> -->
            </picker-view>
          </div>
        </div>
      </div>
      <!--底部按钮组-->
      <div class="tagflex bottomstyle">
        <div class="btn-cancle" @click="orderInfoStatus = 0">取消</div>
        <div class="btn-order" @click="choseTimeSbumit">确认</div>
      </div>
    </div>
    <!-- 支付弹窗 -->
    <div class="orderPay" v-show="orderInfoStatus*1===2">
      <!--item-->
      <div class="paymenu">
        <div class="ordertitle">
          <p>确认支付信息</p>
          <img src="/static/images/close.png" class="close" @click="orderInfoStatus = 0">
        </div>
        <p class="menu">￥{{total}}</p>
        <radio-group class="radio-group" @change="radioChange">
          <label class="radio" v-for="item in payitems" :key="item.id">
            <div class="flex-container payitem commonpad">
              <div class="flex-container">
                <img src="/static/images/wx.png" class="payimg" v-if="item.id==1">
                <img src="/static/images/rmbbg.png" class="payimg" v-if="item.id==2">
                <text style="margin-left:20rpx;">{{item.value}}</text>
              </div>
              <radio :value="item.id" :checked="item.checked"/>
            </div>
          </label>
        </radio-group>
        <div class="paybtn" style="margin-top:50rpx;" @click="payMoney">立即支付</div>
      </div>
      <!--密码输入框-->
      <cover-view class="mypopup" v-if="showPass">
        <cover-view class="content">
          <cover-view class="main">
            <cover-view class="close_btn" @click="closePop">X</cover-view>
            <cover-view class="title">请输入支付密码验证</cover-view>
            <cover-view class="pwipt" @click="inputFocus= !inputFocus">
              <input
                type="password"
                :focus="inputFocus"
                @input="judgePassword"
                v-model="password"
                autofocus
                maxlength="6"
              >
              <cover-view class="border-right relative">
                <cover-view class="span" v-if="show1"></cover-view>
              </cover-view>
              <cover-view class="border-right relative">
                <cover-view class="span" v-if="show2"></cover-view>
              </cover-view>
              <cover-view class="border-right relative">
                <cover-view class="span" v-if="show3"></cover-view>
              </cover-view>
              <cover-view class="border-right relative">
                <cover-view class="span" v-if="show4"></cover-view>
              </cover-view>
              <cover-view class="border-right relative">
                <cover-view class="span" v-if="show5"></cover-view>
              </cover-view>
              <cover-view class="relative">
                <cover-view class="span" v-if="show6"></cover-view>
              </cover-view>
            </cover-view>
            <cover-view class="forgetpw" @click="findPass">忘记密码？</cover-view>
          </cover-view>
        </cover-view>
      </cover-view>
    </div>
    <!-- <Pay :showPay.sync="showPay" :orderNumber="orderNumber" 
    :total="total" :successUrl="'/pages/myorder/main'" :closeUrl="'/pages/myorder/main'"
    ></Pay>-->
  </div>
</template>

<script>
import Pay from "@/components/pay.vue";
import "../../css/common.css";
import "../../css/global.css";
import { get, myget, mypost, post, toLogin } from "../../utils";
import { mapState, mapMutations } from "vuex"; //vuex辅助函数
export default {
  components: {
    Pay
  },
  data() {
    return {
      orderInfoStatus: 0, //填写订单信息列表弹窗 0--下单信息；1--选择时间；2--下单支付
      showPay: false,
      address: "",
      serTip: "", //服务项目
      timetip: "", //服务时间
      cartip: "", //车辆
      total: "", //总价
      userId: "",
      token: "",
      shopId: "",
      shopInfo: {},
      startTime: "",
      endTime: "",
      orderNo: "", //订单编号
      personName: "", //姓名
      personPhone: "", //手机
      CarInfoId: "", //车子id
      ServiceItem: "", //服务项目id
      AppointmentStartTime: "", //预约开始时间
      AppointmentEndTime: "", //预约结束时间
      PicList: "", //图片合集; JsonString格式(多图)
      Remarks: "", //备注
      controls: [
        {
          //控件不随着地图移动
          id: 1,
          iconPath: "/static/images/location.png",
          position: {
            left: 0,
            top: 100,
            width: 30,
            height: 30
          },
          clickable: true
        }
      ],
      latitude: 0,
      longitude: 0,
      markers: [
        {
          iconPath: "/static/images/person.png",
          id: 1,
          latitude: this.latitude,
          longitude: this.longitude,
          width: 40,
          height: 45
        }
      ],
      // 支付弹窗数据***********
      // orderPayStatus:true,  //支付弹窗
      payitems: [
        { id: 1, value: "微信支付", checked: "true" },
        { id: 2, value: "余额支付" }
      ],
      // latitude: wx.getStorageSync("latitude"),
      // longitude: wx.getStorageSync("longitude"),
      showPass: false, //密码输入框
      // orderNo:"",//订单编号
      price: "",
      userId: "",
      token: "",
      password: "", //会员支付密码
      autofocus: true,
      show1: false,
      show2: false,
      show3: false,
      show4: false,
      show5: false,
      show6: false,
      //选择时间弹窗数据******************
      datelist: [],
      active: "0",
      hourses: [],
      minutes: [],
      shopTime: "", //商铺的营业时间
      datetip: "", //选中的日期
      time: [0, 0, 0, 0], //时间
      nowhour: "", //当前的时间
      nowminute: "",
      inputFocus: false
    };
  },

  watch: {
    //图片
    "$store.state.pList": {
      // "$store.state.upImgPathArr": {

      handler: function() {
        const state = this.$store.state;
        this.PicList = state.pList;
        console.log(this.PicList, "添加图片");
      },
      deep: true
    },
    "$store.state.nowPlace": {
      handler: function() {
        const state = this.$store.state;
        this.address = state.nowPlace;
        //console.log(this.address, "地址");
      },
      deep: true
    }
  },
  computed: {
    ...mapState(["cityName", "nowPlace", "longitude", "latitude"])
  },
  onLoad() {
    this.setBarTitle();
    // 初始化缓存---用于提交订单数据
    this.initSubmitInfo();
    this.orderInfoStatus = 0;
    this.personName = "";
    this.personPhone = "";
    this.ServiceItem = "";
    this.AppointmentStartTime = "";
    this.AppointmentEndTime = "";
    this.PicList = "";
    this.Remarks = "";
    this.cartip = "请选择车辆";
    this.timetip = "请选择服务时间";
    this.serTip = "请选择服务项目";
    this.total = "0.00";
    this.datetip = "";
    this.startTime = "";
    this.endTime = "";
    console.log("initdate");
  },
  onShow() {
    this.userId = wx.getStorageSync("userId");
    this.token = wx.getStorageSync("token");
    this.shopInfo = wx.getStorageSync("shopInfo");
    console.log(this.shopInfo, "商户信息***********");
    this.showPay = false;
    this.inputFocus = false;
    this.showPass = false;
    // this.orderInfoStatus=2;
    this.shopId = this.shopInfo.id;
    this.latitude = this.$store.state.latitude;
    this.longitude = this.$store.state.longitude;
    
          this.markers = [
            {
              iconPath: "/static/images/person.png",
              id: 1,
              latitude: this.latitude,
              longitude: this.longitude,
              width: 40,
              height: 45
            }
          ];
    // 获取定位坐标位置
    // this.getMapInfo();
    // 图片
    // const state = this.$store.state;
    this.PicList = this.$store.state.upImgPathArr;
    console.log(this.PicList.length, "展示的图片数量");
    if (wx.getStorageSync("CarInfo").Id) {
      //获取车辆信息
      this.getCar();
    } else {
      console.log(wx.getStorageSync("carId"), "车辆信息");
      this.CarInfoId = wx.getStorageSync("carId");
      //获取默认车辆的信息
      this.getDefaultCar();
    }
    //转换时间
    // this.changeTime();
    //服务项目
    this.getSerItem();
    //备注
    const Remarks = wx.getStorageSync("remarks");
    if (Remarks) {
      this.Remarks = Remarks;
      //console.log(this.Remarks,"接收的备注")
    }

    //获取上门服务的地址
    this.address = this.$store.state.nowPlace;
  },

  methods: {
    ...mapMutations(["update"]),
    setBarTitle() {
      wx.setNavigationBarTitle({
        title: "填写订单"
      });
    },
      //首次进入页面获取手机所在地城市经纬度
    getMapInfo() {
      const that = this;
      wx.getLocation({
        type: "gcj02",
        success: data => {
          // console.log(data,"微信地图")
          that.latitude = data.latitude;
          that.longitude = data.longitude;
          that.markers = [
            {
              iconPath: "/static/images/person.png",
              id: 1,
              latitude: that.latitude,
              longitude: that.longitude,
              width: 40,
              height: 45
            }
          ];
        }
      });
    },

    async getDefaultCar() {
      let res = await post("/User/GetCarInfo", {
        UserId: this.userId,
        Token: this.token,
        IsDefault: 1
      });
      console.log(res, "获取默认车辆");
      if (res.code == 0) {
        const _res = res.data[0];
        this.CarInfoId = _res.Id;
        this.cartip = _res.CarBrand + _res.CarType + _res.CarMumber;
      }
      console.log(this.CarInfoId, "默认车辆id");
    },
    //获取车辆
    getCar() {
      const car = wx.getStorageSync("CarInfo");
      if (typeof car == "object") {
        this.cartip =
          car.CarBrand +
          " " +
          car.CarType +
          " " +
          car.CarColor +
          " " +
          car.CarMumber;
        this.CarInfoId = car.Id;
      }

      //console.log(this.cartip)
    },
    //转换服务项目
    getSerItem() {
      const Service = wx.getStorageSync("serItem");
      if (typeof Service == "object" && Service.length > 0) {
        const SerItembox = [];
        const SerTipbox = [];
        let Serpricebox = 0;
        for (let i = 0; i < Service.length; i++) {
          SerItembox.push(Service[i].Id);
          SerTipbox.push(Service[i].Name);
          //console.log(typeof(this.Service[i].Price ))
          Serpricebox += Service[i].Price * 1 || 0;
        }
        this.total = Serpricebox.toFixed(2);
        this.serTip = SerTipbox.join(" ");
        this.ServiceItem = SerItembox.join(",");
      }
    },
    //转换时间格式
    changeTime() {
      const d = new Date();
      const year = d.getFullYear();
      let _time = wx.getStorageSync("timearr"); //时间
      let _date = wx.getStorageSync("datearr"); //日期
      //console.log(typeof _time, typeof _date,_time, _date,"页面接收");

      if (typeof _time === "object") {
        if (_time.length > 0) {
          let _mon = _date.split("月")[0]; //月份
          let _dd = _date.split("月")[1].split("日")[0]; //日期
          _mon.length < 2 ? (_mon = "0" + _mon) : _mon;
          _dd.length < 2 ? (_dd = "0" + _dd) : _dd;
          const tt = _mon + "-" + _dd;
          const time1 = [];
          for (let i of _time) {
            if (i.toString().length < 2) {
              i = "0" + i;
            }
            time1.push(i);
          }

          let mm = time1[0] + ":" + time1[1] + "-" + time1[2] + ":" + time1[3]; //时间
          if (tt && mm) {
            this.timetip = tt + "  " + mm;
            //开始时间结束时间
            this.AppointmentStartTime =
              year + "-" + tt + " " + mm.split("-")[0];
            this.AppointmentEndTime = year + "-" + tt + " " + mm.split("-")[1];
          }
          //console.log(this.AppointmentStartTime);
        }
      }
    },
    chosePlace() {
      wx.navigateTo({ url: "/pages/locationorder/main" });
    },
    choseItem() {
      //选择服务项目
      wx.navigateTo({ url: "/pages/servince/main?shopId=" + this.shopId });
    },
    choseMoney() {
      wx.navigateTo({ url: "/pages/sum/main?url=location" }); //充值
    },
    choseTime() {
      // wx.navigateTo({ url: "/pages/writeorder/main" }); //时间

      this.choosedate();
      this.gethous();
      this.getMinutes();
      this.datetip = this.datelist[this.active];
      this.orderInfoStatus = 1;
    },
    choseCar() {
      wx.navigateTo({ url: "/pages/mycar/main?url=location" }); //我的车辆
    },
    addpics() {
      wx.navigateTo({ url: "/pages/sceneplace/main" }); //添加图片
    },
    addinfo() {
      wx.navigateTo({ url: "/pages/signinfo/main" }); //添加备注
    },
    //验证手机号
    checkPhone() {
      const phoneNum = this.personPhone;
      if (!/^1[3|4|5|6|7|8][0-9]\d{8}$/.test(phoneNum)) {
        wx.showToast({
          title: "请输入正确的手机号码！",
          icon: "none",
          duration: 2000
        });
        // this.personPhone = "";
        return false;
      }
      return true;
    },
    async toPay() {
      console.log(this.PicList, "this.PicList");
      // return false
      if (
        this.CarInfoId &&
        this.ServiceItem &&
        this.AppointmentEndTime &&
        this.personName
      ) {
        // const _picList = JSON.stringify(this.PicList);
        //console.log(this.latitude, this.address, "经度信息");

        // 判断手机号
        if (!this.checkPhone()) {
          return false;
        }
        this.orderInfoStatus = 2;
      } else {
        wx.showToast({
          title: "带红点为必填内容！",
          icon: "none"
        });
        return false;
      }
    },
    // 初始化下单缓存和vuex---用于提交订单数据
    initSubmitInfo() {
      wx.setStorageSync("serItem", []);
      wx.setStorageSync("CarInfo", "");
      wx.setStorageSync("timearr", "");
      wx.setStorageSync("datearr", "");
      wx.setStorageSync("remarks", "");
      this.$store.commit("update", { pList: [] });
    },

    // ******************************************支付弹窗*****************************************
    // 选择支付方式
    radioChange: function(e) {
      for (const x in this.payitems) {
        if (this.payitems[x].id == e.target.value) {
          this.payitems[x].checked = true;
        } else {
          this.payitems[x].checked = false;
        }
      }
    },
    // 根据临时路径返回base64码
    upDateImg() {
      return new Promise((resolved, rejected) => {
        const imgBase = [];
        if (this.PicList.length < 1) {
          resolved(imgBase);
        }
        for (let i = 0; i < this.PicList.length; i += 1) {
          wx.getFileSystemManager().readFile({
            filePath: this.PicList[i], //选择图片返回的相对路径
            encoding: "base64", //编码格式
            success: ress => {
              //成功的回调
              imgBase.push({
                PicUrl: "data:image/png;base64," + ress.data.toString()
              });
              // console.log("上传后的图片", imgBase);
              if (i === this.PicList.length - 1) {
                console.log("imgBase", imgBase);
                resolved(imgBase);
              }
            },
            fail(err) {
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
    // 支付，创建订单
    async payMoney() {
      // 去获取base64码
      const picList = await this.upDateImg();
      console.log("picList", picList.length);
      if (picList.length !== this.PicList.length) {
        wx.showToast({
          title: "图片上传中，请稍等！",
          icon: "none"
        });
        return false;
      }
      const params = {
        UserId: this.userId,
        Token: this.token,
        Lat: this.latitude,
        Lng: this.longitude,
        Addr: this.address, //地址
        CarInfoId: this.CarInfoId, //车辆信息
        Remarks: this.Remarks, //备注
        ServiceItem: this.ServiceItem, //服务项目
        ContactName: this.personName, //名字
        Tel: this.personPhone, //手机号
        pList: JSON.stringify(picList),
        AppointmentStartTime: this.AppointmentStartTime, //开始时间
        AppointmentEndTime: this.AppointmentEndTime //结束时间
      };
      // const numsTure=this.userId && this.token && this.latitude && this.longitude && this.address && this.CarInfoId  && this.ServiceItem && this.AppointmentStartTime && this.AppointmentEndTime && this.personName && this.personPhone

      const res = await post("Order/SubmitDoorOrders", params);
      //如果有订单编号跳转支付页面
      if (res.code == 0) {
        this.orderNo = res.data;
        //  下单成功， 清空缓存数据，
        // this.initSubmitInfo()
        //this.showPay = true; //支付组件
        // wx.navigateTo({url:"/pages/orderpay/main?price="+this.total+"&orderNo="+this.orderNumber})
      }
      //判断哪种支付方法
      for (let i = 0; i < this.payitems.length; i++) {
        if (this.payitems[i].checked) {
          console.log(i, "哪种支付方式");
          if (i == 0) {
            this.wxPay();
          } else {
            this.showPass = true;
            // this.otherPay()
          }
        }
      }
    },
    // 微信支付
    async wxPay() {
      var res = await post("/Order/ConfirmWeiXinSmallPay", {
        UserId: this.userId,
        Token: this.token,
        //OrderNo:"201904111028561742883"
        OrderNo: this.orderNo
      }).catch(() => {
        setTimeout(() => {
          wx.redirectTo({
            url: `/pages/appointorderdet/main?orderNo=${this.orderNo}`
          });
        }, 1500);
      });
      console.log(res, "微信支付");
      if (res.code == 0) {
        let payData = JSON.parse(res.data.JsParam);
        wx.requestPayment({
          timeStamp: payData.timeStamp,
          nonceStr: payData.nonceStr,
          package: payData.package,
          signType: payData.signType,
          paySign: payData.paySign,
          success(res) {
            wx.redirectTo({
              url: `/pages/appointorderdet/main?orderNo=${this.orderNo}`
            });
          },
          fail() {
            wx.redirectTo({
              url: `/pages/appointorderdet/main?orderNo=${this.orderNo}`
            });
          }
        });
      } else {
        setTimeout(() => {
          wx.redirectTo({
            url: `/pages/appointorderdet/main?orderNo=${this.orderNo}`
          });
        }, 1500);
      }
    },
    // 余额支付
    async otherPay() {
      console.log(this.password, "支付密码");
      var res = await post("/Order/PaymentOrder", {
        UserId: this.userId,
        Token: this.token,
        //OrderNo:"201904111028561742883",
        Password: this.password, //会员支付密码
        OrderNo: this.orderNo
      });
      // 初始化密码框
      this.password = "";
      this.judgePassword();
      console.log(res, "余额支付");
      if (res.code == 0) {
        //余额支付成功
        wx.redirectTo({
          url: `/pages/appointorderdet/main?orderNo=${this.orderNo}`
        });
      } else if (res.code == "9") {
        this.showPass = false;
      }
    },
    // *************余额支付输入密码控件*********
    //关闭密码输入清空输入
    closePop: function() {
      this.password = "";
      this.judgePassword();
      wx.redirectTo({
        url: `/pages/appointorderdet/main?orderNo=${this.orderNo}`
      });
    },
    findPass() {
      wx.navigateTo({ url: "/pages/setpassword/main?url=orderpay" });
    },
    judgePassword: function() {
      console.log(this.password, this.password.length);
      if (this.password.length == 0) {
        this.show1 = false;
        this.show2 = false;
        this.show3 = false;
        this.show4 = false;
        this.show5 = false;
        this.show6 = false;
      }
      if (this.password.length == 1) {
        this.show1 = true;
        this.show2 = false;
        this.show3 = false;
        this.show4 = false;
        this.show5 = false;
        this.show6 = false;
      }
      if (this.password.length == 2) {
        this.show1 = true;
        this.show2 = true;
        this.show3 = false;
        this.show4 = false;
        this.show5 = false;
        this.show6 = false;
      }
      if (this.password.length == 3) {
        this.show1 = true;
        this.show2 = true;
        this.show3 = true;
        this.show4 = false;
        this.show5 = false;
        this.show6 = false;
      }
      if (this.password.length == 4) {
        this.show1 = true;
        this.show2 = true;
        this.show3 = true;
        this.show4 = true;
        this.show5 = false;
        this.show6 = false;
      }
      if (this.password.length == 5) {
        this.show1 = true;
        this.show2 = true;
        this.show3 = true;
        this.show4 = true;
        this.show5 = true;
        this.show6 = false;
      }
      if (this.password.length == 6) {
        this.show1 = true;
        this.show2 = true;
        this.show3 = true;
        this.show4 = true;
        this.show5 = true;
        this.show6 = true;
        console.log(this.show6);
        // 接口校验密码
        this.otherPay();
      }
    },
    // ********选择时间弹窗函数********************************************************
    changebg(item) {
      this.active = item;
      this.datetip = this.datelist[this.active];
    },
    choosedate() {
      this.datelist = [];
      const ddd = new Date();
      //console.log(date)
      const year = ddd.getFullYear()
      const month = ddd.getMonth() + 1;
      const date = ddd.getDate();
      this.nowhour = ddd.getHours();
      this.nowminute = ddd.getMinutes();
      for (let n = 0; n < 5; n++) {
        let nowMonth = new Date(year,month-1,date*1+n).getMonth()+1
        let nowDate = new Date(year,month-1,date*1+n).getDate()
        let i = `${nowMonth}` + "月" + `${nowDate}` + "日";
        this.datelist.push(i);
        // date++;
      }
    },
    gethous() {
      for (let i = 0; i < 24; i++) {
        if (i.toString().length < 2) {
          i = "0" + i;
        }
        this.hourses.push(i);
      }
    },
    getMinutes() {
      for (let i = 0; i < 60; i++) {
        if (i.toString().length < 2) {
          i = "0" + i;
        }
        this.minutes.push(i);
      }
    },
    // 更改时间
    bindDateChangeStart(e) {
      //console.log(e)
      // valIndex 是获取到的年月日在各自集合中的下标
      this.time = e.mp.detail.value;
      // console.log(JSON.stringify(e.mp.detail.value))
      // let hourses = this.hourses[valIndex[0]]
      // let minutes = this.minutes[valIndex[1]]
      // 滚动时再动态 通过年和月获取 这个月下对应有多少天
    },
    // 确认选择时间
    choseTimeSbumit() {
      // console.log(this.shopTime,"日期")
      //console.log(this.time,"时间")
      // 未修复营业时间
      this.shopTime = this.shopInfo.WorkTime;
      const timeStart = this.shopTime.split(" - ")[0].split(":")[0];
      const timeEnd = this.shopTime.split("-")[1].split(":")[0];
      // console.log(timeStart, timeEnd, this.time, "营业时间");
      // console.log(this.datelist, this.datetip.replace(/月/g, "-"), "日期");
      let datetip = this.datetip;
      datetip = datetip.replace(/月/g, "-");
      datetip = datetip.replace(/日/g, " ");
      // console.log(this.time, "月份");
      const year = new Date().getFullYear();
      let time = [];
      this.time.map(t => {
        let ts = "";
        if (String(t).length === 1) {
          ts = "0" + t;
        } else {
          ts = t;
        }
        time.push(ts);
      });
      const timeItem = year + "-" + datetip + time[0] + ":" + time[1] + ":00";
      const timeItem2 = year + "-" + datetip + time[2] + ":" + time[3] + ":00";
      // console.log("time", timeItem, timeItem2);
      post("Order/CheckMakeTime", {
        WorkWeek: this.shopInfo.WorkWeek,
        WorkTime: this.shopInfo.WorkTime,
        AppointmentStartTime: timeItem,
        AppointmentEndTime: timeItem2
      }).then(res => {
        // console.log(res, "校验时间");
        this.AppointmentStartTime = timeItem;
        this.AppointmentEndTime = timeItem2;
        this.startTime = time[0] + ":" + time[1];
        this.endTime = time[2] + ":" + time[3];
        this.orderInfoStatus = 0;
        // wx.setStorageSync("timearr",timeItem)
        // wx.setStorageSync("datearr",'')
        // this.changeTime();
        // wx.navigateBack({ url: "/pages/location/main" });
      }).catch((err)=>{
        console.log('时间请求失败')
                        wx.showToast({
                            title: err.data.msg + '!',
                            icon: 'none',
                            duration:2000
                        })
      });
      return false;
      // let test=(this.time[2]*60 + this.time[3]) - (this.time[0]*60 + this.time[1])
      //  if(this.time[2] >timeEnd || this.time[0] < timeStart ){
      //  wx.showToast({
      //     title: '您选择的时间不在营业范围内',
      //     icon: 'none',
      //     duration: 2000,
      //     complete: function (){

      //     }
      // });
      // return false
      // }else

      // if(test >120){
      //  wx.showToast({
      //     title: '服务时间不超过两个小时哦。。。',
      //     icon: 'none',
      //     duration: 2000,
      //     complete: function (){

      //     }
      // });
      // return false
      //  }else if(this.active==0 && this.time[0]<=this.nowhour && this.time[1] <this.nowminute){
      //     wx.showToast({
      //       title: '您选择的时间已过...',
      //       icon: 'none',
      //       duration: 2000,
      //       complete: function (){

      //       }
      //   });
      // return false
      //  }else if(this.time[0]>this.time[2]){
      //    wx.showToast({
      //       title: '开始时间不能大于结束时间',
      //       icon: 'none',
      //       duration: 2000,
      //       complete: function (){

      //       }
      // });
      // return false
      // }else{
      //     if(this.datetip && this.time.length>0){
      //     console.log(this.datetip,this.time,'选择后的时间')
      //       wx.setStorageSync("timearr",this.time)
      //       wx.setStorageSync("datearr",this.datetip)
      //       this.changeTime();
      //       // wx.navigateBack({ url: "/pages/location/main" });
      //       // 显示下单信息弹窗
      //       this.orderInfoStatus=0;
      //     }else{
      //           wx.showToast({
      //             title: '请选择时间',
      //             icon: 'none',
      //             duration: 2000,
      //             complete: function (){

      //             }
      //         });
      //         return false
      //       }

      //     }
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
.p20 {
  padding: 20rpx;
}
</style>
