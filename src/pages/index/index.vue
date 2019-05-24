<template>
  <div class="backgray">
    <div>
      <!--顶部导航栏-->
      <div class="head white">
        <div @click="goTo(1)">
          <span class="province">{{cityName}}</span>
          <img src="/static/images/bottom.png" class="img-bottom">
        </div>
        <div
          v-for="item in titlelist"
          :key="item.name"
          :class="{active:active==item.name}"
          @click="change(item.name)"
        >{{item.name}}</div>
        <div class="list-img" @click="goTo(2)">
          <img src="/static/images/right.png" class="container-img">
          <img src="/static/images/list.png" class="list">
        </div>
      </div>
      <div class="location glo-relative">
        <!-- <map id="map" 
              :longitude="longitude"  地图中心经度
              :latitude="latitude"  地图中心纬度
              scale="10"  地图放大倍数
              :controls="controls"  地图控件，--我的位置
              @controltap="controltap" 地图控件点击事件
              :markers="markers"  地图标记
              @markertap="markertap"    地图标记点击事件--点击之后
              @regionchange="regionchange"  视野发生变化时 
              @end="getCenterMap"  视野发生变化时 
              show-location 
              style="width: 750rpx; height: 1000rpx;">
        </map>-->
        <map
          id="map"
          :longitude="longitude"
          :latitude="latitude"
          scale="13"
          :markers="markers"
          @markertap="markertap"
          @regionchange="getCenterMap1"
          @end="getCenterMap"
          show-location
          style="width:750rpx; height:99vh;"
        ></map>
        <!-- 中心点 -->
        <!-- v-show="active==='上门'" -->
        <cover-image v-show="active==='上门'"  class="centerImg" src="/static/images/person.png"></cover-image>
        <cover-image @click="getMyPosition" class="backMyPosition" src="/static/images/location.png"></cover-image>
      </div>

      <!--弹框遮罩-->
      <cover-view class="mask-modal" v-if="isshow"></cover-view>
      <!--领取会员弹框-->
      <cover-view v-if="showmember" class="mask">
        <cover-view>
          <cover-image src="/static/images/modal.png" class="mask-img"/>
          <cover-image src="/static/images/close3.png" class="close" @click="closeModal"/>
          <cover-view class="text text1">
            <cover-view>优惠洗车</cover-view>
            <cover-view>到店洗车减免20%</cover-view>
          </cover-view>
          <cover-view class="text text2">
            <cover-view>服务卡券</cover-view>
            <cover-view>不定期赠送洗车的卡券</cover-view>
          </cover-view>
          <cover-view class="text text3">
            <cover-view>预约便捷</cover-view>
            <cover-view>预约洗车更方便快捷</cover-view>
          </cover-view>
        </cover-view>
        <!-- <cover-view @click="close">
              <cover-image src="/static/images/vip.png" class="vipbtn"/>
        </cover-view>-->
      </cover-view>

      <!--提示不在服务范围内-->
      <cover-view v-if="showShop" class="maskshop">
        <cover-view class="title">
          您的当前位置不在服务范围内，无法提供上门服务，是否到店服务？
          <!-- <cover-view style="margin-top:20rpx;">提供上门服务，是否到店服务？</cover-view> -->
        </cover-view>
        <cover-view class="flex-container bottombtn">
          <cover-view
            style="width:50%;text-align:center;height:110rpx;line-height:110rpx"
            @click="close"
          >取消</cover-view>
          <cover-view
            @click="confirm"
            style="width:50%;text-align:center;height:110rpx;line-height:110rpx;color:#ff6325;border-left:1rpx solid #f5f5f5"
          >确认</cover-view>
        </cover-view>
      </cover-view>
      <!--新用户礼券-->
      <cover-view v-if="isnew" class="newgroup">
        <cover-image src="/static/images/newbg.png" class="newpic"/>
        <cover-image src="/static/images/close3.png" class="close" @click="closeNewModal"/>
        <cover-view class="tuantile cover-text">送10元立减券</cover-view>
        <cover-view class="tuannew cover-text">绿妞新用户专享</cover-view>
        <cover-view class="tuanpick cover-text" @click="getNewConpon">立即领取</cover-view>
      </cover-view>
      <!--我要洗车-->
      <cover-view class="modal-xiche" v-if="isXiche">
        <cover-view class="line flex-container" @click="choseLocation">
          <cover-view class="flex-container">
            <cover-image src="/static/images/yellow.png" class="diandian"/>
            <cover-view class="location-self">{{nowPlace}}</cover-view>
          </cover-view>
          <!-- 更改地图位置，搜索位置 -->
          <cover-image src="/static/images/back.png" style="width:12rpx;height:22rpx;padding-left:10rpx;" />
        </cover-view>
        <cover-view class="line flex-container">
          <cover-view class="flex-container">
            <cover-image src="/static/images/yellow.png" class="diandian"/>
            <cover-view class="location-self">{{shopInfo.name||'周边无可服务商户!'}}</cover-view>
          </cover-view>
        </cover-view>
        <cover-view class="wash" @click="washCar">我要洗车</cover-view>
      </cover-view>
      <!--到店-->
      <cover-view class="modal-goshop" v-if="isGoshop">
        <cover-view @click="toShopdet(doorInfo.id)">
          <cover-image :src="doorInfo.Logo" class="showimg"/>
        </cover-view>
        <cover-view class="shopinfoflex" @click="toShopdet(doorInfo.id)">
          <cover-view class="shopname">{{doorInfo.name}}</cover-view>
          <cover-view class="addressflex">
            <cover-view class="flex-container">
              <cover-image
                v-for="item in doorInfo.ServiceScore"
                :key="item"
                src="/static/images/xing.png"
                class="xing"
              />
              <cover-image
                v-show="doorInfo.ServiceScore==0"
                v-for="item1 in 5"
                :key="item1"
                src="/static/images/gray1.png"
                class="xing"
              />
              <!-- <cover-image src="/static/images/xing.png" class="xing"/>
                <cover-image src="/static/images/xing.png" class="xing"/>
                <cover-image src="/static/images/xing.png" class="xing"/>
              <cover-image src="/static/images/xing.png" class="xing"/>-->
            </cover-view>
            <cover-view class="start">{{doorInfo.ServiceScore}}.0分</cover-view>
            <cover-view class="time">{{doorInfo.WorkTime}}</cover-view>
          </cover-view>
          <cover-view class="address">{{doorInfo.Address}}</cover-view>
        </cover-view>
        <cover-view class="rights" style="z-index:999" @click="getMap">
          <cover-image src="/static/images/rembg.png" class="big"/>
          <cover-image src="/static/images/wei.png" class="small"/>
          <cover-view
            class="span-info"
          >{{doorInfo.tofixedDistance>100?'100+':doorInfo.tofixedDistance}}km</cover-view>
        </cover-view>
      </cover-view>
    </div>
  </div>
