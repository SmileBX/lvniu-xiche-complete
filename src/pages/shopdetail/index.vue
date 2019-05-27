<template>
  <div>
    <shopChild
      :detailinfo="detailinfo"
      v-if="detailinfo.length>0"
      :shopLat="shopLat"
      :shopLng="shopLng"
    ></shopChild>
    <!--操作菜单-->
    <div class="shopmian">
      <div class="flex-container title">
        <p
          v-for="item in menulist"
          :key="item.name"
          :class="{active:active==item.name}"
          @click="change(item.name)"
        >{{item.name}}</p>
      </div>
      <!--服务显示-->
      <div class="sticky" v-if="sershow">
        <div class="caritem sershow">
          <!--洗车分类-->
          <text
            v-for="(item,mindex) in servincelist"
            :key="mindex"
            :class="{activecolor:activecolor==item.Id}"
            @click="showItemServe(item.Id)"
            v-if="item.list.length>0"
          >{{item.TypeName}}</text>
        </div>
        <div class="ships">以下为门店服务内容</div>
        <scroll-view scroll-y class="serlist" :scroll-into-view="'item'+activecolor" scroll-with-animation v-if="!isServiceOved">
          <div class="listBox" :id="'item'+servincelists.Id" v-for="(servincelists,servincelistIndex) in servincelist" :key="servincelistIndex">
            <div class="listBoxTitle" v-if="servincelists.list.length>0">{{servincelists.TypeName}}</div>
            <div class="seritem sershow flex-container" v-for="(item,index) in servincelists.list" :key="index">
              <div class="flex-container" style="width:80%" @click="goServiceProductsDetail(item.Id)">
                <img :src="item.PicNo" class="smallcar">
                <div class="flex-container col">
                  <p class="infotitle">{{item.Name}}</p>
                  <p class="infoitem">{{item.Synopsis}}</p>
                  <p class="serprice">
                    <text>￥{{item.Price}}</text>
                    <text>VIP:￥{{item.VipPrice}}</text>
                  </p>
                </div>
              </div>
              <div class="pay" @click="choseItem(item.Id)">立即购买</div>
            </div>
          </div>
        </scroll-view>
        
      <p
        class="ovedMsg"
        v-if="isServiceOved"
        style="text-align:center;padding:20rpx;font-size:26rpx;color:#666;"
      >没有数据了哦！</p>
      </div>
      <!--套餐显示-->
      <div class="meallist" v-if="dishshow">
        <div class="mealitem" v-for="item in meallist" :key="item.Id">
          <img src="/static/images/meal.png" class="mealbg">
          <div class="mealbg mealinfo">
            <div class="flex-col">
              <div class="weight sizebig">{{item.Name}}</div>
              <div class="explain">{{item.Synopsis}}</div>
            </div>
            <div class="linehr"></div>
            <div class="flex-container">
              <p class="mealprice">
                <text>特惠价:￥</text>
                <text>{{item.Price}}</text>
              </p>
              <p class="mealpay" @click="choseItem(item.Id)">立即购买</p>
            </div>
          </div>
        </div>
      </div>
      <!--评价显示-->
      <div v-if="pointshow" class="dish">
        <div class="head flex-container">
            <div class="titleline">
              <p class="shot"></p>
              <p class="main">商户服务评价</p>
              <p class="shot"></p>
            </div>
            <div class="flex-container">
              <p>
                <!-- <pointChildpic :commonlist="detailinfo.ServiceScore" ></pointChildpic> -->
                <img
                  src="/static/images/xing.png"
                  v-for="item in detailinfo[0].ServiceScore"
                  :key="item"
                  class="xing-point bigs"
                >
                <img src="/static/images/gray1.png" 
                v-for="rank in (5-detailinfo[0].ServiceScore)" :key="rank" class="xing-point">
                <!--<img src="/static/images/xing.png" class="xing-point bigs">
                <img src="/static/images/xing.png" class="xing-point bigs">
                <img src="/static/images/xing.png" class="xing-point bigs">
                <img src="/static/images/xing.png" class="xing-point bigs">-->
              </p>
              <text class="grad">{{detailinfo[0].ServiceScore}}.0分</text>
            </div>
        </div>
        <div class="flex-container pointmenu sticky">
          <p
            v-for="(item,index) in pointlist"
            :key="index"
            :class="{active3:first==index}"
            @click="changeComment(index)"
          >{{item.name}}</p>
          <!-- ({{PageCount}}) 数量-->
        </div>
        <div class="pointsheet">
          <pointChildpic :commonlist="commonlist" :showPic="true" v-if="commonlist.length>0"></pointChildpic>
        </div>
      </div>
      <p
        class="ovedMsg"
        v-if="isOved"
        style="text-align:center;padding:20rpx;font-size:26rpx;color:#666;"
      >没有数据了哦！</p>
    </div>
  </div>
