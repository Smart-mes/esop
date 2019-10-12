<template>
  <div id="app" class="container">
    <!--幻灯片-->
    <div class="swiper-container">
      <div class="swiper-wrapper">
        <div v-for="(item, i) in bannerList" :key="i" class="swiper-slide">
          <img :src="`${imgUrl}/${item.url}`">
        </div>
      </div>
      <div class="swiper-pagination"/>
      <div class="swiper-button-prev"/>
      <div class="swiper-button-next"/>
    </div>
    <!-- 下载链接 -->
    <div class="link-box">
      <dl class="link">
        <dt>下载链接:</dt>
        <dd v-for="(item, i) in linkList" :key="i">
          <a :href="item.url" target="_blank">{{ item.fileName }}</a>
        </dd>
      </dl>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
import Swiper from 'swiper'

export default {
  name: 'App',
  data () {
    return {
      configureUrl: 'http://192.168.1.121:8012',
      imgUrl: 'http://192.168.1.121:8020',
      bannerList: [],
      linkList: [],
    }
  },
  mounted () {
    this.getData()
      .then(() => {
        this.swiperInit()
        this.loading = false
      }).catch(error => {
        console.log(error)
      })
  },
  methods: {
    // 幻灯片初始化
    swiperInit () {
      new Swiper('.swiper-container', {
        pagination: {
          el: '.swiper-pagination',
          clickable: true,
          renderBullet (index, className) {
            return '<span class="' + className + '">' + (index + 1) + '</span>'
          },
        },
        navigation: {
          nextEl: '.swiper-button-next',
          prevEl: '.swiper-button-prev',
        },
        autoplay: false,

      })
    },
    // 获取数据
    getData () {
      const match = /pid=(\d+)/.exec(location.search)
      const pid = match && match[1]
      if (!pid) {
        alert('缺少Pid参数！')
        return Promise.reject(new Error('缺少Pid参数！'))
      }
      return axios.get(`${this.configureUrl}/api/BESop`, { pid })
        .then(res => res.data)
        .then(data => {
          if (!data.length) {
            alert('该工艺没有挂载文件！')
          }
          data.map(item => {
            if (item.type === 1) {
              this.bannerList.push(item)
            } else if (item.type === 2) {
              this.linkList.push(item)
            } else {
              item.url = `${this.imgUrl}/${item.url}`
              this.linkList.push(item)
            }
          })
        })
    },
  },
}
</script>

<style lang="scss">
html,
body {
  height: 100%;
}

body {
  background: #eee;
  font-family: Microsoft YaHei;
  font-size: 14px;
  color: #000;
  margin: 0;
  padding: 0;
}

dl,
dt,
dd {
  margin: 0;
  padding: 0;
}

#app.container {
  height: 100%;
  overflow: hidden;
  .swiper-container {
    height: 90%;
    .swiper-slide {
      background: #fff;
      img {
        vertical-align: bottom;
      }
    }
  }
  .swiper-pagination-bullet {
    width: 20px;
    height: 20px;
    font-size: 12px;
    text-align: center;
    line-height: 20px;
    color: #000;
    opacity: 1;
    background: rgba(0, 0, 0, 0.2);
    &-active {
      color: #fff;
      background: #007aff;
    }
  }
  /*下载链接*/
  .link-box {
    height: 10%;
    display: flex;
    align-items: center;
  }

  .link {
    overflow: hidden;
    padding: 0 20px;
    height: 44px;
    dt {
      margin-bottom: 5px;
      font-size: 15px;
      font-weight: bold;
    }

    dd {
      display: inline-block;
      padding-right: 15px;
      &:before {
				content: '·';
			}
    }

    dd a,
    dd a:link {
      padding-left: 3px;
      color: #0000ee;
      text-decoration: underline;
    }
  }
}
</style>
