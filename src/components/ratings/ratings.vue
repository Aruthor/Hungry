<template>
<!-- 评论内容区域组件 -->
  <div class="ratings" ref="ratings">
    <div class="ratings-content">
      <div class="overview">
        <div class="overview-left">
          <h1 class="score">{{seller.score}}</h1>
          <div class="title">综合评分</div>
          <div class="rank">高于周边商家{{seller.rankRate}}%</div>
        </div>
        <div class="overview-right">
          <div class="score-wrapper">
            <span class="title">服务态度</span>
            <star :size="36" :score="seller.serviceScore" class="star"></star>
            <span class="score">{{seller.serviceScore}}</span>
          </div>
          <div class="score-wrapper">
            <span class="title">商品评分</span>
            <star :size="36" :score="seller.foodScore"></star>
            <span class="score">{{seller.foodScore}}</span>
          </div>
          <div class="delivery-wrapper" >
            <span class="title">送达时间</span>
            <span class="delivery">{{seller.deliveryTime}}分钟</span>
          </div>
        </div>
      </div>
      <!-- 商品评价组件 -->
      <split></split>
      <ratingselect :select-type="selectType" :only-content="onlyContent" :desc="desc"
                    :ratings="ratings"></ratingselect>
      <!-- 评价内容 -->
      <div class="rating-wrapper" >
        <ul>
          <li v-for="(rating, index) in ratings" :key="index" class="rating-item border-1px" v-show="needShow(rating.rateType,rating.text)">
            <div class="avatar">
              <img :src="rating.avatar" width="28" height="28"/>
            </div>
            <div class="content">
              <h1 class="name">{{rating.username}}</h1>
              <div class="star-wrapper">
                <star :size="24" :score="rating.score" class="star"></star>
                <span class="delivery" v-show="rating.deliveryTime">{{rating.deliveryTime}}</span>
              </div>
              <p class="text">{{rating.text}}</p>
              <div class="recommend" v-show="rating.recommend && rating.recommend.length">
                <span class="icon-thumb_up"></span>
                <span class="item" v-for="(item, index) in rating.recommend" :key="index">{{item}}</span>
              </div>
              <div class="time">
                {{rating.rateTime | formatDate}}
              </div>
            </div>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script type="text/ecmascript-6">
import star from '../star/star'
import ratingselect from '../ratingselect/ratingselect'
import split from '../split/split'
import BScroll from 'better-scroll'
import Bus from '../../common/js/EventBus.js'
import {formatDate} from '../../common/js/date.js'

const ALL = 2
const ERR_OK = 0
export default {
  props: {
    seller: {
      type: Object
    }
  },
  data () {
    return {
      ratings: [],
      selectType: ALL,
      onlyContent: true,
      desc: {
        all: '全部',
        positive: '推荐',
        negative: '吐槽'
      }
    }
  },
  methods: {
      // select(type) {
      //   this.selectType = type;
      //   this.$nextTick(() => {
      //     this.scroll.refresh();
      //   });
      // },
      // onlyContent2(onlyContent) {
      //   this.onlyContent = onlyContent;
      //   this.$nextTick(() => {
      //     this.scroll.refresh();
      //   });
      // }
     needShow (type, text) { // 评价内容显示
        if (this.onlyContent && !text) {
            return false
        }
        if (this.selectType === ALL) {
            return true
        } else {
            return type === this.selectType
        }
    }
  },
  created () {
    this.$http.get('/api/ratings').then((res) => {
      res = res.body
        if (res.errno === ERR_OK) {
          this.ratings = res.data
          this.$nextTick(() => {
            this.scroll = new BScroll(this.$refs.ratings, {
              click: true
            })
          })
        }
    })
    Bus.$on('ratingtype.select', selectType => {
        this.selectType = selectType
        this.$nextTick(() => {
            this.scroll.refresh()
        })
    })
    Bus.$on('content.toggle', onlyContent => {
        this.onlyContent = onlyContent
        this.$nextTick(() => {
            this.scroll.refresh()
        })
    })
  },
  components: {
    star,
    ratingselect,
    split
  },
  filters: {
    formatDate (time) {
      let date = new Date(parseInt(time))
      return formatDate(date, 'yyyy-MM-dd hh:mm')
    }
  }
}
</script>

