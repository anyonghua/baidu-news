<template>
    <div class="news-list">
        <ul class="list" ref="newsList"
            v-infinite-scroll="loadMore"
            infinite-scroll-disabled="loading"
            infinite-scroll-immediate-check=false
            infinite-scroll-distance="40">
            <li class="list-item" v-for="story in this.$store.state.stories" :key="story.id" @click="viewDetail(story.title, story.url)">
                <!--<a :href="story.url">-->
                    <span class="item-title">{{story.title}}</span>
                    <div class="image-wrapper">
                        <img class="item-image" v-lazy.newsList="attachImageUrl(story.images[0])" :alt="story.title">
                        <i class="icon iconfont icon-duotu multipic" v-if="story.multipic">多图</i>
                    </div>
                <!--</a>-->
            </li>
        </ul>
    </div>
</template>

<script>
import axios from 'axios'
import router from '../router'
import {Indicator} from 'mint-ui'

export default {
  data () {
    return {
      loading: false,
      date: Date,
      dateStr: '' // 加载数据日期字符串格式
    }
  },
  created () {
    console.log('create()...isFirstLoad=' + this.$store.state.isFirstLoad);
    // 判断是否是第一次进入首页
    if (this.$store.state.isFirstLoad) {
      this.fetchData()
      this.$store.dispatch('changeLoadState')
    }
    this.initDate()

    // 下拉刷新数据
    this.$on('refresh', () => {
      this.refreshData()
    })
  },
  methods: {
    // 跳转到对应id的文章详情页
    viewDetail: function (title, url) {
      this.$store.dispatch('changeCurrentNewsId', title);
      this.$store.dispatch('changeNewsType', 0);
      // if (this.$store.state.isFirstLoad) {
      // this.$store.dispatch('changeLoadState');
      // }
      router.push({name: 'newsDetail', params: {id: title, url: url}})
    },
    // 修改图片链接
    attachImageUrl: function (srcUrl) {
      if (srcUrl !== undefined) {
        return srcUrl.replace(/http\w{0,1}:\/\/p/g, 'https://images.weserv.nl/?url=p')
      }
    },
    // 获取最新新闻数据列表
    fetchData: function () {
      // TODO 代入新闻API URL
      axios.post('新闻API')
        .then(response => {
          // 初始化新闻内容和id数组，并添加进state
          let newslist = response.data.data.result;
          // console.log(JSON.stringify(newslist));

          let stories = [];
          let ids = [];
          if (newslist != null) {
            newslist.forEach(news => {
              // console.log('NEWS=>'+JSON.stringify(news));
              // console.log('NEWS.id=>' + news.data.id);
              // console.log('NEWS.title=>' + news.data.title);
              // console.log('NEWS.type=>' + news.type);
              // console.log('NEWS.url=>' + news.data.url);

              if (news.type === 'news') {
                ids.push(news.data.id);

                let story = {};
                story.id = news.data.id;
                story.title = news.data.title;
                story.images = news.data.images;
                story.url = news.data.sourceUrl === undefined ? news.data.url : news.data.sourceUrl;

                stories.push(story);
              }
            });

            // console.log('ids=' + ids);
            // console.log('stories=' + JSON.stringify(stories));
          }
          // let ids = stories.map(story => story.id);

          this.$store.dispatch('addNews', {
            stories: stories,
            ids: ids
          })
        })
        .catch(error => {
          console.log(error)
        })
    },
    // 刷新数据
    refreshData () {
      // 刷新数据
      this.$store.dispatch('refreshNews')

      this.$nextTick(() => {
        this.fetchData()
      })
    },
    // 初始化当前日期
    initDate: function () {
      this.date = new Date()
      this.changeDate2String()
    },
    // 将Date类型的日期转换成String类型
    changeDate2String: function () {
      let year = this.date.getFullYear()
      let month = this.date.getMonth() + 1
      let day = this.date.getDate()
      month = month < 10 ? '0' + month : month // 格式化月份，小于10前置0
      day = day < 10 ? '0' + day : day // 格式化日期，小于10前置0;

      this.dateStr = year + month + day
    },
    // 将日期前推一天
    decreaseDate: function () {
      this.date.setDate(this.date.getDate() - 1)
      this.changeDate2String()
    },
    // 获取更多新闻数据
    fetchMoreData: function () {
      // TODO 代入新闻API URL
      axios.post('新闻API')
        .then(response => {
          /*
          // 合并数据
          let stories = response.data.stories
          let ids = stories.map(story => story.id)

          this.$store.dispatch('addNews', {
            stories: stories,
            ids: ids
          })
          */
          // 初始化新闻内容和id数组，并添加进state
          let newslist = response.data.data.result;
          // console.log(JSON.stringify(newslist));

          let stories = [];
          let ids = [];
          if (newslist != null) {
            newslist.forEach(news => {
              // console.log('NEWS=>'+JSON.stringify(news));
              // console.log('NEWS.type=>' + news.type);
              // console.log('NEWS.id=>' + news.data.id);
              // console.log('NEWS.title=>' + news.data.title);
              // console.log('NEWS.url=>' + news.data.sourceUrl);

              if (news.type === 'news') {
                ids.push(news.data.id);

                let story = {};
                story.id = news.data.id;
                story.title = news.data.title;
                story.images = news.data.images;
                story.url = news.data.sourceUrl === undefined ? news.data.url : news.data.sourceUrl;

                stories.push(story);
              }
            });

            // console.log('ids=' + ids);
            // console.log('stories=' + JSON.stringify(stories));
          }
          // let ids = stories.map(story => story.id);
          this.$store.dispatch('addNews', {
            stories: stories,
            ids: ids
          })

          // 合并完数据关闭加载提示框
          Indicator.close()
        })
        .catch(error => {
          console.log(error)
        })

      this.decreaseDate()
    },
    // 加载更多新闻数据
    loadMore: function () {
      this.loading = true

      // 打开加载提示框
      Indicator.open({
        spinnerType: 'snake'
      })

      // 加载更多数据并更新DOM
      this.$nextTick(function () {
        this.fetchMoreData()
      })

      this.loading = false
    }
  }
}
</script>

<style lang="sass" rel="stylesheet/sass" scoped>
    @import "../assets/sass/components/NewsList.sass";
</style>
