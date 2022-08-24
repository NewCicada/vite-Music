<template>
  <div id="app">
    <h1 class="title">- 爸爸妈妈の网易云评论 -</h1>
    <!-- 视频 -->
    <div id="video">
       <iframe
          src="//player.bilibili.com/player.html?bvid=BV18s411X7Yi"
          scrolling="no"
          border="0"
          frameborder="no"
          framespacing="0"
          allowfullscreen="true"
      />
    </div>
    <!-- 评论部分 -->
    <h2 class="subtitle">热评 TOP15</h2>
    <div id="hot-comment-list">
      <HelloWorld
          v-for="(item,index) in hotCommentList"
          :key="index"
          :comment-obj="item"/>
    </div>

    <h2 class="subtitle">评论回忆</h2>
    <div id="comment-list">
      <HelloWorld
          v-for="(item,index) in commentList"
          :key="index"
          :comment-obj="item"/>
    </div>
    <div class="load-more">
      <a class="load-more-button" @click="loadMoreComments">
        {{loading ? "加载中..." : "加载更多"}}
      </a>
    </div>
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'
import Axios from "axios"
import Bricks from "bricks.js"

export default {
  name: 'App',
  components: {
    HelloWorld
  },
  data() {
    return {
      page: 1,
      hotCommentList: [],
      commentList: [],
      hotCommentInst: undefined,
      commentInst: undefined,
      loading: false
    }
  },
  mounted() {
    // 必须在mounted而不是created时实例化Bricks，
    // 否则document.querySelector("#xxx")获取不到container
    this.initBricks();
    this.loadHotComments();
    this.loadMoreComments();
  },
  methods: {
    initBricks() {
      let sizeOpt = [
        { columns: 2, gutter: 10 },
        { mq: "1000px", columns: 3, gutter: 10 },
        { mq: "1300px", columns: 4, gutter: 12 },
        { mq: "1600px", columns: 5, gutter: 14 },
      ];
      this.hotCommentInst = Bricks({
        container: "#hot-comment-list",
        packed: "data-packed",
        sizes: sizeOpt,
      })
      this.commentInst = Bricks({
        container: "#comment-list",
        packed: "data-packed",
        sizes: sizeOpt,
      });
    },
    getData(limit = 50, offset) {
      return Axios.get("https://netease-cloud-music-api-omega-murex.vercel.app/comment/music", {
        withCredentials: true,
        params: {
          id: 407450223,
          limit,
          offset, // 倒序
        },
      }).then((res) => res.data);
    },
    loadHotComments() {
      this.getData(20).then((data) => {
        this.hotCommentList = data.hotComments;
        // 必须用$nextTrick，否则pack不生效
        this.$nextTick(() => {
          this.hotCommentInst.pack()
        })
      });
    },
    loadMoreComments() {
      if (this.loading) return;
      this.loading = true;
      this.getData(50, 2027450 - this.page * 50).then((data) => {
        this.commentList = this.commentList.concat(data.comments);
        // 必须用$nextTrick，否则pack不生效
        this.$nextTick(() => {
          this.commentInst.pack()
          this.page === 1 ? this.commentInst.pack() : this.commentInst.update();
        })
        this.page++;
      }).finally(() => {
        this.loading = false;
      });
    }
  }
}
</script>

<style>
body {
  background-color: #cd353b;
  font-family: -apple-system,BlinkMacSystemFont,"Segoe UI","Roboto","Oxygen","Ubuntu","Cantarell","Fira Sans","Droid Sans","Helvetica Neue",sans-serif;
}

#app {
  padding: 20px;
}
#iframe {
  width: 45%;
  height: 430px;
}

.title {
  color: #fff;
  text-align: center;
  margin-bottom: 30px;
}

.subtitle {
  border-left: 6px solid #fff;
  padding-left: 10px;
  color: #eee;
}

/* 视频部分 */
#video {
  width: 70%;
  height: 500px;
  margin: 0 auto 10px;
  padding: 0;
}
#video iframe {
  width: 100%;
  height: 100%;
}

#hot-comment-list {
  margin-bottom: 40px;
}

.load-more {
  margin-top: 30px;
  text-align: center;
}

.load-more .load-more-button {
  cursor: pointer;
  background-color: #fff;
  border-radius: 4px;
  width: 100px;
  padding: 10px;
  display: inline-block;
  color: #cd353b;
}

.load-more-button:hover {
  background-color: #cd353b;
  color: #fff;
  border: 1px solid #fff;
}

</style>