<style lang="scss" rel="stylesheet/scss" scoped>
@import '../../common/scss/_mixins.scss';
.ratings{
  position: absolute;
  top: 174px;
  bottom: 0;
  width: 100%;
  left: 0;
  overflow: hidden;
  .ratings-content{
    .overview{
      display: flex;
      padding: 18px 0;
      .overview-left{
        flex: 0 0 137px;
        width: 137px;
        border-right: 1px solid rgba(7,17, 27, 0.1);
        text-align: center;
        padding: 6px 0;
        /* 兼容ipone5 @media only screen and (max-width: 320px)*/
        @media only screen and (max-width: 320px) {
          flex: 0 0 120px;
          width: 120px;
        }
        .score{
          line-height: 28px;
          font-size: 24px;
          color:rgb(255, 153,0);
          margin-bottom: 6px;
        }
        .title{
          margin-bottom: 8px;
          line-height: 12px;
          font-size: 12px;
          color: rgb(7,17,27);
          font-weight: 700;
        }
        .rank{
          line-height: 10px;
          font-size: 10px;
          color: rgb(147, 153, 159)
        }
      }
      .overview-right{
        flex: 1;
        padding: 6px 0 6px 24px;
        @media only screen and (max-width: 320px) {
          padding-left: 6px;
        }
        .score-wrapper{
          margin-bottom: 8px;
          font-size: 0;
          .title{
            font-size: 12px;
            line-height: 18px;
            color: rgb(7, 17, 27);
            font-weight: 700;
          }
          .star{
            display: inline-block;
            margin:0 12px;
            vertical-align: top;
          }
          .score{
            display: inline-block;
            line-height: 18px;
            font-size: 12px;
            vertical-align: top;
            color: rgb(255, 153, 0);
          }
        }
        .delivery-wrapper{
          font-size: 0;
          .title{
            line-height: 18px;
            font-size: 12px;
            color: rgb(7,17,27);
            font-weight: 700;
          }
          .delivery{
            margin-left: 12px;
            font-size: 12px;
            color: rgb(147, 153, 159)
          }
        }
      }
    }
    .rating-wrapper{
      padding: 0 18px;
      .rating-item{
        display: flex;
        padding: 18px 0;
        @include border-1px(rgba(7,17,27,0.1));
        .avatar{
          flex: 0 0 28px;
          width: 28px;
          margin-right: 12px;
          img{
            border-radius: 50%;
          }
        }
        .content{
          flex: 1;
          position: relative;
          .name{
            margin-bottom: 4px;
            color: rgb(7, 17, 27);
            font-size: 10px;
            line-height: 12px;
          }
          .star-wrapper{
            margin-bottom: 6px;
            font-size: 0;
            .star{
              display: inline-block;
              vertical-align: top;
              margin-right: 6px;
            }
            .delivery{
              display: inline-block;
              vertical-align: top;
              line-height: 12px;
              font-size: 10px;
              color: rgb(147, 153, 159);
            }
          }
          .text{
            line-height: 18px;
            color: rgb(7, 17, 27);
            margin-bottom: 8px;
            font-size:12px;
          }
          .recommend{
            line-height: 16px;
            font-size: 0;
            .icon-thump_up,.item{
              display: inline-block;
              margin:0 8px 4px 0;
              font-size: 9px;
            }
            .icon-thumb_up{
              color: rgb(0, 160, 220);
            }
            .item{
              padding: 0 6px;
              border: 1px solid rgba(7, 17, 27,.1);
              border-radius: 5px;
              color: rgb(147, 153, 159);
              background: #fff
            }
          }
          .time{
            position: absolute;
            top: 10px;
            right: 5px;
            font-size: 10px;
            line-height: 12px;
            color: rgb(147, 153, 159);
          }
        }
      }
    }
  }
}
</style>