</template>

<script>
import { get, myget, mypost, post, toLogin } from "../../utils";
import shopChild from "@/components/shopChild"; //公用组件
import pointChildpic from "@/components/pointChildpic"; //公用评论带图片组件
import "../../css/common.css";
import "../../css/global.css";

export default {
  data() {
    return {
      shopLat: "", //商铺经纬度 用于服务详情导航
      shopLng: "",
      shopid: "",
      lat: "",
      lng: "",
      detailinfo: [],
      typeid: 0,
      active: "服务",
      activecolor: '',
      servincelist: [],
      meallist: [],
      first: 0,
      Token: "",
      UserId: "",
      Page: "1",
      PageSize: 8,
      PageCount: "",
      allPage: "", //页数
      ProductId: "",
      isLoad: false,
      isLoadCom: false,
      barlist: [], //洗车分类
      menulist: [
        { id: 1, name: "服务" },
        { id: 2, name: "套餐" },
        { id: 3, name: "评价" }
      ],
      pointlist: [
        { id: 0, name: "全部" },
        { id: 1, name: "晒图" },
        { id: 2, name: "低分" },
        { id: 3, name: "最新" }
      ],
      commonlist: [],
      sershow: true,
      dishshow: false,
      pointshow: false,
      isOved: false,
      isServiceOved:false,
      isVisit:0, //是否到店还是上门2--到店;1--上门0--展示全部
    };
  },

  components: {
    shopChild,
    pointChildpic
  },
  onLoad() {
    this.shopid = this.$root.$mp.query.shopid;
    // isVisit 2--到店;1--上门;0--展示全部
    if(this.$root.$mp.query.isVisit){
      this.isVisit = this.$root.$mp.query.isVisit||0;
    }else{
      this.isVisit =0
    }
    console.log(this.shopid,this.isVisit ,"详情页接收");
  },
  async onShow() {
    this.Token = wx.getStorageSync("token");
    this.UserId = wx.getStorageSync("userId");
    await this.getMyPosition()
    // this.lat = this.$store.state.latitude;
    // this.lng = this.$store.state.longitude;
    this.servincelist = [];
    this.meallist = [];
    this.commonlist = [];
    this.detailinfo = [];
    this.activecolor='';
    this.sershow = true;
    this.dishshow = false;
    this.pointshow = false;
    this.isOved = false;
    this.isServiceOved = false;
    this.Page = 1;
    this.typeid = 0;
    this.active = "服务";
    this.setBarTitle();
    this.getShopDetail();
    console.log(this.typeid, "page服务类型id");
  },
  methods: {
    setBarTitle() {
      let title = "商户详情"
      if(this.isVisit==1){
        title = '商户上门'
      }
      if(this.isVisit==2){
        title = '商户到店'
      }
      // 其他位置进来，重新设置坐标
      if(this.isVisit==0){
        this.setMapMask()
      }
      wx.setNavigationBarTitle({
        title
      });
    },
    
    // 获取定位
    getMyPosition() {
      return new Promise((resolved, rejected) => {
        wx.getLocation({
          type: "gcj02",
          success: data => {
            // console.log(data,"微信地图")
            this.lat = data.latitude;
            this.lng = data.longitude;
            // 根据坐标获取城市信息
              resolved();
          },
          fail(){
            wx.showToast({
              title:'获取定位失败！请重新尝试',
              icon:'none'
            })
          }
        });
      });
    },
    async getShopDetail() {
      //商户详情
      var res = await post("Shop/GetMerchantDetail", {
        ShopId: this.shopid,
        Lat: this.lat,
        Lng: this.lng
      });
      this.getBarlist();
      if (res.code == 0) {
        this.$set(
          res.data[0],
          "Distance",
          parseFloat(res.data[0].Distance).toFixed(2)
        );
        this.detailinfo = res.data;
        this.shopLat = res.data[0].Lat * 1;
        this.shopLng = res.data[0].Lng * 1;
      }
      //console.log(this.detailinfo,"商家详情")
    },
      //服务项目分类菜单
    async getBarlist() {
      var res = await post("Server/GetServerType", {});
      if (res.code == 0) {
        this.servincelist = [];
        this.barlist =[]
        this.barlist = res.data;
        // this.typeid = res.data[0].Id
        console.log(this.barlist, "barlist服务分类");
      }
      this.showItem();
    },
      //服务项目列表
    async showItem() {
        let isOvedstatus = true;
        for(let i=0;i<this.barlist.length;i+=1){
          const barlist = this.barlist[i]
          const params={
            UserId:this.UserId,
            Token:this.Token,
            Page: this.Page,
            PageSize: this.PageSize,
            TypeId: barlist.Id,
            //TypeId:33,
            ShopId: this.shopid,
            lat:this.lat,
            lng:this.lng 
          }
            barlist.list=[]
            const res = await post('Server/ServiceProducts',params)
            if (res.code == 0) {
              res.data.map(item=>{
                console.log(this.isVisit,'this.isVisit')
                // isVisit 2--到店;1--上门;0--展示全部
                // 上门
                if(this.isVisit==1&&item.ServiceMode==1){
                  barlist.list.push(item)
                }else
                // 到店
                if(this.isVisit==2&&item.ServiceMode!=1){
                  barlist.list.push(item)
                }else if(this.isVisit==0){ //全部
                  barlist.list.push(item)
                }
              })
              // 默认active分类
                  if(!this.activecolor&&res.data.length>0){
                    this.activecolor = barlist.Id
                  }
                // 判断是否有展示列表
                console.log('123',barlist)
                  if(barlist.list.length>0){
                    isOvedstatus = false;
                  }

                  this.servincelist.push(barlist)
              console.log(this.servincelist, "服务产品列表");
            }
        }
                this.isServiceOved = isOvedstatus
    },
      //服务分类click
    showItemServe(id) {
      this.activecolor = id;
    },
    changeComment(e) {
      //获取不同类型评价列表
      this.first = e;
      this.CommentType = e;
      this.commonlist = [];
      this.Page = 1;
      this.isOved=false;
      this.showComment();
    },
    //获取评价列表
    async showComment() {
      if(this.isOved){
        return false;
      }
      var res = await post("/Server/ServiceCommentList", {
        Page: this.Page,
        PageSize: this.PageSize,
        // ProductId:this.ProductId,
        ShopId: this.shopid,
        CommentType: this.first
      });
      console.log(res, "评价列表");
      if (res.code == 0) {
        // this.PageCount = res.count;
        // if (parseInt(this.PageCount) % this.PageSize === 0) {
        //   this.allPage = this.PageCount / this.PageSize;
        // } else {
        //   this.allPage = parseInt(this.PageCount / this.PageSize) + 1;
        // }
        if(res.data.length!==this.PageSize){
          this.isOved=true;
        }
        if(this.Page==1){
          this.commonlist = [];
        }
        this.commonlist = this.commonlist.concat(res.data);
        for (let i of this.commonlist) {
          this.$set(i, "AddTime", i.AddTime.split("T").join("  "));
        }
        // console.log(this.commonlist);
        // if (this.allPage > this.Page) {
        //   this.isLoadCom = true;
        // } else {
        //   this.isLoadCom = false;
        // }
      }
    },
      //获取商户套餐
    async getServiceMeal() {
      if(this.isOved){
        return false;
      }
      const res = await post("/Server/ServiceMealProducts", {
        //获取套餐
        Page: this.Page,
        PageSize: this.PageSize,
        ShopId: this.shopid
      });
      if (res.code == 0) {
        if(res.data.length!==this.PageSize){
          this.isOved=true;
        }
        if(this.Page==1){
          this.meallist =[];
        }
          this.meallist = this.meallist.concat(res.data);
        console.log(res.data, "套餐列表");
      }
    },
    async change(e) {
      console.log(e);
      this.active = e;
      this.Page = 1;
      this.isOved=false;
      if (e == "服务") {
        (this.sershow = true),
          (this.dishshow = false),
          (this.pointshow = false)
        // this.getBarlist();
      } else if (e == "套餐") {
        (this.dishshow = true),
          (this.sershow = false),
          (this.pointshow = false);
        this.getServiceMeal();
      } else {
        (this.pointshow = true),
          (this.sershow = false),
          (this.dishshow = false);
        this.showComment();
      }
    },

    async goServiceProductsDetail(e) {
      wx.navigateTo({
        url:
          "/pages/serdetail/main?proid=" +
          e +
          "&shopLat=" +
          this.shopLat +
          "&shopLng=" +
          this.shopLng
      }); //获取商户服务产品详情
    },
    choseItem(e) {
      //console.log(e)
      this.$store.commit("setVisitConfirmOrder", {
        ProductId: e
      });
      wx.navigateTo({ url: "/pages/visitconfirmorder/main" });
    },
    // 重新设置坐标
    setMapMask(){
      // 获取定位
      wx.getLocation({
        type: "gcj02",
        success: res => {
          this.$store.commit('update',{
            latitude:res.latitude,
            longitude:res.longitude
          })
        }
          });
    }
  },

  created() {
    // let app = getApp()
  },
  onReachBottom() {
    if(!this.isOved){
      this.Page+=1;
      if(this.active==='服务'){
        // this.showItem();
      }else if(this.active==='套餐'){
        this.getServiceMeal();
      }else{
        this.showComment();
      }
    }
  }
};
</script>

<style lang="scss" scoped>
@import "./style";
@import "../../css/common.css";
</style>
