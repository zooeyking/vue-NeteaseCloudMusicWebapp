<template>
  <div class="rank-songs" ref="ranks">
      <my-header  class="my-header" :style="{'background-color':'rgba(0,0,0,'+ bgOpacity +')'}">
        <div class="h4-header">
          <h4 :class="{'h-title':true,'h-title-move':isTextOver(songsTitle)}">{{songsTitle}}</h4>
        </div>
        <div class="h-search" @click="showsearchList=true"><i class="iconfont icon-search02"></i></div>
        <div class="h-more"><i class="iconfont icon-more3"></i></div>
      </my-header>
      <iscroll-view class="scroll-content" style="touch-action: none;" 
      :options="options"
      @scroll="sMove"
      >
        <div>
          <div class="songs-hh">
          <img :src="songsDetail.coverImgUrl" alt="" class="songs-bg">
          <div class="bg-cover"></div>
          <img :src="songsDetail.coverImgUrl" alt="" class="songs-bg front">
          <div class="hh-detail"  :style="{opacity:HOpacity}">
            <div class="h-img-box">
              <div class="h-img-box-con">
                <img :src="songsDetail.coverImgUrl" alt="" v-pic v-if="songsDetail.coverImgUrl">
                <span class="description"><i class="iconfont icon-icon_warn_line_circle warns"></i></span>
                <span class="play-count">
                  <i class="iconfont icon-kefu"></i>
                  <span class="p-c-text">{{covertWan(songsDetail.playCount)}}</span>
                </span>
              </div>
            </div>
            <div class="list-name">
              <h6 class="h6-text">{{songsDetail.name}}</h6>
              <div class="update-time">最近更新:{{getUpdateText(songsDetail.updateTime,'date')}}</div>
              <div class="l-n-d">
                <span class="l-n-d-img">
                  <span class="vip-type" v-if="songsDetail.creator.vipType"><i class="iconfont icon-jinlingyingcaiwangtubiao49"></i></span>
                  <img :src="songsDetail.creator.avatarUrl" alt="" v-pic>
                  </span><span class="l-n-d-text">
                {{songsDetail.creator.nickname}}</span><i class="iconfont icon-more"></i>
              </div>
            </div>
          </div>
          <div class="hh-control"  :style="{opacity:HOpacity}">
            <div class="h-c-item">
              <i class="iconfont icon-add"></i>
              <span class="h-c-i-text">{{covertWan(songsDetail.subscribedCount)}}</span>
            </div>
            <div class="h-c-item">
              <i class="iconfont icon-pinglun1"></i>
              <span class="h-c-i-text">{{covertWan(songsDetail.commentCount)}}</span>
            </div>
            <div class="h-c-item">
              <i class="iconfont icon-share"></i>
              <span class="h-c-i-text">{{covertWan(songsDetail.shareCount)}}</span>
            </div>
            <div class="h-c-item">
              <i class="iconfont icon-xiazai1"></i>
              <span class="h-c-i-text">下载</span>
            </div>
          </div>
        </div>
        <div class="songs-list">
          <div class="songs-list-header">
            <span class="icon-box"><i class="iconfont icon-bofang"></i></span>
            <span class="icon-text">播放全部<span class="counts">(共{{songsDetail.trackCount}}首)</span></span> 
            <div class="multi-c">
              <i class="iconfont icon-gengduo"></i><span>多选</span>
            </div>
          </div>
          <div class="s-l-con">
              <ul class="s-l-ul">
                <li class="s-l-li" v-for="(item,index) in songsDetail.tracks" 
                @touchstart="playStart"
                @touchend="playEnd($event,item.id)"
                :key="index">
                <div class="num" v-if="item.id == songId"><i class="iconfont icon-volume-"></i></div>
                <div class="num" v-else>{{index+1}}</div>
                  <div class="text-right">
                    <div class="s-l-li-text">
                      <div class="song-name">{{item.name}}
                        <!--<span class="alias" v-if="item.alia.length">({{item.alia[0]}})</span>-->
                        <span class="mv" v-if="item.mvid"><i class="iconfont icon-MV"></i></span>
                      </div>
                      <div class="singer">{{getArtistsName(item.artists)}} - {{item.album.name}}</div>
                    </div>
                    <div class="more"@click="goLayer(item.id,item.name)" @touchend.stop.prevent>
                      <i class="iconfont icon-more3"></i>
                    </div>
                  </div>
                </li>
              </ul>
          </div>
        </div>
        </div>
      </iscroll-view>
        <rank-search 
        v-on:startSearch="searchList" 
        v-if="showsearchList"
        :searchResult="searchResult"
        ></rank-search>
  </div>
