<template>
  <div class="wrapper">
    <div class="breadcrumb">
      <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/' }"><span class="rj-breadcrumb">如家公寓</span></el-breadcrumb-item>
            <el-breadcrumb-item :to="{ path: '/rent' }"><span class="rj-breadcrumb">{{this.$store.state.city}}租房</span></el-breadcrumb-item>
            <el-breadcrumb-item v-for="(item, index) of roomInfo.areaTag" :key="index"><span class="rj-breadcrumb">{{item}}租房</span></el-breadcrumb-item>
            <el-breadcrumb-item><span class="rj-breadcrumb">{{roomInfo.roomTitle}}</span></el-breadcrumb-item>
      </el-breadcrumb>
    </div>
    <div class="room-info">
      <!-- 房屋介绍轮播图 -->
      <div class="room-carousel">
        <thumb-carousel :roomPic="roomInfo.roomPic"></thumb-carousel>
      </div>
      <div class="rj-info">
        <div class="rj-title">
          <el-tag class="item-tag" effect="dark" type="info">签</el-tag>
          <h2>{{roomInfo.roomTitle}}</h2>
        </div>
        <div class="rj-price">
          ￥<span>{{roomInfo.rent}}</span>/月（季付价）
        </div>
        <p class="price-rule">
          服务费另计，详情见
          <el-button class="payment-method" type="text" @click="dialogPaymentMethod = true">付款方式</el-button>
          <el-dialog class="dialog-title" title="付款方式" :visible.sync="dialogPaymentMethod">
            <el-table class="dialog-item" :data="paymentData">
              <el-table-column property="paymentMethod" label="付款方式" width="150"></el-table-column>
              <el-table-column property="rent" label="租金" width="200"></el-table-column>
              <el-table-column property="serviceCharge" label="服务费/年"></el-table-column>
              <el-table-column property="cashPledge" label="押金"></el-table-column>
            </el-table>
            <div class="instructions">
              <p>说明</p>
              <p>1.目前仅支持网上签约</p>
              <p>2.押金会在您办理解约时一并返还</p>
              <p>3.解约时，未发生的租金和服务费按月结算</p>
            </div>
          </el-dialog>
        </p>
        <div class="rj-tags">
          <span class="tag">可预签7天</span>
          <span class="tag">深呼吸1.0</span>
          <span class="tag">拿铁4.0</span>
          <span class="tag">可长租一年</span>
          <span class="tag">非首次出租</span>
          <span class="tag">可长租一年</span>
          <span class="tag">非首次出租</span>
        </div>
        <el-divider class="rj-divider"></el-divider>
        <el-row class="home-info" :gutter="20">
          <el-col :span="6">
            <p class="home-info-title">建筑面积</p>
            <p class="home-info-data">{{roomInfo.buildArea}}㎡</p>
          </el-col>
          <el-col :span="6">
            <p class="home-info-title">朝向</p>
            <p class="home-info-data">朝{{roomInfo.toward}}</p>
            </el-col>
          <el-col :span="6">
            <p class="home-info-title">户型</p>
            <p class="home-info-data">{{roomInfo.doorModel}}</p>
          </el-col>
        </el-row>
        <el-divider class="rj-divider"></el-divider>
        <div class="rj-home-info">
          <span class="title">位置</span>
          <span class="info">{{roomInfo.location}}</span>
        </div>
        <el-divider class="rj-divider"></el-divider>
        <div class="rj-home-info">
          <div class="title">楼层</div>
          <div class="info">{{roomInfo.floor}}</div>
        </div>
        <el-divider class="rj-divider"></el-divider>
        <div class="rj-home-info">
          <div class="title">电梯</div>
          <div class="info">{{roomInfo.lift}}</div>
        </div>
        <el-divider class="rj-divider"></el-divider>
        <div class="rj-home-info">
          <div class="title">区域</div>
          <div class="info" v-for="(item,index) of roomInfo.areaTag" :key="index">
            <span class="area-item">{{item}}</span>
          </div>
        </div>
        <el-divider class="rj-divider"></el-divider>
        <div class="rj-home-info">
          <div class="title">年代</div>
          <div class="info">{{roomInfo.era}}年建成</div>
        </div>
        <div class="rj-button">
          <p class="order-table" @click="orderTableClick">预约看房</p>
          <p class="collection" @click="collectClick">{{roomInfo.collectState}}</p>
        </div>
        <el-dialog class="dialog-order-table" title="咨询约看" :visible.sync="dialogOrderTable" width="23%">
          <div class="order-table-info">
            <img :src="stewardInfo.portrait">
            <div class="dialog-content">
              <h3>{{stewardInfo.name}} <i class="iconfont">&#xe654;</i></h3>
              <span>提交约看后我会尽快联系您确认时间和地点</span>
            </div>
          </div>
          <div class="order-table-dialog">
            <el-date-picker format="yyyy 年 MM 月 dd 日" value-format="yyyy-MM-dd" v-model="orderTime" type="date" placeholder="选择日期"></el-date-picker>
          </div>
          <div class="order-table-dialog">
            <i class="iconfont">&#xe6f4;</i>
            <input type="text" disabled class="order-table-input" placeholder="请输入您的手机" v-model="orderPhone">
          </div>
          <div slot="footer" class="dialog-footer">
            <el-button @click="dialogOrderTable = false">取 消</el-button>
            <el-button type="primary" @click="orderRoom">确 定</el-button>
          </div>
        </el-dialog>
      </div>
    </div>
  </div>
