<template>
  <section class="article layout-content container">
    <div class="article-lay row">
      <div class="col-xs-12 col-sm-8--4 col-md-8--4">
        <main class="main client-card">
          <h1 class="article-title">{{ article.title }}</h1>

          <div class="article-view"
               v-if="article.aid">
            <div class="article-user-info">
              <div class="author">
                <router-link :to="{
                    name: 'user',
                    params: { uid: article.user.uid, routeType: 'article' }
                  }"
                             class="avatar">
                  <img v-lazy="article.user.avatar"
                       alt />
                </router-link>
                <div class="info">
                  <div class="name">
                    <router-link :to="{
                        name: 'user',
                        params: { uid: article.user.uid, routeType: 'article' }
                      }">{{ article.user.nickname }}</router-link>
                  </div>
                  <!-- 文章数据信息 -->
                  <div class="meta">
                    <span class="publish-time">{{ article.create_dt }}</span>
                    <span class="views-count">阅读 {{ article.read_count }}</span>
                    <span class="comments-count">评论 {{ article.comment_count }}</span>
                    <span class="likes-count">点赞 {{ article.thumb_count }}</span>
                    <em class="source">{{ sourceTypeList[article.source] }}</em>
                    <em class="type"
                        :class="`type${article.type}`">{{
                      articleTypeText[article.type]
                    }}</em>
                  </div>
                </div>
              </div>
            </div>

            <div class="article-content"
                 v-html="article.content"></div>

            <div class="attachment"
                 v-if="article.is_attachment === isOpen.yes">
              <div class="title">附件</div>
              <div class="attachment"
                   v-if="articleAnnex.attachment && personalInfo.islogin"
                   v-html="articleAnnex.attachment"></div>

              <div class="price-info"
                   v-if="
                  articleAnnex.is_free === isFree.pay && !articleAnnex.isBuy
                ">
                <div class="price-info-view">
                  此内容需要
                  <em class="login"
                      @click="$router.push({ name: 'signIn' })"
                      v-if="!personalInfo.islogin">登录</em>
                  <em class="buy"
                      @click="onBuy">支付</em>
                  {{ articleAnnex.price }}贝壳，才可继续查看
                </div>
              </div>

              <div class="price-info"
                   v-else-if="
                  articleAnnex.is_free === isFree.free && !personalInfo.islogin
                ">
                <div class="price-info-view">
                  此内容需要
                  <em class="login"
                      @click="$router.push({ name: 'signIn' })"
                      v-if="!personalInfo.islogin">登录</em>
                  ，才可继续查看
                </div>
              </div>
            </div>

            <ul class="tag-body">
              <li v-for="(itemTag, key) in article.tag"
                  :key="key">
                <router-link class="tag-class frontend"
                             :to="{
                    name: 'article_tag',
                    params: { en_name: itemTag.en_name }
                  }">
                  <div class="img"
                       :alt="itemTag.name"
                       :style="`background-image: url(${itemTag.icon})`"></div>
                  {{ itemTag.name }}
                </router-link>
              </li>
            </ul>

            <div class="meta-bottom clearfix">
              <div class="meta-bottom-item like"
                   @click="onUserThumbArticle"
                   :class="{ active: isThumb(article) }">
                <i :class="isThumb(article) ? 'el-icon-thumb' : 'el-icon-thumb'"></i>
              </div>
              <div class="meta-bottom-item share">
                <Dropdown>
                  <div class="el-dropdown-link"
                       slot="button">
                    <i class="el-icon-share"></i>
                  </div>
                  <div class="dropdown-menu-view">
                    <div class="dropdown-menu-item"
                         @click="shareChange({ type: 'qq', data: article })">
                      分享到QQ
                    </div>
                    <div class="dropdown-menu-item"
                         @click="shareChange({ type: 'sina', data: article })">
                      分享到新浪
                    </div>
                    <div class="dropdown-menu-item"
                         @click="shareChange({ type: 'qzone', data: article })">
                      分享到QQ空间
                    </div>
                  </div>
                </Dropdown>
              </div>
            </div>
            <!--article footer end-->
            <!--文章评论-->
            <ArticleComment />
          </div>
          <p class="no-aricle"
             v-else>文章不存在</p>
        </main>
      </div>
      <div class="col-xs-12 col-sm-3--6 col-md-3--6 aside">
        <ArticleAside />
      </div>
    </div>

    <Dialog :visible.sync="isBuyDialog"
            :close-on-click-modal="false"
            :close-on-press-escape="false"
            width="380px">
      <div class="buy-view"
           v-loading="isBuyLoading">
        <h3 class="title">购买信息确认</h3>
        <ul>
          <li class="p-name">商品名称：<em>文章附件</em></li>
          <li class="p-pay-type">
            支付方式：<em>{{ payTypeText[articleAnnex.pay_type] }}</em>
          </li>
          <li class="p-pay-price">
            价格：<em>￥{{ articleAnnex.price }}</em>
          </li>
        </ul>
        <div class="footer-view">
          <button class="btn btn-buy"
                  @click="enterBuy">确认购买</button>
          <button class="btn btn-cancel"
                  @click="isBuyDialog = false">
            取消
          </button>
        </div>
      </div>
    </Dialog>
  </section>
  <!--home-lay layout-content end-->
