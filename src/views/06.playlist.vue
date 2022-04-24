<template>
  <div class="playlist-container">
    <div class="top-wrap">
      <div class="img-wrap">
        <!-- 封面 -->
        <img :src="playlist.coverImgUrl" alt="" />
      </div>
      <div class="info-wrap">
        <!-- 名字 -->
        <p class="title">{{ playlist.name }}</p>
        <div class="author-wrap">
          <!-- 头像 -->
          <img class="avatar" :src="playlist.creator.avatarUrl" alt="" />
          <span class="name">{{ playlist.creator.nickname }}/</span>
          <span class="time">{{ playlist.createTime }} 创建</span>
        </div>
        <div class="play-wrap">
          <span class="iconfont icon-circle-play"></span>
          <span class="text">播放全部</span>
        </div>
        <div class="tag-wrap">
          <span class="title">标签:</span>
          <!-- 分类标签 -->
          <ul>
            <li v-for="(item, index) in playlist.tags" :key="index">
              {{ item }}
            </li>
          </ul>
        </div>
        <div class="desc-wrap">
          <span class="title">简介:</span>
          <!-- 简介 -->
          <span class="desc">
            {{ playlist.description }}
          </span>
        </div>
      </div>
    </div>
    <el-tabs v-model="activeIndex">
      <el-tab-pane label="歌曲列表" name="1">
        <table class="el-table playlit-table">
          <thead>
            <th></th>
            <th></th>
            <th>音乐标题</th>
            <th>歌手</th>
            <th>专辑</th>
            <th>时长</th>
          </thead>
          <tbody>
            <tr class="el-table__row" v-for="(item,index) in playlist.tracks" :key="index"
            @click="play(item.id)">
              <td>{{ index+1 }}</td>
              <td>
                <div class="img-wrap">
                  <img :src="item.al.picUrl" alt="" />
                  <span class="iconfont icon-play"></span>
                </div>
              </td>
              <td>
                <div class="song-wrap">
                  <div class="name-wrap">
                    <span>{{item.name}}</span>
                    <span class="iconfont icon-mv"  v-if="item.mv!== 0" @click="toMV(item.mv)"></span>
                  </div>
                  
                </div>
              </td>
              <td>{{item.ar[0].name}}</td>
              <td>{{item.al.name}}</td>
              <td>{{item.dt}}</td>
            </tr>

          </tbody>
        </table>
      </el-tab-pane>
      <el-tab-pane :label="`评论(${total})`" name="2">
        <!-- 精彩评论 -->
        <div class="comment-wrap">
          <p class="title">
            精彩评论
            <span class="number">({{ hotCount }})</span>
          </p>
          <div class="comments-wrap">
            <!-- 评论 -->
            <div v-for="(item, index) in hotComment" :key="index" class="item">
              <div class="icon-wrap">
                <!-- 头像 -->
                <img :src="item.user.avatarUrl" alt="" />
              </div>
              <div class="content-wrap">
                <div class="content">
                  <!-- 昵称 -->
                  <span class="name">{{ item.user.nickname }}:</span>
                  <span class="comment">{{ item.content }}</span>
                </div>
                <!-- 评论的回复 -->
                <div class="re-content" v-if="item.beReplied.length != 0">
                  <span class="name"
                    >{{ item.beReplied[0].user.nickname }}：</span
                  >
                  <span class="comment">{{ item.beReplied[0].content }}</span>
                </div>
                <div class="date">{{item.time}}</div>
              </div>
            </div>
          </div>
        </div>
        <!-- 最新评论 -->
        <div class="comment-wrap">
          <p class="title">
            最新评论
            <span class="number">( {{total}} )</span>
          </p>
          <div class="comments-wrap">
            <div class="item" v-for="(item,index) in comments" :key="index">
              <div class="icon-wrap">
                <img :src="item.user.avatarUrl" alt="" />
              </div>
              <div class="content-wrap">
                <div class="content">
                  <span class="name">{{item.user.nickname}}：</span>
                  <span class="comment">{{ item.content }}</span>
                </div>
                <div class="re-content" v-if="item.beReplied.length!=0">
                  <span class="name">{{ item.beReplied[0].user.nickname }}：</span>
                  <span class="comment">{{ item.beReplied[0].content }}</span>
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
        ></el-pagination>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script>