</template>
<script>
import myHeader from '@/components/header'
import {topListSongs} from '@/service/getData';
import {mapState,mapMutations} from 'vuex';
import {scrollMixin} from '@/common/js/mixin';
import { Indicator } from 'mint-ui';
import rankSearch from '@/components/rank-search';

export default {
  name:'ranksongs',
  data(){
    return{
      songsDetail:'',
      bgOpacity:0,
      HOpacity:1,
      songsTitle:'歌单',
      //iscroll配置
       options:{
          mouseWheel: true,
          click: true,
          preventDefault: false,
          tap: false,
          bounce: true,
          disableTouch: false,
          disableMouse:false,
          scrollbars:true,
          fadeScrollbars:true,
          interactiveScrollbars:true,
          shrinkScrollbars:'clip',
          probeType: 2
       },
       searchResult:[],
       showsearchList:false,
    }
    
  },
  mixins:[scrollMixin],
  created(){
    Indicator.open();
    topListSongs({
      idx:this.idx
    }).then((res)=>{
      console.log(res,'排行歌曲');
      if(res.data.code == 200){
          this.songsDetail = res.data.result;
          setTimeout(()=>{
            Indicator.close();
          },300)
      }
      
    }).catch((err)=>{
       console.log(err);
    })
  },
  components:{
    myHeader,
    rankSearch
  },
  computed:{
    ...mapState(['idx']),
  },
  methods:{
    ...mapMutations(['updateshowSongMore']),
    covertWan(count){
      return count < 10000 ? count : Math.floor(count/10000)+'万';
    },
    goLayer(id,name){
      this.$store.commit('updateshowSongMore',true);
      this.$store.commit('updatenext',{
        id,
        name
      });
    },
    sMove(iscroll){
      let Y = -iscroll.y;
      let H1 = this.$refs.ranks.querySelector('.my-header').offsetHeight;
      let H2 = this.$refs.ranks.querySelector('.songs-hh').offsetHeight;
      if(Y > H2 - H1){
        return;
      }
      if(Y <= H2 - H1){
          this.bgOpacity = + (Y/(H2 - H1)).toFixed(1)*.8;
          this.HOpacity = + (((H2 - H1)-Y)/(H2 - H1)).toFixed(1);
      }
      if(Y >= H1){
         this.songsTitle = this.songsDetail.name;
      }else{
         this.songsTitle = '歌单';
      }
    },
    isTextOver(text){
      let len = text.split('').length;
      return len > 13 ? true : false;
    },
   searchList(value){
      this.searchResult = [];
      if(value == ''){
         this.searchResult = [];
         return;
      }
      this.songsDetail.tracks.forEach((item)=>{
        let pat = new RegExp(value);
        if(pat.test(item.name)){
            this.searchResult.push(item);
        }
      })
    },

  }
  
}
</script>
<style lang="scss" scoped>
@import 'src/common/css/mixin.scss';
.rank-songs{
  position:fixed;
  top:0;
  width:100%;
  bottom:131/$a+rem;
  overflow-x: hidden;
  overflow-y: auto;
  background:#ccc;
  .my-header{
    position: fixed;
    top:0;
    left:0;
  }
  .scroll-content{
    position:fixed;
    top:0;
    width:100%;
    bottom:131/$a+rem;
    overflow-y: auto;
  }
  .h4-header{
    color:white;
    padding: 28/$a+rem 0;
    width:554/$a+rem;
    overflow: hidden;
    .h-title{
       width:100%;
       font-weight: normal;
       font-size:1.1em;
       white-space: nowrap;
    }
    .h-title-move{
      animation: titlemove 8s linear infinite;
      -webkit-animation: titlemove 8s linear infinite;
    }
    @keyframes titlemove {
        from{transform: translate3d(0,0,0)}
        to{transform: translate3d(-100%,0,0)}
    }
    @-webkit-keyframes titlemove {
        from{-webkit-transform: translate3d(0,0,0)}
        to{-webkit-transform: translate3d(-100%,0,0)}
    }
  }
  .h-search{
     width:128/$a+rem;
     @include center;
     .icon-search02{
       font-size:1.1rem;
       color:white;
     }

  }
  .h-more{
    width:128/$a+rem;
    @include center;
    .icon-more3{
       font-size:1.2rem;
       color:white;

    }

  }
  .songs-hh{
    height: 676/$a+rem;
   color:white;
    padding-top:137/$a+rem;
    position:relative;
    overflow: hidden;
    .songs-bg{
      width:100%;
      height:100%;
      position:absolute;
      left: 0;
      top:0;
      z-index:-1;
    }
    .bg-cover{
      width:100%;
      height:100%;
      position:absolute;
      left: 0;
      top:0;
      background:rgba(0,0,0,.5);
      z-index:3;
    }
    .front{
      -webkit-filter: blur(50px);
      -moz-filter: blur(50px);
      -o-filter: blur(50px);
      -ms-filter: blur(50px);
      filter: blur(50px);
      z-index:1;
    }
    .hh-detail{
      height:385/$a+rem;
      display: flex;
      display: -webkit-flex;
      position: relative;
      z-index:8;
      .h-img-box{
        width: 398/$a+rem;
        height:100%;
        position: relative;
        .h-img-box-con{
          background:gray;
          position: absolute;
          right: 0;
          top:14/$a+rem;
          width: 338/$a+rem;
          height:338/$a+rem;
          img{
            position: absolute;
            right: 0;
            top:0;
            width: 338/$a+rem;
            height:338/$a+rem;
          }
          .description{
            position: absolute;
            right: 0;
            bottom:0;
            width:80/$a+rem;
            height:80/$a+rem;
          }
          .warns{
            position: absolute;
            bottom:5/$a+rem;
            right:5/$a+rem;
            font-size:1.1rem;
            color:white;
          }
          .play-count{
            position: absolute;
            right: 12/$a+rem;
            top:12/$a+rem;
            .icon-kefu{
              font-size:.7rem;
              color:white;
              display: inline-block;
              vertical-align: middle;
            }
            .p-c-text{
              font-size:.95em;
              display: inline-block;
              vertical-align: middle;
            }
          }

        }
      }
      .list-name{
        padding: 0 0 110/$a+rem 40/$a+rem;
        .h6-text{
          font-size:1.2em;
          font-weight:500;
          max-width:450/$a+rem;
          margin-top:26/$a+rem;
        }
        .update-time{
          font-size:.8em;
          color:rgba(255,255,255,.8);
          margin-top:20/$a+rem;

        }
        .l-n-d{
            height:72/$a+rem;
            margin-top:66/$a+rem;
            .l-n-d-img,img,.l-n-d-text,.icon-more{
              display: inline-block;
              vertical-align: middle;
            }
            .l-n-d-img{
              width: 74/$a+rem;
              height: 74/$a+rem;
              margin-right: 16/$a+rem;
              border-radius:50%;
              position: relative;
              .vip-type{
                position: absolute;
                width: 34/$a+rem;
                height: 34/$a+rem;
                bottom:-5/$a+rem;
                right:-5/$a+rem;
                background:#ffbd20;
                border-radius:50%;
                @include center;
                i{
                  font-size:.4rem;
                  position: absolute;
                  top:3/$a+rem;
                  left:6/$a+rem;
                }
              }
            }
            img{
              width: 74/$a+rem;
              height: 74/$a+rem;
              border-radius:50%;
              
            }
            .l-n-d-text{
              color:#dad9d9;
              font-size:.95em;
            }
            .icon-more{
                font-size:.7rem;
                position: relative;
                top:3/$a+rem;
            }
         }
        
      }
    }
  }
  .hh-control{
    width:100%;
    height: 154/$a+rem;
    display: flex;
    display: -webkit-flex;
    position: relative;
    z-index:8;
    .h-c-item{
      width:25%;
      height:100%;
      display: flex;
      flex-direction: column;
      display: -webkit-flex;
      align-items: center;
      justify-content: center;
      .h-c-i-text{
        margin-top:2px;
      }
      .icon-add{
        font-size:1.15rem;
        position: relative;
        top:6/$a+rem;
      }
      .icon-pinglun1{
        font-size:1rem;
        position: relative;
        top:5/$a+rem;

      }
      .icon-share{
        font-size:1.3rem;
        position: relative;
        top:-5/$a+rem;

      }
      .icon-share ~ .h-c-i-text{
         position: relative;
        top:-5/$a+rem;
      }
      .icon-xiazai1{
        font-size:1.1rem;
        position: relative;
        top:4/$a+rem;
      }

    }

  }
  .songs-list{
    background:#f2f4f5;
    .songs-list-header{
      height:134/$a+rem;
      line-height:134/$a+rem;
      border-bottom:1px solid #e4e5e6;
      position: relative;
      display: flex;
      display: -webkit-flex;
      .icon-box{
        width:133/$a+rem;
        @include center;
        .icon-bofang{
           font-size:1rem;
        }
      }
      .icon-text{
        font-size:1.12em;
        .counts{
          color:#8d8e8f;
          font-size:0.95em;
        }
      }
      .multi-c{
        width:174/$a+rem;
        position:absolute;
        right:0;
        top:0;
        height:100%;
        @include center;
        justify-content: space-around;
        i,span{
          display: inline-block;
          vertical-align: middle;
        }
        span{
          margin-right: 18/$a+rem;
        }
        .icon-gengduo{
           font-size:.9rem;
        }

      }

    }
    .s-l-con{
      
      .s-l-ul{
        .s-l-li{
          height:147/$a+rem;
          display: flex;
          display: -webkit-flex;
          .num{
            width: 133/$a+rem;
            height:100%;
            @include center;
            color:#8f9091;
            text-align: center;
            font-size: 1em;
            i{
              font-size: 1rem;
              color: #d33a31;
            }
          }
          .text-right{
            width:86.14%;
            border-bottom:1px solid #e7e9ea;
            position: relative;
            display: flex;
            display: -webkit-flex;
            align-items: center;
            .s-l-li-text{
               max-width: 700/$a+rem;
               height:100%;
               padding: 14/$a+rem 0;
                .song-name{
                  color:#313231;
                  font-size:1.1em;
                  white-space: nowrap;
                  overflow: hidden;
                  text-overflow: ellipsis;
                  font-weight:500;
                  .alias{
                    color:#777879;
                    font-size:.9em;
                  }
                  .mv{
                    position: relative;
                    top:6/$a+rem;
                    .icon-MV{
                      font-size:1rem;
                      color:$bgcolor;
                    }
                  }
                }
                .singer{
                  font-size:.9em;
                  color:#777879;
                  white-space: nowrap;
                  overflow: hidden;
                  text-overflow: ellipsis;
                  margin-top:10/$a+rem;
                }

              }
              .more{
                width: 126/$a+rem;
                position: absolute;
                right:0;
                top:0;
                height: 100%;
                @include center;
                .icon-more3{
                  color:#aaabac;
                  font-size:1.1rem;
                }

              }
          }
          
        }

      }

    }
  }

}
</style>


