<template>
  <div class="mv-container">
    <div class="mv-wrap">
      <h3 class="title">mv详情</h3>
      <!-- mv -->
      <div class="video-wrap">
        <video
          autoplay
          controls
          :src="url"
        ></video>
      </div>
      <!-- mv信息 -->
      <div class="info-wrap">
        <div class="singer-info">
          <div class="avatar-wrap">
            <img :src="icon" alt="" />
          </div>
          <span class="name">{{ mvInfo.artistName }}</span>
        </div>
        <div class="mv-info">
          <h2 class="title">{{ mvInfo.name }}</h2>
          <span class="date">发布：{{ mvInfo.publishTime }}</span>
          <span class="number">播放：{{ mvInfo.playCount }}次</span>
          <p class="desc">
           {{ mvInfo.desc }}
          </p>
        </div>
      </div>
      <!-- 精彩评论 -->
      <div class="comment-wrap" v-if="hotComments!=undefined">
        <p class="title">精彩评论<span class="number">{{ hotTotal }}</span></p>
        <div class="comments-wrap">
          <div class="item" v-for="(item,index) in hotComments" :key="index">
            <div class="icon-wrap">
              <img :src="item.user.avatarUrl" alt="" />
            </div>
            <div class="content-wrap">
              <div class="content">
                <span class="name">{{item.user.nickname}}：</span>
                <span class="comment">{{item.content}}</span>
              </div>
              <!-- 评论回复 -->
                <div class="re-content" v-if="item.beReplied.length!=0">
                  <span class="name">{{item.beReplied[0].user.nickname}}:</span>
                  <span class="comment">{{item.beReplied[0].content}}</span>
                </div>
              <div class="date">{{item.time}}</div>
            </div>
          </div>
        </div>
      </div>
 <!-- 最新评论 -->
      <div class="comment-wrap">
        <p class="title">最新评论<span class="number">({{total}})</span></p>
        <div class="comments-wrap">
          <div class="item" v-for="(item,index) in comments" :key="index">
            <div class="icon-wrap">
              <img :src="item.user.avatarUrl"  />
            </div>
            <div class="content-wrap">
              <div class="content">
                <span class="name">{{item.user.nickname}}：</span>
                <span class="comment">{{item.content}}</span>
              </div>
              <!-- 评论回复 -->
                <div class="re-content" v-if="item.beReplied.length!=0">
                  <span class="name">{{item.beReplied[0].user.nickname}}：</span>
                  <span class="comment">{{item.beReplied[0].content}}</span>
                </div>
              <div class="date">{{item.time}}</div>
            </div>
          </div>
        
        
        </div>
      </div>
      <!-- 分页器 -->
      <el-pagination
        @current-change="handleCurrentChange"
        background
        layout="prev, pager, next"
        :total="total"
        :current-page="page"
        :page-size="5"
        :limit="limit"
      >
      </el-pagination>
    </div>
    <div class="mv-recommend">
      <h3 class="title">相关推荐</h3>
      <div class="mvs">
        <div class="items">
          <div v-for="(item,index) in simiMvs" :key="index" class="item" @click="toMV(item.id)" >
            <div class="img-wrap">
              <img :src="item.cover" alt="" />
              <span class="iconfont icon-play"></span>
              <div class="num-wrap">
                <div class="iconfont icon-play"></div>
                <div class="num">{{ item.playCount }}</div>
              </div>
              <span class="time">{{ item.duration }}</span>
            </div>
            <div class="info-wrap">
              <div class="name">{{ item.name }}</div>
              <div class="singer">{{ item.artistName }}</div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