</template>

<script>
import ArticleComment from '@views/Comment/ArticleComment'
import ArticleAside from '@views/Article/component/ArticleAside'
import { share, baidu, google } from '@utils'
import { mapState } from 'vuex'
import googleMixin from '@mixins/google'
import { Dropdown, Dialog } from '@components'
// 在这里导入模块，而不是在 `store/index.js` 中
import shopModule from '../../store/module/shop'
import hljs from '@utils/highlight.pack'

import {
  statusList,
  articleType,
  statusListText,
  articleTypeText,
  modelName,
  payTypeText,
  isFree,
  isOpen
} from '@utils/constant'
export default {
  name: 'Article',
  mixins: [googleMixin], //混合谷歌分析
  metaInfo () {
    return {
      title: this.article.title || '',
      titleTemplate: `%s - ${this.website.meta.website_name || ''}`,
      meta: [
        {
          // set meta
          name: 'description',
          content: `${this.article.excerpt || ''}`
        },
        {
          // og:site_name
          property: 'og:site_name',
          content: this.website.meta.website_name
        },
        {
          // og:site_name
          property: 'og:image',
          content: this.article.cover_img || this.website.meta.logo
        },
        {
          // og:type
          property: 'og:type',
          content: `article`
        },
        {
          // og:title
          property: 'og:title',
          content: this.article.title
        },
        {
          // og:description
          property: 'og:description',
          content: this.article.excerpt
        },
        {
          // og:url
          property: 'og:url',
          content: `${this.website.meta.domain_name}/p/${this.article.aid}`
        }
      ],
      htmlAttrs: {
        lang: 'zh'
      },
      script: [
        ...baidu.resource({
          route: this.$route,
          config: this.website.config,
          random: this.article.aid
        }),
        ...google.statisticsCode({
          route: this.$route,
          googleCode: this.website.config.googleCode,
          random: this.$route.params.blogId
        })
      ],
      __dangerouslyDisableSanitizers: ['script']
    }
  },
  asyncData ({ store, route, accessToken = '' }) {
    // 触发 action 后，会返回 Promise
    return Promise.all([
      store.dispatch('article/GET_ARTICLE', {
        aid: route.params.aid,
        accessToken
      })
    ])
  },
  data () {
    return {
      sourceTypeList: ['', '原创', '转载'],
      articleTypeText,
      payTypeText,
      isFree,
      isOpen,
      modelName,
      isBuyDialog: false,
      isBuyLoading: false,
      articleAnnex: {} // 文章附件信息
    }
  },
  mounted () {
    this.$store.registerModule('shop', shopModule)
    if (this.article.is_attachment == this.isOpen.yes) {
      this.getArticleAnnex()
    }
    this.initHljs()
  },
  methods: {
    initHljs () {
      let blocks = document.querySelectorAll('pre code')
      blocks.forEach(block => {
        hljs.highlightBlock(block)
      })
    },
    isThumb (item) {
      // 是否收藏
      if (this.personalInfo.islogin) {
        if (
          this.user.associateInfo.articleThumdId &&
          ~this.user.associateInfo.articleThumdId.indexOf(item.aid)
        ) {
          return true
        } else {
          return false
        }
      } else {
        return false
      }
    },
    onBuy () {
      //
      if (!this.personalInfo.islogin) {
        this.$message.warning('请先登录，再继续操作')
        this.$router.push({ name: 'signIn' })
        return false
      }
      this.isBuyDialog = true
    },
    enterBuy () {
      if (!this.personalInfo.islogin) {
        this.$message.warning('请先登录，再继续操作')
        return false
      }
      this.isBuyLoading = true
      this.$store
        .dispatch('shop/BUY', {
          product_id: this.articleAnnex.id,
          product_type: this.modelName.article_annex
        })
        .then(result => {
          this.isBuyLoading = false
          if (result.state === 'success') {
            this.isBuyDialog = false
            if (this.article.is_attachment == this.isOpen.yes) {
              this.getArticleAnnex()
            }
            this.$message.success(result.message)
          } else {
            this.$message.warning(result.message)
          }
        })
    },
    onUserThumbArticle () {
      /*用户like 文章*/
      this.$store
        .dispatch('common/SET_THUMB', {
          associate_id: this.article.aid,
          type: modelName.article
        })
        .then(result => {
          if (result.state === 'success') {
            this.$store.dispatch('user/GET_ASSOCIATE_INFO')
          } else {
            this.$message.warning(result.message)
          }
        })
        .catch(err => {
          console.log(err)
        })
    },
    getArticleAnnex () {
      this.$store
        .dispatch('article/GET_ARTICLE_ANNEX', {
          aid: this.article.aid
        })
        .then(result => {
          if (result.state === 'success') {
            this.articleAnnex = result.data.articleAnnex
          } else {
            this.$message.warning(result.message)
          }
        })
        .catch(err => {
          console.log(err)
        })
    },
    shareChange (val) {
      // 分享到其他
      let urlOrigin = window.location.origin // 源地址
      if (val.type === 'sina') {
        // 新浪
        share.shareToXl(
          val.data.title,
          urlOrigin + '/p/' + val.data.aid,
          this.website.meta.logo
        )
      } else if (val.type === 'qzone') {
        // qq空间
        share.shareToQq(
          val.data.title,
          urlOrigin + '/p/' + val.data.aid,
          this.website.meta.logo
        )
      } else if (val.type === 'qq') {
        // qq空间
        share.shareQQ(
          val.data.title,
          urlOrigin + '/p/' + val.data.aid,
          this.website.meta.logo
        )
      }
    }
  },
  computed: {
    article () {
      return this.$store.state.article.article || {}
    },
    ...mapState(['website', 'personalInfo', 'user'])
  },
  components: {
    ArticleComment,
    Dropdown,
    Dialog,
    ArticleAside
  },
  destroyed () {
    this.$store.unregisterModule('shop')
  }
}
</script>

