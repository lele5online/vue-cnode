<template>
  <div class="content-box">
    <div class="bar-box">
      <div class="mask" v-if="menuShow" @click="closeMenu"></div>
      <div class="header" :class="getOpenClass">
        <div class="btn-menu" @click="openMenu">
          <span class="line"></span>
        </div>
        <p class="title">{{title}}</p>
      </div>
      <div class="sidebar" :class="getOpenClass">
        <div class="logo">
          <img src="//o4j806krb.qnssl.com/public/images/cnodejs_light.svg" alt="">
        </div>
        <div class="menu-list">
          <router-link :to="{name:'all',query:{tab:'all'}}">- 全部</router-link>
          <router-link :to="{name:'all',query:{tab:'ask'}}">- 问答</router-link>
          <router-link :to="{name:'all',query:{tab:'share'}}">- 分享</router-link>
          <router-link :to="{name:'all',query:{tab:'job'}}">- 招聘</router-link>
          <router-link :to="{name:'all',query:{tab:'good'}}">- 精华</router-link>
          <router-link :to="{name:'about'}">- 关于</router-link>
        </div>
      </div>
    </div>
    <div class="topic-box">
      <ul class="topic-list">
        <li v-for="topic in topics">
          <router-link :to="{name:'topic',params:{id: topic.id}}" :data-id="topic.id">
            <div class="topic-title">{{topic.title}}</div>
            <div class="topic-content">
              <div class="topic-avatar">
                <img class="avatar" :src="topic.author.avatar_url" alt="">
              </div>
              <div class="topic-info">
                <p>
                  <span class="topic-author">{{topic.author.loginname}}</span>
                  <span class="topic-total"><b>{{topic.reply_count}}</b>/{{topic.visit_count}}</span>
                </p>
                <p>
                  <span class="topic-ctime">Post:{{topic.create_at | getFreeTime}}</span>
                  <span class="topic-rtime">Reply:{{topic.last_reply_at| getFreeTime}}</span>
                </p>
              </div>
            </div>
          </router-link>
        </li>
      </ul>
    </div>
    <backTop></backTop>
  </div>
</template>
<script>
  import $ from 'webpack-zepto';
  import backTop from '../components/backtop.vue';
  export default {
    name: "all",
    components: {
      backTop,
    },
    filters: {
      getFreeTime: function (time) {
        var timestamp = +new Date(time)
        return formatTime(timestamp, "yyyy.MM.dd");
        function formatTime(time, format) {
          var t = new Date(time);
          var tf = function (i) {
            return ( i < 10 ? '0' : '') + i
          };
          return format.replace(/yyyy|MM|dd|HH|mm|ss/g, function (a) {
            switch (a) {
              case 'yyyy':
                return tf(t.getFullYear());
                break;
              case 'MM':
                return tf(t.getMonth() + 1);
                break;
              case 'mm':
                return tf(t.getMinutes());
                break;
              case 'dd':
                return tf(t.getDate());
                break;
              case 'HH':
                return tf(t.getHours());
                break;
              case 'ss':
                return tf(t.getSeconds());
                break;
            }
          })
        }
      }
    },
    data (){
      return {
        SCROLL_LOCK: false,
        menuShow: false,
        topics: [],
        params: {
          page: 1,
          limit: 20,
          tab: 'all',
          mdrender: true
        }
      }
    },
    mounted: function () {
      // 先判断是否存在tab
      if (this.$route.query && this.$route.query.tab) {
        this.params.tab = this.$route.query.tab; // 当前所在类别
        $(".menu-list a").eq(this.getTitle(this.$route.query.tab).idx).addClass("active").siblings("a").removeClass("active");
      } else {
        $(".menu-list a").eq(0).addClass("active").siblings("a").removeClass("active");
      }
      this.getTopics();
      $("body").removeClass("os-mode");

      // 滚动
      var self = this;
      $(window).on("scroll", function () {
        self.getScrollData()
      })
    },
    methods: {
      getTitle: function (val) {
        var obj = {};
        switch (val) {
          case 'ask':
            obj.title = "问答";
            obj.idx = 1;
            break;
          case 'share':
            obj.title = "分享";
            obj.idx = 2;
            break;
          case 'job':
            obj.title = "招聘";
            obj.idx = 3;
            break;
          case 'good':
            obj.title = "精华";
            obj.idx = 4;
            break;
          default:
            obj.title = "全部";
            obj.idx = 0;
            break;
        }
        return obj;
      },
      getTopics: function (type) {
        var self = this;
        $.get("https://cnodejs.org/api/v1/topics", self.params, function (res) {
          if (res && res.data) {  // 如果查到数据
            self.SCROLL_LOCK = true;
            self.topics = res.data;
          }
        })
      },
      // 滚动加载数据
      getScrollData() {
        if (this.SCROLL_LOCK) {

          var totalheight = $(window).height() + $(window).scrollTop();
          if ($(document).height() <= totalheight + 200) {
            this.SCROLL_LOCK = false;
            this.params.limit += 20;
            this.getTopics();
          }
        }
      },
      // 显示sidebar
      openMenu: function () {
        this.menuShow = true;
        $("body").addClass("os-mode");
      },
      // 收起sidebar
      closeMenu: function () {
        this.menuShow = false;
        $("body").removeClass("os-mode");
      },
    },
    computed: {
      title: function () {
        return this.getTitle(this.params.tab).title;
      },
      getOpenClass: function () {
        if (this.menuShow) {
          return "open";
        } else {
          return "";
        }
      }
    },
    watch: {
      $route: function (to, from) {
        // 如果是当前页面切换分类的情况
        if (to.query && to.query.tab) {
          this.params.tab = to.query.tab;
          $(".menu-list a").eq(this.getTitle(to.query.tab).idx).addClass("active").siblings("a").removeClass("active");
        } else {
          $(".menu-list a").eq(0).addClass("active").siblings("a").removeClass("active");
        }
        this.getTopics();
        // 隐藏导航栏
        this.menuShow = false;
        $("body").removeClass("os-mode");
      }
    }
  }