// 导入 axios
import axios from "axios";
export default {
  name: "playlist",
  data() {
    return {
      activeIndex: "1",
      // 总条数
      total: 0,
      // 页码
      page: 1,
      // 歌单详情数据
      // tracks 歌曲列表
      playlist: {},
     
      // 热门评论
      hotComment: [],
      // 热门评论的个数
      hotCount: 0,
      limit:10,
      currentMusicIndex: 0, // 当前播放的歌曲在本页歌单中的索引
      // 普通评论
      comments:[]
    };
  },
  created() {
    // 获取歌单详情
    axios({
      url: "https://autumnfish.cn/playlist/detail",
      method: "get",
      params: {
        id: this.$route.query.q
      }
    }).then(res => {
      // console.log(res.data.playlist.tracks)
      this.playlist = res.data.playlist;
      //创建时间
      
          var unixTimestamp = new Date(this.playlist.createTime) ;
          //重载方法
          Date.prototype.toLocaleString = function() {
                this.hour = this.getHours() < 10 ? "0" + this.getHours() : this.getHours();
                this.minute = this.getMinutes() < 10 ? "0" + this.getMinutes() : this.getMinutes();
                this.second = this.getSeconds() < 10 ? "0" + this.getSeconds() : this.getSeconds();
                return this.getFullYear() + "/" + (this.getMonth() + 1) + "/" + this.getDate() + "/ " + this.hour  + ":" + this.minute + ":" + this.second;
          };
          this.playlist.createTime = unixTimestamp.toLocaleString();
        
      let arr=this.playlist.tracks;
       for(let i=0;i<arr.length;i++){ 
          let duration = arr[i].dt;
          //console.log(duration+"我是时间");  
          let min = parseInt(duration/1000/60);
          if(min<10){
            min='0'+min
          }
          let sec = parseInt(duration/1000%60)
          if(sec<10){
            sec='0'+sec
          }
          //console.log(min+" "+sec)
          this.playlist.tracks[i].dt=`${min}:${sec}`       
        }
    });
    // 获取 评论
    axios({
      url: "https://autumnfish.cn/comment/hot",
      method: "get",
      params: {
        id: this.$route.query.q,
        // 传递类型
        type: 2
      }
    }).then(res => {
      // console.log(res)
      this.hotComment = res.data.hotComments;
      // 保存个数
      this.hotCount = res.data.total;
      for(let i=0;i<this.hotComment.length;i++){
          var unixTimestamp = new Date(this.hotComment[i].time) ;
          //重载方法
          Date.prototype.toLocaleString = function() {
                this.hour = this.getHours() < 10 ? "0" + this.getHours() : this.getHours();
                this.minute = this.getMinutes() < 10 ? "0" + this.getMinutes() : this.getMinutes();
                this.second = this.getSeconds() < 10 ? "0" + this.getSeconds() : this.getSeconds();
                return this.getFullYear() + "/" + (this.getMonth() + 1) + "/" + this.getDate() + "/ " + this.hour  + ":" + this.minute + ":" + this.second;
          };
          this.hotComment[i].time = unixTimestamp.toLocaleString();
        }
    });
    // 获取 最新评论
    this.getNewlist()
  },
  methods: {
    getNewlist(){
      axios({
      url:"https://autumnfish.cn/comment/playlist",
      method:"get",
      params:{
        id:this.$route.query.q,
        limit:this.limit,
        offset:(this.page-1)*this.limit
      }
    }).then(res=>{
      // console.log(res)
      // 总个数
      this.total = res.data.total
      // 评论数据
      this.comments = res.data.comments
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
    handleCurrentChange(val) {
      // console.log(`当前页: ${val}`);
      // 保存页码
      this.page = val
      this.getNewlist()
    },
     //播放
    play(id){
       axios({
          url: 'https://autumnfish.cn/song/url',
          method: 'get',
          params: {
            id // id:id
          }
        }).then(res => {
            //console.log(res)
          let url = res.data.data[0].url
          if(url==null||url==""||url==undefined){
            this.$message({
              message: '您没有取得权限！',
              type: 'warning'
            })
            
          }else{
            // console.log(this.$parent.musicUrl)
          // 设置给父组件的 播放地址
          this.$parent.musicUrl = url
          }
          
        })
    },
    toMV(id){
      console.log(id)
        this.$router.push(`/mv?q=${id}`)
      }
  }
};
</script>

<style></style>