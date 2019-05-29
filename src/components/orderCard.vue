<template>
  <div v-show="showCoupon" >
      <!--遮罩层-->
    <div class="mask-modal" @click="closeMask"></div>
    <div class="paymask white" >
      <div class="paytile">
        <text>请选择服务卡券</text>
      </div>
      <div>
        <radio-group class="radio-group" @change="selectCoupon">
          <label class="flex-container couponItem" v-for="(item,index) in couponList" :key="index">
            {{item.Title}}
            <input type="radio" name="coupon" :value="item.CardTicketId" :checked="CardTicketId==item.CardTicketId" >
          </label>
          <label class="flex-container couponItem">
            不使用服务卡券
             <input type="radio" name="coupon"  :checked="CardTicketId==0" value="0" />
          </label>
        </radio-group>
      </div>
      <div class="paybtn btnbottom" @click="closeMask">完成</div>
    </div>
  </div>
</template>
<script>
import {post} from '@/utils/index'
export default {
    props:{
        showCoupon:{
          type:Boolean,
          default:false
        },
        couponList: [],
        // couponId:0,
        CardTicketId:{
          type:Number,
          default:0
        },
        CardTicketName:{
          type:String,
          default:''
        }
    },
    watch:{
     
    },
  data() {
    return {
    };
  },
  methods: {
    selectCoupon(e){
      const id = e.mp.detail.value*1
      // this.CardTicketId = id;
      // console.log(e.mp.detail,this.CardTicketId,this.CardTicketId)
      let cardName = ''
      for(let i=0;i<this.couponList.length;i++){
          const _res = this.couponList[i]
          if(_res.CardTicketId == id){
            // cardName = _res.Denomination*1
            cardName = _res.Title
          }
      }
        this.$emit('update:CardTicketId',id)
    // console.log(this.CardTicketId,'listid')
        this.$emit('update:CardTicketName',cardName)
        this.closeMask();
      
  },
  closeMask(){ 
    // console.log(this.CardTicketId,'list')
    // return false;
      this.$emit('update:showCoupon', false); //触发 input 事件，并传入新值
    //   this.showCoupon = !this.showCoupon
  }
  }
};
</script>
<style lang="scss" scoped>
  @import "../css/common.css";
.paymask {
  position: fixed;
  bottom: 0;
  left: 0;
  width: 100%;
  max-height:720rpx;
  overflow:auto;
  box-sizing: border-box;
  z-index: 60;
  padding-bottom: 60rpx;
}
.paytile {
  position: relative;
  padding: 30rpx;
  border-bottom: 1rpx solid #eee;
  text-align: center;
  font-size: 36rpx;
  font-weight: bold;
}
.couponItem {
  padding: 20rpx;
  border-bottom: 1rpx solid #ececec;
}
.btnbottom {
  margin-top: 60rpx;
}
</style>