</script>
<style rel="stylesheet/scss" type="text/css" lang="scss" scoped>
  @import "../../assets/css/common/_variable.scss";

  .content-box {
    position: relative;
    width: 100%;
    .bar-box {
      .mask {
        position: fixed;
        left: 0;
        top: 0;
        width: 100%;
        height: 100%;
        background-color: rgba(0, 0, 0, 0.6);
        z-index: 9;
      }
      .header {
        position: fixed;
        left: 0;
        top: 0;
        width: 100%;
        height: 44px;
        box-shadow: 0 0 3px $border;
        z-index: 5;
        background-color: $white;
        transition: all 0.3s ease-out;
        .btn-menu {
          position: absolute;
          left: 0;
          top: 0;
          width: 44px;
          height: 44px;
          .line {
            position: absolute;
            display: block;
            left: 50%;
            top: 50%;
            width: 20px;
            height: 1px;
            transform: translate(-50%, -50%) translateZ(0);
            background-color: $black;
            &:before {
              content: '';
              position: absolute;
              left: 0;
              width: 100%;
              height: 100%;
              top: -5px;
              background-color: $black;
            }
            &:after {
              content: '';
              position: absolute;
              left: 0;
              width: 100%;
              height: 100%;
              top: 5px;
              background-color: $black;
            }
          }
        }
        .title {
          font-size: 16px;
          color: $black;
          text-align: center;
          line-height: 44px;
        }
      }
      .sidebar {
        position: fixed;
        z-index: 99;
        left: 0;
        top: 0;
        width: 150px;
        height: 100%;
        background-color: $white;
        transition: all 0.3s ease-out;
        transform: translate(-150px, 0) translateZ(0);
        .logo {
          margin: 0 15px;
          padding: 20px 0;
          width: 120px;
          border-bottom: solid 1px $border;
          img {
            background-color: #444444;
          }
        }
        .menu-list {
          padding: 20px;
          a {
            display: block;
            line-height: 24px;
            font-size: 16px;
            padding: 10px 0;
          }
          a.active {
            color: $green;
          }
        }
      }
      .header.open {
        transform: translate(150px, 0) translateZ(0);
      }
      .sidebar.open {
        transform: translate(0, 0) translateZ(0);
      }
    }
    .topic-box {
      padding-top: 44px;
      background-color: $white;
      .topic-list {
        li {
          border-bottom: solid 1px $border;
          a {
            padding: 10px 15px;
            display: block;
            .topic-title {
              color: $black;
              font-size: 16px;
              font-weight: bolder;
              line-height: 24px;
              height: 24px;
              text-overflow: ellipsis;
              overflow: hidden;
              white-space: nowrap;
            }
            .topic-content {
              display: flex;
              padding-top: 10px;
              .topic-avatar {
                width: 40px;
                height: 40px;
                margin-right: 10px;
                border-radius: 50%;
                overflow: hidden;
              }
              .topic-info {
                flex: 1;
                width: 0;
                p {
                  display: flex;
                  padding: 3px 0;
                  span:first-child {
                    flex: 1;
                    width: 0;
                  }
                  span:last-child {
                    text-align: right;
                  }
                  .topic-author {
                    color: $secondBlack;
                  }
                  .topic-total {
                    color: $secondBlack;
                    b {
                      color: $green;
                    }
                  }
                  .topic-ctime {
                    color: $dark;
                  }
                  .topic-rtime {
                    color: $dark;
                  }
                }
              }
            }
          }
        }
      }
    }
  }
</style>