</template>


<script>
import { get, myget, mypost, post, toLogin } from "../../utils";
import amapFile from "../../utils/amap-wx"; //高德地图API调用JS SDK
import { mapState, mapMutations } from "vuex"; //vuex辅助函数
import { transformFromGCJToWGS } from "@/utils/WSCoordinate"; //将gcj02坐标转wgs84
import QQMapWX from "@/utils/qqmap-wx-jssdk"; //腾讯地图，reverseGeocoder逆地址转码
import "../../css/common.css";
import "../../css/global.css";
export default {
  watch: {},
  computed: {
    ...mapState(["cityName", "nowPlace", "longitude", "latitude", "showMask"])
  },
  data() {
    return {
      qqmapsdk: null, //实例化地图sdk
      latitude: 0,
      longitude: 0,
      shopId: "", //展示的商户id
      name: "", //展示的商户名称
      shopNick: "", //导航的店铺名称
      shopLat: "",
      shopLng: "", //店铺的经纬度 用户导航
      address: "",
      userId: "",
      token: "",
      markerId: "1",
      points: "", //缩放视野以包含所有给定的坐标点  //bindmarkertap  点击标记点时触发，会返回marker的id  bindcallouttap 点击标记点对应的气泡时触发，会返回marker的id  bindcontroltap	点击控件时触发，会返回control的id
      shopArr: [], //商铺信息集合
      shopInfo: {}, //上门的店铺信息
      doorInfo: {}, //到店的商铺信息
      markers: [], //不显示
      controls: [
        {
          //控件不随着地图移动
          id: 0,
          iconPath: "/static/images/person.png",
          position: {
            left: "375rpx",
            top: 300,
            width: 30,
            height: 30
          },
          clickable: true
        }
      ],
      titlelist: [{ name: "上门" }, { name: "到店" }],
      active: "上门",
      isshow: false,
      showShop: false, //提示不在服务范围
      showmember: false, //是否是vip
      isXiche: true, //我要洗车
      isGoshop: false, //到店洗车最近的一家商铺
      isnew: false, //是否是新人
      dddd: false,
      cityName: "定位中..",

    };
  },

  onLoad() {
    this.getMyPosition(); //获取当前位置
    // 判断是否朋友分享
    if(this.$root.$mp.query.shareId){
      console.log('root',this.$root.$mp.query.shareId)
      this.$store.commit('update',{shareId:this.$root.$mp.query.shareId})
      
      console.log('root222',this.$store.state.shareId)
    }
    // 实例化API核心类
    this.qqmapsdk = new QQMapWX({
      key: "3P2BZ-6G4WD-CEX43-PIV5G-3VDYH-N5BGH" // 必填
    });
  },
  onShow() {
    wx.stopPullDownRefresh(); //关闭下拉刷新
    this.initData();
    if (this.userId && this.token) {
      let lat = this.$store.state.latitude
      let lon = this.$store.state.longitude

      // 初始展示上门还是到店
      // 判断城市名称是否被更改，获取城市中心位置
      if (this.cityName !== this.$store.state.cityName) {
        console.log("更改了城市名称", this.$store.state.cityName);
        // this.isXiche = false;
        this.cityName = this.$store.state.cityName;
        this.cityNameGetPosition();
      }else 
      // 更新了地理位置
      if(lat != this.latitude||lon!==this.longitude){
            this.latitude = lat;
            this.longitude = lon;
            this.getCityinfo()
      }

      // if (this.$root.$mp.query.name) {
      //   this.change(this.$root.$mp.query.name);
      // }
      //获取定位位置和城市
      // this.getMyPosition().then(res => {
      // });
      this.getCoupon()
      this.isNewVip(); //检查是否vip---应开启，还有新人
    } 
    // else {
    //   wx.navigateTo({ url: "/pages/login/main" });
    // }
  },
  methods: {
    ...mapMutations(["update"]),
    initData() {
      this.isshow = false;
      this.showmember = false;
      this.isnew = false;
      // this.isXiche = false;
      // this.isGoshop = false;
      this.showShop = false;
      this.userId = wx.getStorageSync("userId");
      this.token = wx.getStorageSync("token");
      // 获取定位,中文定位发生改变时
      // this.latitude = this.$store.state.latitude;
      // this.longitude = this.$store.state.longitude;

      // this.active='上门'
      // this.$store.commit("update", {
      //   latitude: 0,
      //   longitude: 0
      // });
    },
    // 获取定位
    getMyPosition() {
      return new Promise((resolved, rejected) => {
        wx.getLocation({
          type: "wgs84",
          success: data => {
            // console.log(data,"微信地图")
            this.latitude = data.latitude;
            this.longitude = data.longitude;
            this.$store.commit("update", {
              latitude: data.latitude,
              longitude: data.longitude
            });
            // 根据坐标获取城市信息
            this.getCityinfo().then(() => {
              resolved();
            });
          },
          fail() {
            //失败回调
            //如果用户拒绝授权,默认为北京
            this.cityName = "北京市";
            this.update({ cityName: "北京市" });
          }
        });
      });
    },
    //根据城市名称获取地图坐标
    cityNameGetPosition() {
      return new Promise((resolved, rejected) => {
        const that = this;
        this.qqmapsdk.geocoder({
          address: this.cityName,
          success(res) {
            console.log("根据地址转换坐标", res);
            const _res = res.result.location;
            that.latitude = _res.lat;
            that.longitude = _res.lng;
            that.update({
              latitude: _res.lat,
              longitude: _res.lng
            });
            that.getCityinfo();
          },
          fail(err) {
            console.log("根据地址转换坐标err", err);
          }
        });
      });
    },
    //根据经纬度获取城市名称nowPlace 反地理转码
    getCityinfo() {
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
            that.cityName = _res.address_component.city;
            that.update({
              cityName: _res.address_component.city,
              nowPlace:
                _res.formatted_addresses.recommend + " - " + _res.address
            });
            that.getShopData();
          },
          fail: function(res) {
            console.log(res);
          }
        });
      });
    },
    //获取店铺信息,店铺集合
    async getShopData() {
      // 获取完城市中文名后获取数据
      let url = "";
      // 初始化到店和上门选中的商户信息
      this.shopInfo = {};
      this.doorInfo = {};
      if (this.active === "上门") {
        url = "Shop/NearbyShop";
          this.isXiche = true;
      } else {
        url = "Shop/SearchShopList";
        this.isGoshop = true;
      }
      const res = await post(url, {
        Lat: this.latitude,
        Lng: this.longitude,
        Page: 1
      });
      if (res.code == 0) {
        this.shopArr = JSON.parse(JSON.stringify(res.data)); //所有店铺信息的集合
        //根据上门的数组长度判断是否在服务范围之内
        let arr = []; //保存markers数组
        for (let i = 0; i < res.data.length; i++) {
          //console.log(res.data[i])
          const _res = res.data[i];
          let marker = {
            iconPath: "/static/images/cart.png",
            id: _res.ShopId,
            Distance: _res.Distance,
            name: _res.ShopNick || "",
            Logo: _res.Logo,
            ServiceScore: _res.ServiceScore,
            BusinessHours: _res.BusinessHours,
            Address: _res.Address,
            WorkWeekSTR: _res.WorkWeekSTR,
            WorkTime: _res.WorkTime,
            WorkWeek:_res.WorkWeek,
            latitude: _res.Lat,
            longitude: _res.Lng,
            width: 24,
            height: 27.5
          };
          arr.push(marker);
        }
        this.markers = arr;
        function sortId(a, b) {
          return a.Distance - b.Distance;
        }
        console.log("距离排序", this.markers[0].Distance);
        this.markers.sort(sortId);
        console.log("距离排序2", this.markers[0].Distance);
        if (this.active === "上门") {
          this.shopInfo = this.markers[0];
        } else {
          // 到店
          // 判断如果大于100公里就不显示最近商家
          if (this.markers[0].Distance * 1 < 100) {
            this.doorInfo = this.markers[0];
            this.doorInfo.tofixedDistance = this.doorInfo.Distance.toFixed(2);
            
          } else {
            this.isGoshop = false;
            return false;
          }
        }
        // 最近的商户放大
          this.markers[0].iconPath = "/static/images/bigcar.png";
          this.markers[0].width = 48;
          this.markers[0].height = 55;
      }
    },
    //点击标记点
    markertap(e) {
      if (!e.mp.markerId) {
        return false;
      } else {
        const markerId = e.mp.markerId;
        //更改用户选中的标记样式
        this.markers.map((item, index) => {
          if (item.id === markerId) {
            item.iconPath = "/static/images/bigcar.png";
            item.width = 48;
            item.height = 55;
            console.log(item, "点击的商户信息");
            if (this.active === "上门") {
              this.shopInfo = item;
              this.isXiche = true;
            } else {
              // 到店
              this.doorInfo = item;
              this.doorInfo.Distance = (item.Distance * 1).toFixed(2);
              this.isGoshop = true;
              this.latitude = item.latitude * 1;
              this.longitude = item.longitude * 1;
            }
          } else {
            // 把不选中的图标变回原形
            item.iconPath = "/static/images/cart.png";
            item.width = 24;
            item.height = 27.5;
          }
        });
      }
    },
    //判断是否是新人  三天内
    async getCoupon() {
      console.log(this.$store.state.showMask, "this.$store.state.showMask");
      if (!this.$store.state.showMask) {
        const res = await post("/Coupon/IsNewUser", {
          UserId: this.userId,
          Token: this.token
        });
        console.log(res, "判断是否是新人");
        if (res.data.IsNewUser == 1 && res.data.IsNewCoupon == 1) {
          //if(res.data.IsNewUser==1){
          this.isshow = true;
          this.isnew = true;
          this.showmember = false;
        }
      }
    },
    //领取新人礼券
    async getNewConpon() {
      let res = await post("/Coupon/GetNewCoupon", {
        UserId: this.userId,
        Token: this.token
      });
      console.log(res, "领取新人礼券");
      if (res.code == 0) {
        wx.showToast({
          title: "领取成功",
          icon: "success",
          duration: 2000,
          complete: function() {}
        });
      }
      // this.isshow=false
      // this.isnew=false
      this.isnew = false;
      this.isshow = true;
      this.showmember = true;
      // this.isXiche=false
    },
    //验证是否vip
    async isNewVip() {
      console.log(this.$store.state.showMask, "this.$store.state.showMask");
      if (!this.$store.state.showMask) {
        let res = await post("/User/VerifyVIP", {
          UserId: this.userId,
          Token: this.token
        });
        //console.log(res,"领取vip  ")
        if (res.code == 0) {
          if (res.data.IsVip == 0) {
            //不是vip弹出领取vip点击开通vip
            if (this.isnew == true) {
              //如果弹出来新人框
              this.showmember = false;
            } else {
              this.isshow = true;
              this.showmember = true;
              this.isnew = false;
            }
          }
        }
        setTimeout(() => {
          this.update({ showMask: true });
        }, 3000);
      }
    },
    //关闭新人框
    closeNewModal() {
      (this.isnew = false), (this.isshow = true), (this.showmember = true);
      // this.isXiche=false
    },
    //关闭vip框
    closeModal() {
      (this.isnew = false), (this.isshow = false), (this.showmember = false);
      // this.isXiche=true
    },
    // 关闭上门周边无服务商家弹窗
    close() {
      this.isshow = false;
      this.showShop = false;
    },
    // 确认上门周边无服务商家弹窗，跳转到店
    confirm() {
      this.isshow = false;
      this.showShop = false;
      this.active = "到店";
      this.isGoshop = false;
      this.isXiche = false; //我要洗车
      this.getShopData();
    },
    // 改变上门还是到店状态
    change: function(name) {
      //console.log(name)
      this.active = name;
      this.isXiche = false;
      this.isGoshop = false;
      this.shopArr = [];
      this.getShopData();
    },
    // 跳转更改地图位置，搜索地图位置
    choseLocation() {
      wx.navigateTo({ url: "/pages/locationorder/main" });
    },
    washCar() {
      console.log('shopInfo',this.shopInfo);
      if (!this.shopInfo.id) {
        this.isshow = true;
        this.showShop = true;
        return false;
      }
      // wx.setStorageSync("shopInfo", this.shopInfo);
      // console.log(this.shopInfo, this.shopId, "跳转我要洗车");
      // wx.navigateTo({ url: "/pages/location/main?shopId=" + this.shopId });
      this.toShopdet(this.shopInfo.id,1)
      
    },
    goTo(e) {
      if (e == 1) {
        wx.navigateTo({ url: "/pages/city/main" });
      }
      if (e == 2) {
        wx.navigateTo({ url: "/pages/shoplist/main" });
      }
    },
    // 跳转商户详情，isVisit--1，上门；2，到店
    toShopdet(shopId,type) {
      wx.navigateTo({ url: `/pages/shopdetail/main?shopid=${shopId}&&isVisit=${type||2}`});
    },
    //打开导航app功能
    getMap() {
      wx.openLocation({
        latitude: this.shopLat * 1,
        longitude: this.shopLng * 1,
        address: this.address,
        name: this.shopNick,
        scale: 18
      });
    },
    // 地图视野是触发
    getCenterMap1() {},
    // 地图视野结束是触发
    getCenterMap() {
      if (this.active === "上门") {
      const that = this;
      console.log("自身位置坐标", this.longitude, this.latitude);
      const map = wx.createMapContext("map");
      map.getCenterLocation({
        success(res) {
          // 判断坐标一致，不用重复请求数据
          if (
            that.longitude === res.longitude &&
            that.latitude === res.latitude
          ) {
            return false;
          }
          //  const ress =  transformFromGCJToWGS(res.latitude,res.longitude)
          that.latitude = res.latitude;
          that.longitude = res.longitude;
          
            that.$store.commit("update", {
              latitude: res.latitude,
              longitude: res.longitude
            });
          console.log("中心位置坐标", that.longitude, that.latitude);
          // console.log('转换后的中心位置坐标',ress)
          that.getCityinfo();
        }
      });
      }
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
.cover-text {
  position: absolute;
  color: #fff;
}
.centermark {
  width: 80rpx;
  height: 92rpx;
}
</style>
