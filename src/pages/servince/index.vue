<template>
  <div class="flex-container bigcontainer">
    <!--左菜单-->
    <div class="ser-menu" style="position:fixed;width:32%;">
      <scroll-view scroll-y style="height:100vh;">
        <div
          style="padding:30rpx 0"
          v-for="(item,index) in servicelist"
          :key="index"
          :class="{active:scrollItem==item.Id}"
          @click="change(item.Id)"
          class="nemeitem"
          v-if="item.list.length>0"
        >
          <text class="title">{{item.TypeName}}</text>
        </div>
      </scroll-view>
    </div>
    <!--右列表-->
    <div v-if="servicelist.length<0">暂无满足条件的商品，敬请期待！</div>
    <scroll-view
      scroll-y
      :scroll-into-view="'item'+scrollItem"
      scroll-with-animation
      class="list"
      style="margin-left:32%;padding-bottom:90rpx;height:100vh;box-sizing:border-box;"
     >
      <div :id="'item'+servicelists.Id" v-for="(servicelists,pindex) in servicelist" :key="pindex" v-if="servicelists.list.length>0">
        <div
          class="listTitle"
          :class="servicelists.list.length<1?'listTitle_margin100':''"
        >{{servicelists.TypeName}}</div>
        <div class="flex-container intitem" v-for="(item,index) in servicelists.list" :key="index">
          <div class="flex-container" style="width:85%;" 
              @click="oneClick(pindex,index)">
            <img :src="item.PicNo" class="carpic">
            <div class="flex-container clomn iteminfo" style="width:70%">
              <p class="itemtitle">{{item.Name}}</p>
              <p class="progress">
                <text
                  style="margin-top:10rpx"
                  v-for="(itemtip,tindex) in item.KeywordName"
                  :key="tindex"
                >{{itemtip?itemtip:''}}</text>
              </p>
              <p class="sales">销量: {{item.SalesVolume}}</p>
              <p class="price">￥{{item.Price}}</p>
            </div>
          </div>
          <div>
            <input
              type="checkbox"
              class="checkbox-cart"
              :checked="item.isSelect" 
              @click="oneClick(pindex,index)"
            >
          </div>
        </div>
      </div>

    </scroll-view>
      <!--底部按钮-->
      <div class="btn">
        <div class="btn-close" style="background:#fff">
          <input
            type="checkbox"
            class="checkbox-cart"
            @click="selectProduct()"
            :checked="isSelectAll"
          >
          <text>全选</text>
        </div>
        <div class="btn-confirm" @click="submit">确定</div>
      </div>
  </div>
</template>

<script>
import { get, myget, mypost, post, toLogin } from "../../utils";
import "../../css/common.css";
import "../../css/global.css";
export default {
  onLoad() {
    this.latitude = this.$store.state.latitude;
    this.longitude = this.$store.state.longitude;
    this.shopId = this.$root.$mp.query.shopId;
    this.selectArr = []; //要提交的数组
    this.setBarTitle();
    console.log(this.shopId);
  },
  onShow() {
    this.Page = 1;
    this.servicelist = [];
    if (wx.getStorageSync("serItem").length > 0) {
      this.selectArr = wx.getStorageSync("serItem");
    }
    this.getMenulist();
  },
  data() {
    return {
      latitude: "",
      longitude: "",
      shopId: "",
      Page: 1, //当前页码
      PageSize: 30, //数量
      menulist: [],
      TypeId: "", //获取列表种类
      servicelist: [],
      selectArr: [], //要提交的数组
      active: "0",
      isSelectAll: false,
      scrollItem: "" //滑动中的item
    };
  },

  components: {},
  computed: {},
  methods: {
    setBarTitle() {
      wx.setNavigationBarTitle({
        title: "服务项目"
      });
    },
    async getMenulist() {
      // 洗车分类  0:默认全部 
      // 21:服务项目 22:服务套餐23:服务卡券 24:镀晶版本
      var result = await post("Server/GetCarWash", {
        BrandList: 0
      });
      //console.log(result)
      if (result.code == 0) {
          if(result.data[0]){
            this.scrollItem = result.data[0].Id
          }
        this.menulist = result.data;
        // this.TypeId=this.menulist[this.active].Id
        this.getProlist();
      }
    },
    //获取产品列表
    async getProlist() {
      let arr = [];
      // this.menulist.map(async (item, index) => {
        for(let i=0;i<this.menulist.length;i+=1){
          const item = this.menulist[i]
        // }
        let res = await post("Server/ChooseServiceProducts", {
          Page: this.Page,
          PageSize: this.PageSize,
          TypeId: item.Id,
          ShopId: this.shopId,
          // Lat:"1.000000",
          // Lng:"2.000000"
          Lat: this.latitude,
          Lng: this.longitude
        });
        if (res.code == 0) {
          arr.push({
            Id: item.Id,
            TypeName: item.TypeName,
            list: res.data
          });
        }
      }
          // if (index === this.menulist.length - 1) {
          // }
      this.servicelist = arr;
            this.checkedSelectArr();
    },
    // 查找是否满足已选择条件
    checkedSelectArr() {
      this.servicelist.map(servicelist => {
        servicelist.list.map(list=>{
        this.selectArr.map(arr => {
          if (list.Id === arr.Id) {
            list.isSelect = true;
          }
        })
        });
      });
    },
    change(id) {
      //点击菜单获取产品列表
      console.log("checkedSelectArr", id);
      this.scrollItem = id;
    },
    selectProduct: function() {
      //遍历servicelist，全部取反
      this.isSelectAll = !this.isSelectAll;
      //console.log(this.isSelectAll)
      for (var i = 0, len = this.servicelist.length; i < len; i++) {
        this.servicelist[i].isSelect = this.isSelectAll;
        console.log(this.servicelist[i].isSelect);
      }
    },
    oneClick(index,i) {
      const servicelist = this.servicelist[index].list[i]
      
          console.log("3",servicelist);
      if (!servicelist.isSelect) {
        this.selectArr.push(servicelist);
      } else {
        this.selectArr.map((arr, index) => {
          if (arr.Id === servicelist.Id) {
            this.selectArr.splice(index, 1);
          }
        });
      }
      servicelist.isSelect = !servicelist.isSelect;
      let isSelectAll = true; //判断是否全选
      for (var j = 0; j < this.servicelist.length; j++) {
        if (!servicelist.isSelect) {
          isSelectAll = false;
          this.isSelectAll = false;
          console.log("3");
          return false;
        }
      }
      this.isSelectAll = isSelectAll;
    },
    submit() {
      if (this.selectArr.length > 0) {
        wx.setStorageSync("serItem", this.selectArr);
        wx.navigateBack();
      } else {
        wx.showToast({
          title: "请选择要服务项目",
          icon: "none",
          duration: 2000
        });
        return false;
      }
    }
  },

  created() {
    // let app = getApp()
  }
  // onReachBottom(){
  //   if(this.isLoad){
  //     this.Page++;
  //   }else{
  //     wx.showToast({
  //         title: "没有更多商品啦。。。",
  //         icon: "none",
  //         duration: 2000
  //       });
  //   }
  // }
};
</script>

<style lang="scss" scoped>
@import "./style";
@import "../../css/common.css";
</style>