</template>

<script>
import ThumbCarousel from './ThumbCarousel'
import axios from 'axios'

export default {
  name: 'Detail',
  props: {
    roomInfo: Object
  },
  data () {
    return {
      dialogPaymentMethod: false,
      dialogOrderTable: false,
      orderTime: '',
      orderPhone: '',
      stewardInfo: {
        name: '',
        portrait: ''
      },
      paymentData: [{
        paymentMethod: '月付',
        rent: '￥8600',
        serviceCharge: '￥8256',
        cashPledge: '￥8600'
      }, {
        paymentMethod: '季付',
        rent: '￥8600',
        serviceCharge: '￥8256',
        cashPledge: '￥8600'
      }, {
        paymentMethod: '半年付',
        rent: '￥8600',
        serviceCharge: '￥8256',
        cashPledge: '￥8600'
      }, {
        paymentMethod: '年付',
        rent: '￥8600',
        serviceCharge: '￥8256',
        cashPledge: '￥8600'
      }]
    }
  },
  components: {
    ThumbCarousel
  },
  methods: {
    orderTableClick () {
      if (this.$store.state.VERIFIED_ACCOUNT === '') {
        this.$message.error('请先登录！')
        return
      }
      axios({
        url: '/api/getStewardInfo',
        data: {
          managementArea: this.roomInfo.areaTag[0]
        },
        method: 'post',
        header: {
          'Content-Type': 'application/json'
        }
      })
        .then(res => {
          var data = res.data.data
          this.stewardInfo.name = data.steward.name
          this.stewardInfo.portrait = data.steward.portrait
          this.orderPhone = data.userPhone
          this.dialogOrderTable = true
        })
    },
    collectClick () {
      if (this.roomInfo.collectState === '已收藏') {
        this.$message({
          message: '已收藏该房间',
          type: 'warning'
        })
        return
      }
      if (this.$store.state.VERIFIED_ACCOUNT !== '') {
        axios({
          url: '/api/collectRoom',
          data: {
            roomId: this.roomInfo.id
          },
          method: 'post',
          header: {
            'Content-Type': 'application/json'
          }
        })
          .then(res => {
            if (res.data.status === 109) {
              this.$message.error('请先登录！')
              this.dialogOrderTable = false
            } else if (res.data.status === 114) {
              this.$message({
                message: res.data.message,
                type: 'warning'
              })
            } else {
              this.$message({
                message: '收藏成功',
                type: 'success'
              })
              this.roomInfo.collectState = '已收藏'
            }
          })
      } else {
        this.$message.error('请先登录！')
      }
    },
    orderRoom () {
      if (this.orderTime === '') {
        this.$message({
          message: '请选择约看时间',
          type: 'warning'
        })
        return
      }
      if (this.$store.state.VERIFIED_ACCOUNT !== '') {
        axios({
          url: '/api/orderRoom',
          data: {
            orderTime: this.orderTime,
            roomId: this.roomInfo.id
          },
          method: 'post',
          header: {
            'Content-Type': 'application/json'
          }
        })
          .then(res => {
            if (res.data.status === 109) {
              this.$message.error('请先登录！')
              this.dialogOrderTable = false
            } else if (res.data.status === 0) {
              this.$message({
                message: '约看成功，请等待管家联系您',
                type: 'success'
              })
              this.dialogOrderTable = false
            } else {
              this.$message({
                message: res.data.message,
                type: 'warning'
              })
              this.dialogOrderTable = false
            }
          })
      } else {
        this.$message.error('请先登录！')
        this.dialogOrderTable = false
      }
    },
    getCollectState () {
      axios({
        url: '/api/getCollectState',
        data: {
          roomId: this.roomInfo.id
        },
        method: 'post',
        header: {
          'Content-Type': 'application/json'
        }
      })
        .then(res => {
          this.roomInfo.collectState = res.data.data
        })
    }
  },
  computed: {
    VERIFIED_ACCOUNT () {
      return this.$store.state.VERIFIED_ACCOUNT
    }
  },
  watch: {
    VERIFIED_ACCOUNT () {
      this.getCollectState()
    }
  }
}
</script>

