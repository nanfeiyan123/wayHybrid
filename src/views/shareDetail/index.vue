<template>
  <div class="rootEl">
    <AppHeaderBar></AppHeaderBar>
    <Loading v-if="isLoading"></Loading>
    <contDeleted v-else-if="isContDeleted"></contDeleted>
    <contError v-else-if="isContError"></contError>
    <contUnpass v-else-if="isContUnpass"></contUnpass>
    <netError v-else-if="isNetError"></netError>
    <!-- 有视频,且不是微信环境 st -->
    <div v-else style="margin-top: 1.706667rem;">
<!--      <div v-if="isVideo" class="video-wrapper">-->
<!--        <xg-player id="player"-->
<!--          :url="player.url"-->
<!--          :height="player.height"-->
<!--          :autoplay="player.autoplay"-->
<!--          :poster="player.poster"-->
<!--          ref="xgplayer">-->
<!--        </xg-player>-->
<!--      </div>-->
      <!-- 有视频，且是微信环境 -->
      <!-- <div v-if="isVideo && isWeichat()" class="video-wrapper-inwx" :style="{height: player.height}">
          <img :src="player.poster" alt="">
          <div class="play-button" @click="dnApp">
            <div class="img-wrap">
              <img src="../../assets/play-button.png" alt="">
            </div>
          </div>
      </div> -->
      <!-- 有视频 end -->
      <!-- 是状态 且有轮播图 st -->
      <div v-if="isStatus && info.images.length > 1" class="swiper-wrapper" >
        <swiper id='swiper' class='swiper' ref='swiper' :options='swiperOption'>
          <swiper-slide v-for='(item, index) in info.images' :key="index" style="width: 100%">
            <div class="slide-item" :style="{height: item.height > maxHeight ? maxHeight + 'rem' : item.height + 'rem', width: '100%'}">
              <img  v-lazy="item.src">
              <p class="pagination" v-if="info.images && info.images.length > 1">{{index +1}}/{{info.images.length}} </p>
            </div>
          </swiper-slide>
        </swiper>
      </div>
      <div v-if="isStatus && info.images.length == 1"  class="banner-wrapper" :style="{maxHeight: maxHeight + 'rem', width: '100%'}">
        <img v-lazy="info.images[0].src" alt="" :style="{height: info.images[0].height + 'rem'}">
      </div>
      <!-- 是状态 且有轮播图 end -->
      <div v-if="isArticle" class="banner-wrapper" :style="{maxHeight: maxHeight + 'rem'}">
        <img v-lazy="info.coverSrc" alt="" :style="{height: info.coverHeight}">
      </div>
      <div v-if="isArticle && info.title">
        <p class="article-title">{{info.title}}</p>
        <div class="bar-small"></div>
      </div>
      <!-- 用户头像和用户名发布时间 -->
      <publish-time :userinfo="info" @clickAvatar="clickAvatar" />
      <div class="content-wrapper" id="content-wrapper">
        <!-- 这是状态现实的内容 st-->
        <p v-if="isStatus">{{info.text}}</p>
        <!-- 这是状态现实的内容 end -->
        <!-- 这是文章现实的内容  st -->
        <!-- 如果是数组结构，启动懒加载写法 -->
        <div v-if="isArticle">
          <template v-if="info.contentArray && info.contentArray.length">
            <p v-for="(item,index) in info.contentArray" :key="index" >
              <span v-if="item.type =='text'" v-html="item.cont"></span>
              <img v-if="item.type == 'img'" v-lazy="item.src" width="100%" :height="(clientWidth - 40) / item.width * item.height">
            </p>
          </template>
          <!-- 如果不是数组结构，直接显示 -->
          <template v-else>
            <div v-html="info.content"></div>
          </template>
        <!-- 这是文章现实的内容 end -->
        </div>
      </div>
      <div class="content-info">
        <p class="read">{{info.cntReadStr}} {{isVideo ? '播放' : '阅读'}}</p>
        <div class="clearfix"></div>
        <p class="copyright" v-if="isCopyRightShow">
          本文版权归该作者所有，任何形式转载请联系作者。本文仅代表作者个人观点，不代表本平台的立场
        </p>
      </div>
      <user-info :userinfo="info" @clickCared="clickCared"></user-info>
      <comment-list  @showComment="showComment" @clickLike="clickLike" @clickAvatar="clickAvatar" :cntComment="cntComment" :commentList="commentList"/>
    </div>
  </div>
</template>

<script>
import AppHeaderBar from '@/components/AppHeaderBar'
import Loading from '@/components/Loading'
// 引入按钮水波纹指令
import waves from '@/directive/waves'
// 引入播放器
// import xgPlayer from '@/components/XgPlayer'
// 引入Api
import { addUserbrowseCntApi, commentListApi, contInfoApi } from '@/api/detail'
// 引入删除页，审核不通过页面，加载异常，网络异常
import contDeleted from '@/components/ErrorPage/contDeleted'
import contError from '@/components/ErrorPage/contError'
import contUnpass from '@/components/ErrorPage/contUnpass'
import netError from '@/components/ErrorPage/netError'
import UserInfo from '@/components/common/userinfo/UserInfo'
import PublishTime from '@/components/common/userinfo/PublishTime'
import CommentList from '@/components/common/userinfo/CommentList'
// 引入评论的封装
import {comment} from '@/utils/constant'
import blockImg from '../../assets/img_block.png'

