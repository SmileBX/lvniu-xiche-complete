<template>
  <div class="proinfo flex-container pad">
    <div @click="goDetail">
      <img :src="data.img" class="orderimg">
    </div>
    <div class="inforight">
      <div class="infotitle" @click="goDetail">{{data.title}}</div>
      <div class="infospec">{{data.shopName}}</div>
      <div class="infoprice flex-container  flex-right">
        <p class="price">￥{{price}}</p>
        <p v-if="showButton">x{{data.num}}</p>
        <div v-else class="flex-center">
          <img src="/static/images/add5.png" @click="addNum" class="specpic">
          <input class="shopnum" v-model="data.num" type="number" @change="editNumInput">
          <img src="/static/images/shot5.png" @click="lessNum" class="specpic">
        </div>
      </div>
    </div>
  </div>
</template>
<script>
// 该组件使用的数据格式：
//       {
//           img:datas.ProductImg,
//           title: datas.ProductName,
//           sku: datas.SpecText,
//           skuSubmit: datas.SpecText,
//           shopName: datas.ShopName,
//            productId:datas.productId,
//           shopId:datas.ShopId,
//           price: datas.SalePrice,
//           num: datas.Number,
//           stock:datas.Stock,
   //        type:商品分类0--全部分类，21--商品，22--套餐，23--卡券
//         }
// showButton；为数量是否可编辑，true--显示数量；false--显示编辑按钮
export default {
  onLoad() {
    // this.setBarTitle();
  },
  props: {
    showButton:{
      type:Object,
      default(){
        return {}
      },
    },
      data:{
        type:Boolean,
        default:false
      },
      gotoDetail:{
        type:Boolean,
        default:false
      }
    },
  data() {
    return {
      isshow: false,
      showpay: false,
      showway: false,
      a: 4
    };
  },
  computed: {
    price() {
      let num = 0;
      num = (this.data.price * this.data.num).toFixed(2);
      return num;
    }
  },
  watch: {
  },
  components: {},
  methods: {
    setBarTitle() {
      wx.setNavigationBarTitle({
        title: ""
      });
    },
    editNumInput(e){
      this.data.num = e.mp.detail.value;
      this.$emit("editCar", this.data.id,this.data.num);
    },
    addNum() {
      this.$emit("editCar", this.data.id,this.data.num+1);
    },
    lessNum() {
        this.$emit("editCar", this.data.id,this.data.num-1);
    },
    
    // 跳转详情
    goDetail() {
      if(this.gotoDetail){
        
      //type商品分类0--全部分类，21--商品，22--套餐，23--卡券
        let type = this.data.type||this.data.BrandId||this.data.BrandId;
        let id = this.data.productId
        console.log(this.data,'type-id',id)
      var a = type*1;
      if (a === 21) {
        wx.navigateTo({ url: "/pages/serdetail/main?proid=" + id });
      }
      if (a === 22||a===23) {
        wx.navigateTo({ url: "/pages/coupondetail/main?id="+id });
      }
      }
      
    },
  },

  created() {
    // let app = getApp()
  }
};
</script>

<style lang="scss" scoped>
/*弹性布局*/
.flex-container {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.clomn {
  flex-direction: column;
}
.proinfo {
  padding: 20rpx 30rpx;
}
.inforight{
  width:100%;
}
.orderimg {
  width: 160rpx;
  height: 160rpx;
  vertical-align: middle;
  margin-right: 30rpx;
}
.price{
  width:100%;
}
.infospec {
  color: #999;
  font-size: 24rpx;
}
.infoprice {
  font-size: 36rpx;
  color: #ff6666;
  flex-flow:column nowrap;

}
.flex-right{
  // flex-flow:row-reverse nowrap;
  // justify-content:flex-end;
  align-items:flex-end;
}
.infoprice p + p {
  color: #1a1a1a;
  font-size: 28rpx;
}
.pad {
  padding: 20rpx 30rpx;
}
.infotitle,.infospec,.infoprice{
  width:100%;
}
.specpic {
  width: 48rpx;
  height: 44rpx;
  vertical-align: middle;
}
.shopnum {
  margin: 0 20rpx;
  color: #1a1a1a;
  font-size: 24rpx;
  width: 40rpx;
  text-align: center;
}
.flex-center {
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>