export default {
  name: 'mv',
  data() {
    return {
      // 总条数
      total: 20,
      // 页码
      page: 1,
      // 页容量
      limit: 10,
      //mv地址
      url:"",
      //mv推荐
      simiMvs:[],
      //mv信息
      mvInfo:{},
      //歌手头像
      icon:"",
      //热门评论
      hotComments:[],
      hotTotal:0,
      //普通评论数
      comTotal:0,
      comments:[]
    };
  },
  created(){
    //获取mv地址
    axios({
      url:"https://autumnfish.cn/mv/url",
      method:"get",
      params:{
        id:this.$route.query.q
      }
    }).then(res=>{
      this.url=res.data.data.url
    })
    //获取相关mv
    axios({
      url:"https://autumnfish.cn/simi/mv",
      method:"get",
      params:{
        mvid:this.$route.query.q
      }
    }).then(res=>{
      this.simiMvs=res.data.mvs
      for(let i=0;i<this.simiMvs.length;i++){
          if(this.simiMvs[i].playCount>100000){
              this.simiMvs[i].playCount=parseInt(this.simiMvs[i].playCount/10000)+'万'
            }
          let duration = this.simiMvs[i].duration;  
          let min = parseInt(duration/1000/60);
          if(min<10){
            min='0'+min
          }
          let sec = parseInt(duration/1000%60)
          if(sec<10){
            sec='0'+sec
          }
          //console.log(min+" "+sec)
          this.simiMvs[i].duration=`${min}:${sec}`
        }
    })
    //获取mv信息
    axios({
      url:"https://autumnfish.cn/mv/detail",
      method:"get",
      params:{
        mvid:this.$route.query.q
      }
    }).then(res=>{
      this.mvInfo=res.data.data
      //获取歌手信息
      axios({
        url:"https://autumnfish.cn/artists",
        method:"get",
        params:{
          id:this.mvInfo.artists[0].id
        }
      }).then(res=>{
        this.icon=res.data.artist.picUrl
      })
    })
   this.getcomments()
  },
  methods: {
     getMvList(){
     axios({
      url:' https://autumnfish.cn/comment/mv',
      method:'get',
      params:{
        id:this.$route.query.q,
        limit:this.limit,
        offset:(this.page-1)*this.limit
      }
    }).then(res => {
      console.log(res);
      this.hotComments=res.data.hotComments;
      this.comments=res.data.comments;
    })
    },
    //评论
    getcomments(){
       //mv评论
   axios({
      url:' https://autumnfish.cn/comment/mv',
      method:'get',
      params:{
        id:this.$route.query.q,
        limit:this.limit,
        offset:(this.page-1)*this.limit
      }
    }).then(res => {
      //console.log(res);
      this.hotComments=res.data.hotComments;
      this.hotTotal=this.hotComments.length;
      this.comments=res.data.comments;
      this.total=res.data.total;
      for(let i=0;i<this.hotComments.length;i++){
          var unixTimestamp = new Date(this.hotComments[i].time) ;
          //重载方法
          Date.prototype.toLocaleString = function() {
                this.hour = this.getHours() < 10 ? "0" + this.getHours() : this.getHours();
                this.minute = this.getMinutes() < 10 ? "0" + this.getMinutes() : this.getMinutes();
                this.second = this.getSeconds() < 10 ? "0" + this.getSeconds() : this.getSeconds();
                return this.getFullYear() + "/" + (this.getMonth() + 1) + "/" + this.getDate() + "/ " + this.hour  + ":" + this.minute + ":" + this.second;
          };
          this.hotComments[i].time = unixTimestamp.toLocaleString();
        }
         for(let i=0;i<this.comments.length;i++){
          var unixTimestamp = new Date(this.comments[i].time) ;
          //重载方法
          Date.prototype.toLocaleString = function() {
                this.hour = this.getHours() < 10 ? "0" + this.getHours() : this.getHours();
                this.minute = this.getMinutes() < 10 ? "0" + this.getMinutes() : this.getMinutes();
                this.second = this.getSeconds() < 10 ? "0" + this.getSeconds() : this.getSeconds();
                return this.getFullYear() + "/" + (this.getMonth() + 1) + "/" + this.getDate() + "/ " + this.hour  + ":" + this.minute + ":" + this.second;
          };
          this.comments[i].time = unixTimestamp.toLocaleString();
        }
    })
    },
    // 去mv详情页
      toMV(id){
      this.$router.push(`/mv?q=${id}`)
			this.$router.go(0)
      },
    handleCurrentChange(val) {
      this.page=val;
      this.getMvList()
      this.getcomments()
    }
  
  }
  
};
</script>

<style></style>