import avatar from '@/components/AVATAR/'
export default {
  components: {
    // 'xg-player': xgPlayer,
    Loading,
    AppHeaderBar,
    contError,
    contUnpass,
    netError,
    contDeleted,
    avatar,
    UserInfo,
    PublishTime,
    CommentList
  },
  computed: {
    maxHeight () {
      return document.documentElement.clientHeight / this.rootfontSize - 1.706667// 最大高度屏幕高度  rem
    },
    clientWidth () {
      return parseFloat(document.documentElement.clientWidth)
    },
    rootfontSize () {
      return parseFloat(document.getElementsByTagName('html')[0].style.fontSize)
    }
  },
  data () {
    return {
      contId: '',
      isLoading: false,
      isContDeleted: false,
      isContError: false,
      isContUnpass: false,
      isNetError: false,
      player: { // http://jzvd.nathen.cn/8abcdf98ec6a418b945a70fe9dd6fc7f/5cb36416a23a4da8b15d3eaa5e19a1e6-5287d2089db37e62345123a1be272f8b.mp4  https://ww1.sinaimg.cn/large/663d3650gy1fq66vw50iwj20ff0aaaci.jpg
        url: '',
        autoplay: false,
        poster: '',
        height: ''
      },
      // swiper的配置
      swiperOption: {
        width: Math.trunc(document.documentElement.clientWidth),
        autoHeight: true, // 自动赋值高度
        loop: false, // 循环播放
        effect: 'fade', // 切换swiper-slide的效果
        fadeEffect: {
          crossFade: true
        }
      },
      isLogin: 0,
      info: {
        avatar: '',
        nickName: '',
        org: '',
        authTitle: '',
        isAuth: 0, // 认证标记
        isFollow: 0, // 是不是关注
        isOwner: 0, // 是不是作者
        publishDateStr: '',
        cntReadStr: '0',

        images: [], // swiper的列表
        text: '', // 状态的内容

        title: '',
        coverSrc: '', // 文章的封面图
        coverWidth: '100%', // 文章的封面图的宽高
        coverHeight: 'auto',
        content: '',
        contentArray: []
      },
      cared: false,
      isVideo: false,
      isStatus: false,
      isArticle: !false,
      isCopyRightShow: true,
      cntComment: 0,
      commentList: []
    }
  },
  mounted () {
    this.contId = this.$route.query.contId
    if (this.contId) {
      this.requestCommentList()
      this.requestDetail()
      this.handleAddUserbrowseCnt()
    } else {
      this.$vux.toast.text('参数缺失')
    }
  },
  methods: {
    clickCared () {
      this.dnApp()
    },
    showComment () {
      this.dnApp()
    },
    clickLike () {
      this.dnApp()
    },
    clickAvatar () {
      this.dnApp()
    },
    dnApp () {
      window.location.href = 'http://url.cn/553SnHV'
    },
    isWeichat () {
      var agent = navigator.userAgent.toLowerCase()
      if (agent.match(/MicroMessenger/i) == 'micromessenger') {
        return true
      } else {
        return false
      }
    },
    // 请求评论列表
    requestCommentList () {
      commentListApi({pageNum: 1, lastTimeMillis: new Date().getTime(), contId: this.contId, uid: ''})
        .then(res => {
          if (res.data.code === 2000) {
            this.cntComment = res.data.totalNum - 0
            this.commentList = comment.getCommentList(res.data.data, 3)
          } else {
            this.$vux.toast.hide()
          }
        })
        .catch(_ => {
          this.$vux.toast.hide()
        })
    },
    // 请求文章详情
    requestDetail () {
      this.isLoading = true
      let _this = this
      contInfoApi({contId: this.contId})
        .then(res => {
          this.isLoading = false
          if (res.data.code === 2000) {
            if (res.data.data.delStatus == 0) {
              if (res.data.data.contStatus === 3) {
                this.isContUnpass = true
              } else {
                this.isLogin = res.data.data.isLogin
                this.info.cntFansStr = res.data.data.cntFansStr
                this.info.cntReadStr = res.data.data.cntReadStr
                this.info.isOwner = res.data.data.isOwner
                // 对头像做onload
                this.info.avatar = ''
                var img = new Image()
                img.src = res.data.data.avatar
                img.onload = function () {
                  _this.info.avatar = res.data.data.avatar
                }
                this.info.nickName = res.data.data.nickName
                this.info.isFollow = res.data.data.isFollow
                this.info.isAuth = res.data.data.isAuth
                this.info.org = res.data.data.org
                this.info.authTitle = res.data.data.authTitle
                this.info.signature = res.data.data.signature
                this.info.publishDateStr = res.data.data.publishDateStr
                // 这是状态
                if (res.data.data.contType == 0) {
                  this.isCopyRightShow = false
                  this.isStatus = true
                  this.isArticle = false
                  this.isVideo = false
                  let content = {}
                  try {
                    content = JSON.parse(res.data.data.content)
                    // 这是有video的
                    if (res.data.data.timelineType == 1) {
                      this.isVideo = true
                      this.player.url = content.video.src
                      this.player.height = content.video.height / content.video.width * this.clientWidth / this.rootfontSize - 0.706667 + 'rem'
                      console.log(this.player)
                      try {
                        var coverSrcObj = JSON.parse(res.data.data.coverSrc)
                        this.player.poster = coverSrcObj.src
                      } catch (e) {
                        this.isContError = true
                      }
                      console.log(this.player.poster)
                      this.player.time = res.data.data.time
                    } else if (res.data.data.timelineType == 0) {
                      let array = []
                      console.log(content.images)
                      content.images.forEach((item, index) => {
                        let obj = {}
                        // 这是默认图片显示路径
                        obj.src = item.src
                        obj.width = '100%'
                        obj.height = item.height / item.width * this.clientWidth / this.rootfontSize // rem
                        console.log('height', item.height, 'clientWidth', this.clientWidth, 'wdith', item.width, 'obj.height', obj.height)
                        array.push(obj)
                      })
                      this.info.images = array
                      console.log(this.info.images)
                    }
                    this.info.text = content.text
                  } catch (error) {
                    console.log(error)
                    this.isContError = true
                  }
                // 这是文章
                } else if (res.data.data.contType === 1) {
                  this.isCopyRightShow = true
                  this.isArticle = true
                  this.isStatus = false
                  this.isVideo = false
                  this.info.title = res.data.data.title
                  try {
                    let coverObj = JSON.parse(res.data.data.coverSrc)
                    console.log(coverObj)
                    this.info.coverSrc = coverObj.src
                    this.info.coverWidth = '100%'
                    this.info.coverHeight = coverObj.height / coverObj.width * this.clientWidth / this.rootfontSize + 'rem'
                  } catch (error) {
                    this.isContError = true
                  }
                  this.info.content = res.data.data.content
                  this.$nextTick(() => {
                    let imgs = $('#content-wrapper img')
                    for (let i = 0; i < imgs.length; i++) {
                      let oSrc = imgs.eq(i).attr('src')
                      let oW = imgs.eq(i).attr('width')
                      let oH = imgs.eq(i).attr('height')
                      imgs.eq(i).attr('src', blockImg)
                      var iH = parseInt(oH / oW * (this.clientWidth - 40))
                      imgs.eq(i).css({width: '100%', height: iH + 'px'})
                      let Img = new Image()
                      Img.src = oSrc
                      Img.onload = () => {
                        imgs.eq(i).attr('src', oSrc)
                      }
                    }
                  })
                  // try {
                  //   let content = JSON.parse(res.data.data.content)
                  //   if (content && Array.isArray(content)) {
                  //     this.info.contentArray = content
                  //   } else {
                  //     console.log('content is not array')
                  //     this.info.content = res.data.data.content
                  //   }
                  // } catch (error) {
                  //   this.info.content = res.data.data.content
                  // }
                } else {
                  this.isContError = true
                }
              }
            } else {
              this.isContDeleted = true
            }
          } else {
            this.isContError = true
          }
        })
        .catch(e => {
          this.isLoading = false
          this.isNetError = true
        })
    },
    // 添加用户浏览数
    handleAddUserbrowseCnt () {
      addUserbrowseCntApi({contId: this.contId})
        .then(res => {
          console.log(res.data)
        })
    }
  },
  directives: {
    waves
  }
}
</script>

<style scoped>
.content-wrapper >>> p {
  white-space: pre-line;
  word-break: break-all;
  text-align: left;
  font-size: 16px;
  line-height: 26px;
  margin-top: 10px;
}
.content-wrapper >>> img {
  width: 100%;
  height: auto;
  display: block;
}
</style>

<style lang='less' scoped>
@import '../detail/index';
.loading {
  position: fixed;
  z-index: 10000;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  background: #fff;
}
.placeholder{
  width: 100%;
  display: block;
}
.video-wrapper-inwx{
  position: relative;
  width: 100%;
  background: #000;
  img{
    width: 100%;
    height: 100%;
    display: block;
  }
  .play-button{
    width: 100%;
    height: 100%;
    position: absolute;
    left: 0;
    top: 0;
    display: flex;
    justify-content: center;
    align-items: center;
    background: rgba(0, 0, 0, 0.3);
    .img-wrap{
      width: 1.2rem;
      height: 1.2rem;
      opacity: 0.7;
      img {
        width: 100%;
        height: 100%;
        display: block;
      }
    }
  }
}
</style>