<style scoped lang="scss">
.article.layout-content {
  margin-bottom: 12px;
  .main {
    width: 100%;
    flex: 1;
    padding: 15px 25px;
    .article-title {
      text-align: left;
      max-width: 100%;
      margin-top: 15px;
      position: static;
      color: #48494d;
      font-size: 28px;
      font-weight: 700;
      line-height: 1.3;
    }
    .article-view {
      .article-user-info {
        padding: 30px 0 15px;
        .author {
          .avatar {
            width: 48px;
            height: 48px;
            vertical-align: middle;
            display: inline-block;
            img {
              width: 100%;
              height: 100%;
              border: 1px solid #ddd;
              border-radius: 50%;
            }
          }
          .info {
            vertical-align: middle;
            display: inline-block;
            margin-left: 8px;
            .name {
              margin-right: 3px;
              font-size: 16px;
              vertical-align: middle;
            }
            .follow {
              padding: 0 7px 0 5px;
              font-size: 12px;
              border-color: #42c02e;
              border-radius: 40px;
              color: #fff;
              background-color: #42c02e;
              line-height: 1;
              &.active {
                background: #999999;
                border-color: #999999;
              }
            }
            .meta {
              font-size: 12px;
              color: #969696;
              span + span::before {
                display: inline;
                content: " \B7 ";
                color: #9b9b9b;
                padding-left: 2px;
                padding-right: 2px;
              }
              .source {
                background: #ea6f5a;
                padding: 1px 5px;
                border-radius: 3px;
                color: #fff;
                margin-left: 15px;
                margin-right: 5px;
              }
              .type {
                padding: 2px 5px;
                border-radius: 5px;
                @for $i from 1 through 5 {
                  &.type#{$i} {
                    @if $i==1 {
                      background-color: rgb(253, 239, 207);
                      color: rgb(255, 174, 14);
                    } @else if $i==2 {
                      background-color: rgb(245, 243, 231);
                      color: rgb(135, 125, 106);
                    } @else if $i==3 {
                      background-color: rgb(231, 243, 240);
                      color: rgb(145, 203, 186);
                    } @else if $i==4 {
                      background-color: rgb(135, 198, 179);
                      color: rgb(255, 255, 255);
                    } @else if $i==5 {
                      background-color: rgb(245, 243, 231);
                      color: rgb(135, 125, 106);
                    }
                  }
                }
              }
            }
          }
        }
      }
      .support-author {
        min-height: 144px;
        padding: 20px 0;
        text-align: center;
        clear: both;
        p {
          padding: 0 30px;
          margin-bottom: 20px;
          min-height: 24px;
          font-size: 17px;
          font-weight: 700;
          color: #969696;
        }
        .btn-pay {
          margin-bottom: 20px;
          padding: 8px 25px;
          font-size: 16px;
          color: #fff;
          background-color: #ea6f5a;
          border-radius: 20px;
        }
      }
      .attachment {
        background: #f2f2f2;
        padding: 10px;
        margin-bottom: 15px;
        .title {
          font-size: 15px;
        }
        .price-info {
          font-size: 15px;
          .buy,
          .login {
            cursor: pointer;
          }
          .buy {
            color: #ea6f5a;
          }
          .login {
            color: #42c02e;
          }
        }
      }
      .tag-body {
        display: -webkit-box;
        display: flex;
        li {
          position: relative;
          list-style: none;
          margin: 0 15px 15px 0;
          font-size: 12px;
          > a {
            color: var(--layer-color);
            padding: 2px 5px;
            border: 1px solid transparent;
            position: relative;
            white-space: nowrap;
            word-wrap: normal;
            display: block;
            background-color: var(--light);
            border-radius: 3px 3px 3px 3px;
            line-height: 21px;
            .img {
              border-radius: 2px 2px 2px 2px;
              height: 16px;
              width: 16px;
              float: left;
              margin: 2px 3px 0 0;
              background-color: rgba(0, 0, 0, 0.02);
              background-size: cover;
              background-repeat: no-repeat;
              background-position: 50%;
            }
          }
        }
      }

      .meta-bottom {
        margin-top: 20px;
        .meta-bottom-item {
          display: inline-block;
          width: 38px;
          height: 38px;
          line-height: 38px;
          border: 1px solid #e0e0e0;
          text-align: center;
          margin: 0 8px;
          cursor: pointer;
          border-radius: 20px;
          i {
            font-size: 16px;
            color: #333;
          }
          &.active {
            border: 1px solid #e67e7e;
            i {
              color: #e67e7e;
            }
          }
        }
      }
      .share-group {
        float: right;
        margin-top: 6px;
        .share-circle {
          width: 50px;
          height: 50px;
          margin-left: 5px;
          text-align: center;
          border: 1px solid #dcdcdc;
          border-radius: 50%;
          vertical-align: middle;
          display: inline-block;
          position: relative;
        }
        .more-share {
          width: auto;
          padding: 4px 18px;
          font-size: 14px;
          color: #9b9b9b;
          line-height: 40px;
          border-radius: 50px;
        }
      }
    }
  }
  .no-aricle {
    width: 100%;
    padding: 30px;
    border-radius: 5px;
    background: #ea6f5a;
    color: #fff;
    text-align: center;
    font-size: 25px;
    margin: 20px 0;
  }
}

@media (max-width: 575px) {
  .article.layout-content {
    .main {
      padding: 10px;
    }
  }
}
</style>