<style scoped>
  .wrapper{
    width: 1190px;
    margin: 0 auto;
    margin-top: 50px;
  }
    .rj-breadcrumb{
        font-size: 16px;
        color: #999;
        font-weight: 400;
    }
    .room-info{
      margin-top: 25px;
    }
      .room-carousel{
        width: 60%;
        display: inline-block;
      }
      .rj-info{
        display: inline-block;
        position: absolute;
        margin-left: 5px;
        width: 476px;
      }
        .rj-title h2{
          display: inline-block;
          margin: 10px auto 0;
        }
          .item-tag{
              padding: 5px 6px;
              height: 30px;
              line-height: 16px;
              font-size: 15px;
              background-color: #ff961e;
          }
        .rj-price{
          color: #ff961e;
          font-size: 30px;
          margin-top: 10px;
        }
        .price-rule{
          color: rgba(0,0,0,.6);
          font-size: 13px;
          margin: 5px 0 0 3px;
        }
          .payment-method{
            padding: 0;
            color: rgba(0,0,0,.6);
            text-decoration: underline;
          }
          .payment-method:hover{
            color: #ff961e;
          }
          .dialog-title >>> .el-dialog__title{
            font-weight: 600;
            font-size: 30px;
          }
          .dialog-item >>> th > .cell{
            font-size: 17px;
          }
          .instructions{
            color: rgba(0,0,0,.4);
          }
        .rj-tags{
          margin: 15px 0 15px;
        }
          .tag{
            background: rgba(0,0,0,.03);
            padding: 6px 10px;
            font-size: 14px;
            margin-right: 5px;
            margin-bottom: 8px;
            border-radius: 2px;
            color: rgba(0,0,0,.6);
            display: inline-block;
          }
        .rj-divider{
          margin: 0;
        }
        .home-info{
          padding: 5px;
        }
          .home-info-title{
            color: rgba(0,0,0,.4);
            margin: 6px 0 10px;
          }
          .home-info-data{
            color: rgba(0,0,0,.85);
            font-size: 20px;
            margin: 6px 0;
          }
        .rj-home-info{
          padding: 15px 5px;
          font-size: 18px;
        }
          .rj-home-info .title{
            color: rgba(0,0,0,.4);
            display: inline-block;
            margin-right: 2px;
            min-width: 62px;
          }
          .rj-home-info .info{
            display: inline-block;
          }
            .rj-home-info .info .area-item{
              text-decoration: underline;
              margin-right: 5px;
            }
        .rj-button{
          text-align: center;
          padding: 10px 20px;
        }
          .rj-button .order-table{
            padding: 15px;
            background-color: #ff961e;
            font-size: 20px;
            color: #fff;
            width: 350px;
            margin: 0 auto 5px;
            cursor: pointer;
          }
          .rj-button .collection{
            padding: 15px;
            background-color: #06a9b3;
            font-size: 20px;
            color: #fff;
            width: 350px;
            margin: 0 auto;
            cursor: pointer;
          }
        .order-table-info{
          margin-bottom: 20px;
        }
          .order-table-info img{
            width: 60px;
            display: inline-block;
          }
          .order-table-info .dialog-content{
            display: inline-block;
            transform: translateY(-15px);
          }
            .dialog-content .iconfont{
              font-size: 25px;
              color: #999;
            }
        .dialog-order-table >>> .el-dialog__title{
          font-size: 25px;
          font-weight: 500;
        }
        .dialog-order-table >>> .el-dialog__body{
          padding: 0 20px;
        }
          .order-table-dialog{
            position: relative;
            margin-bottom: 10px;
          }
            .order-table-dialog >>> .el-date-editor.el-input, .el-date-editor.el-input__inner{
              width: 100%;
            }
            .order-table-dialog .iconfont{
              position: absolute;
              top: 12px;
              left: 10px;
              color: #999;
            }
            .order-table-input{
              background-color: #FFF;
              background-image: none;
              border-radius: 4px;
              border: 1px solid #DCDFE6;
              box-sizing: border-box;
              color: #606266;
              display: inline-block;
              font-size: inherit;
              height: 40px;
              line-height: 40px;
              outline: 0;
              padding: 0 32px;
              transition: border-color .2s cubic-bezier(.645,.045,.355,1);
              width: 100%;
            }
</style>
