<template>
  <div class="page">
    <v-header>
      <div class="title" slot="title">个人信息</div>
      <div class="comeBackBtn" slot="left" @click="comeBack"></div>
    </v-header>
    <div id="userInfo">
      <div>
        <div class="accountCon">
          <div class="accountImg"><img :src="userImg" alt="" width="100%"></div>
          <span class="accountName" v-text="userName"></span>
          <div class="accountOther">
            <span class="coin" v-text="'积分：'+score"></span>
            <span class="time" v-text="createdTime"></span>
          </div>
        </div>       
        <v-collapse @updateEve = 'updateEve' v-for="item in list" :item = 'item'></v-collapse>
        <div class="cancelBtn" @click="cancel($router)">注销</div>
      </div>
    </div>
  </div>
</template>

<script>
import { mapState, mapMutations } from 'vuex';
import BScroll from 'better-scroll';
import Header from './header';

const Collapse = {
  template: `<div class="collapseCon iconfont">
        <div class="collapseHead" @click="showView">
          <p class="columnHead">{{item.name}}<span :class='["arrow",showList?"":"show"]'>&#xe6dc;</span></p>
        </div>
        <ul v-show="showList" v-if='item.lists.length>0'>
          <li v-for="value in item.lists" @click='showTopic(value.id)'>{{value.title}}</li>
        </ul>
        <ul v-show="showList" v-else>
          <li>空</li>
        </ul>
      </div>`,
  data() {
    return {
      showList: false,
    };
  },
  methods: {
    showView() {
      this.showList = !this.showList;
    },
    showTopic(id) {
      this.$router.push(`/homepage/topic/${id}`);
    }
  },
  updated() {
    this.$emit('updateEve');
  },
  props: ['item']
};
export default {
  name: 'userInfo',
  data() {
    return {
      score: '',
      time: 0,
      list: [
        {
          name: '我的收藏',
          lists: [],
        },
        {
          name: '最近回复',
          lists: [],
        },
        {
          name: '最近发布',
          lists: [],
        },
      ]
    };
  },
  computed: {
    ...mapState(['accesstoken', 'userName', 'userImg', 'userId']),
    createdTime() {
      if (this.time !== 0) {
        const nowTime = new Date();
        const createTime = new Date(this.time);
        const num = (nowTime - createTime) / (1000 * 3600 * 24);
        let timeStr;
        if (num < 1) {
          if (num * 24 < 1) {
            timeStr = parseInt(num * 24 * 60, 10);
            timeStr = timeStr === 0 ? 1 : timeStr;
            timeStr += '分钟前';
          } else {
            timeStr = parseInt(num * 24, 10);
            timeStr += '小时前';
          }
        } else {
          timeStr = parseInt(num, 10);
          timeStr += '天前';
        }
        return `注册于:${timeStr}`;
      }
      return '';
    }
  },
  methods: {
    ...mapMutations(['getCollect', 'getUserInfo', 'cancel']),
    comeBack() {
      console.log(this.$router);
      if (history.length === 1) {
        this.$router.push('/');
      } else {
        history.go(-1);
      }
    },
    updateEve() {
      if (this.scroll) {
        this.scroll.refresh();
      }
    },
  },
  created() {
    if (!this.accesstoken) {
      this.$router.push('/homepage/login');
    }
    if (this.userName) {
      this.list[0].lists.length = 0;
      this.list[1].lists.length = 0;
      this.list[2].lists.length = 0;
      this.getCollect((data) => {
        this.list[0].lists.push(...data);
      });
      this.getUserInfo((data) => {
        console.log(data);
        this.list[1].lists.push(...data.recent_replies);
        this.list[2].lists.push(...data.recent_topics);
        this.time = data.create_at;
        this.score = data.score;
      });
    }
  },
  mounted() {
    const wrapper = document.querySelector('#userInfo');
    this.scroll = new BScroll(wrapper, {
      click: true,
    });
  },
  updated() {
    this.scroll.refresh();
    console.log(1);
  },
  components: {
    'v-header': Header,
    'v-collapse': Collapse,
  },
  watch: {
    userName(newVal) {
      this.list[0].lists.length = 0;
      this.list[1].lists.length = 0;
      this.list[2].lists.length = 0;
      if (newVal) {
        this.getCollect((data) => {
          this.list[0].lists.push(...data);
        });
        this.getUserInfo((data) => {
          this.list[1].lists.push(...data.recent_replies);
          this.list[2].lists.push(...data.recent_topics);
          this.time = data.create_at;
          this.score = data.score;
        });
      }
    }
  }
};
</script>
<style >
.comeBackBtn {
  width: 1rem;
  height: 1rem;
  background: url("../assets/images/comeBackBtn.png") no-repeat;
  background-size: 0.4rem 0.4rem;
  background-position: center;
  position: absolute;
  top: 0;
  left: 0;
}
#userInfo {
  width: 96%;
  position: absolute;
  top: 1rem;
  left: 0;
  right: 0;
  bottom: 0;
  margin: auto;
  overflow: hidden;
}
.accountCon {
  display: flex;
  align-items: center;
  flex-direction: column;
  border-bottom: 1px solid rgb(224, 228, 228)
}
.accountImg{
  border-radius: 1.2rem;
  width: 1.2rem;
  height: 1.2rem;
  border: 0.1rem white solid;
  overflow: hidden;
  margin-top: 0.65rem;
}
.accountName{
  margin-top: 0.14rem;
  font-size: 0.3rem;
  font-weight: bold;
}
.accountOther{
  margin: 0.26rem 0;
  width: 5.5rem;
  font-size: 0.3rem;
}
.coin{
  float: left;
}
.time{
  float: right;
}
.collapseCon{
  height: auto;
  margin-top: 0.1rem;
}
.collapseHead {
  height: 0.78rem;
  display: flex;
  align-items: center;
  border-left: 3px solid orange;
  background-color: rgb(240,231,225);
}
.columnHead{
  margin-left: 0.2rem; 
}
.arrow{
  position: absolute;
  right: 0.2rem;
  transform: rotateZ(0deg);
  transition: transform 0.2s;
}
.arrow.show{
  transform: rotateZ(-90deg);
}
.collapseCon li{
  height: 0.78rem;
  display: flex;
  align-items: center;
  background-color: rgba(240,231,225,0.6);
  text-indent: 0.2rem;
  border-bottom: 1px rgb(235,234,231) solid;
}
.cancelBtn{
  height: 0.8rem;
  background: gray;
  margin: .5rem 0 0;
  line-height: 0.8rem;
  font-size: 0.3rem;
  color: white;
  border-radius: 0.05rem;
}
</style>

