<template>
  <div class="content">

    <div
      class="post-list"
      v-infinite-scroll="loadData"
      infinite-scroll-disabled="isLoading"
      infinite-scroll-distance="10">

        <el-card
          class="post"
          v-for="post in postList"
          v-bind:key="post.id"
          v-if="getImage(post)"
          @click.native="openPost(post)">
            <div class="post__image"><img :src="getImage(post)" /></div>
            <div class="post__content">
              <p class="post__content__title">{{post.root_title}}</p>

              <div class="post__content__footer">
                <p>by @{{post.author}}</p>
                <div>
                  <span> <i class="fas fa-comments"></i> {{post.children}}</span>
                  <span> <i class="fas fa-heart"></i> {{post.net_votes}}</span>
                  <span> <i class="material-icons">monetization_on</i> {{getPayout(post)}}</span>
                </div>
              </div>
            </div>
        </el-card>

        <div
        v-loading="isLoading"
        style="width: 100%; height: 50px;"
        ></div>

    </div>

    <div v-if="currentPost">
      <el-dialog
        :title="currentPost.root_title"
        :visible.sync="dialogVisible"
        custom-class="post-details"
        :before-close="handleCloseDialog"
        top="5vh">
          <div class="post-details__header">
            <a :href="`https://steemit.com/@${currentPost.author}`" class="profile-link" target="_blank">
              by @{{currentPost.author}}
            </a>
            <div>
              <span> <i class="fas fa-comments"></i> {{currentPost.children}}</span>
              <span> <i class="fas fa-heart"></i> {{currentPost.net_votes}}</span>
              <span> <i class="material-icons">monetization_on</i> {{getPayout(currentPost)}}</span>
            </div>
          </div>

          <div class="post-details__image">
            <img :src="getImage(currentPost)" />
          </div>

          <div class="post-details__footer">
            <a :href="getCurrentPostLink" class="steem-link" target="_blank">
              <el-card>
                <i class="fas fa-info-circle"></i> Steemit Post
              </el-card>
            </a>
          </div>

      </el-dialog>
    </div>


    <!--<div class="post-details" style="
        background-color: #fff;
        position: fixed;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        background-color: rgba(0,0,0,.6);"></div> -->

  </div>
</template>

<script>

import { Card, Dialog } from 'element-ui';
import steem from 'steem';

steem.api.setOptions({ url: 'https://api.steemit.com' });

export default {
  name: 'Homepage',
  components: {
    'el-card': Card,
    'el-dialog': Dialog,
  },
  data() {
    return {
      isLoading: false,
      postList: [],
      dialogVisible: false,
      currentPost: null,
      lastMetadata: { author: null, permlink: null },
    };
  },
  methods: {
    getImage(post) {
      const postMetadata = JSON.parse(post.json_metadata);
      if (!postMetadata.image) {
        return false;
      }
      const image = postMetadata.image[0];
      return image;
    },

    openPost(post) {
      if (!this.currentPost) {
        this.currentPost = post;
        this.dialogVisible = true;
      }
    },

    handleCloseDialog(done) {
      this.currentPost = null;
      done();
    },

    getPayout(post) {
      return Number.parseInt(post.pending_payout_value.replace(' SBD', ''));
    },

    loadData() {
      this.isLoading = true;

      const query = {
        tag: 'photography',
        limit: 30,
        start_author: this.lastMetadata.author,
        start_permlink: this.lastMetadata.permlink,
      };

      // steem.api.getState('/trending/photography', (err, result) => {
      steem.api.getDiscussionsByTrending(query, (err, result) => {
        const newPostList = this.lastMetadata.author ? result.slice(1) : result;
        this.postList = this.postList.concat(newPostList);

        this.lastMetadata.permlink = result[result.length - 1].permlink;
        this.lastMetadata.author = result[result.length - 1].author;

        this.isLoading = false;
      });
    }, // loadData
  },
  computed: {
    getCurrentPostLink() {
      const link = `${this.currentPost.category}/@${this.currentPost.author}/${this.currentPost.permlink}`;
      return `https://steemit.com/${link}`;
    },
  },
  mounted() {
    this.loadData();
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>

.content{
  max-width: 1400px;
  margin: auto;
}

.post-list{
  margin-top: 30px;
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
}

.post-list .el-loading-mask{
  background: 0;
  margin-top: 50px;
}

.post-list > .post{
  width: 400px;
  margin-right: 20px;
  margin-top: 20px;
}

.post-list > .post .el-card__body{
  padding: 0;
  position: relative;
}

.post-list > .post .post__image{
  height: 320px;
}

.post-list > .post .post__image img{
  width: 100%;
  height: 100%;
}

.post-list > .post:hover .post__content{
  opacity: 1;
  cursor: pointer;
}

.post-list > .post .post__content{
  transition: .2s;
  opacity: 0;

  position: absolute;
  padding: 10px;
  width: 100%;
  height: 100%;
  left: 0;
  top: 0;
  box-sizing: border-box;

  /*background: rgba(255, 255, 255, 0.6);*/

  /*
  background-image: linear-gradient(180deg,rgba(255,255,255,.5) 0,
                    transparent 40%,transparent 60%,rgba(255,255,255,.7));
  background-image: linear-gradient(180deg,rgba(0,0,0,.2) 0,
                    transparent 40%,transparent 60%,rgba(0,0,0,.3));
  background-image: linear-gradient(180deg,rgba(0,0,0,.5) 0,
                    transparent 20%,transparent 80%,rgba(0,0,0,.5));
  */

  background-image: linear-gradient(180deg,rgba(0,0,0,.3) 0,
                    transparent 30%,transparent 70%,rgba(0,0,0,.4));

  color: #fff;
}

.post-list > .post .post__content__title{
  font-weight: 600;
  color: rgba(255,255,255,0.95);
}

.post-list > .post .post__content__footer{
  position: absolute;
  padding: 10px;
  box-sizing: border-box;
  bottom: 0;
  left: 0;
  font-weight: 600;
  display: flex;
  justify-content: space-between;
  width: 100%;
}

.post-list > .post .post__content__footer > div{
  font-size: 14px;
  color: rgba(255,255,255,0.9); /*#eee*/
}

.post-list > .post .post__content__footer > div span,
.post-details .post-details__header > div span{
  margin-right: 10px;
}

/*//////////////////////////////////////////////////////*/
/*//////////////////////////////////////////////////////*/

.post-details{
  width: 80%;
  min-width: 320px;
  max-width: 1100px;
}

.post-details .el-dialog__header{
  padding-bottom: 0;
}

.post-details .el-dialog__body{
  padding: 0 15px 10px;
}

.post-details .el-dialog__body .post-details__header{
  font-weight: 600;
  display: flex;
  justify-content: space-between;
  margin-bottom: 20px;
}

.post-details .post-details__header a.profile-link:hover{
  opacity: 0.5;
}

.post-details .post-details__image{
  text-align: center;
}

.post-details .post-details__image img{
  max-height: 80vh;
  max-width: 100%;
}

.post-details .post-details__footer{
  margin-top: 10px;
}

.post-details .post-details__footer .steem-link{
  /*text-align: right;*/
  text-align: right;
  display: block;
}

.post-details .post-details__footer .steem-link i{
  color: #777;
}

.post-details .post-details__footer .steem-link .el-card{
  display: inline-block;
  border: 1px solid #ccc;
}

.post-details .post-details__footer .steem-link .el-card .el-card__body{
  /*padding: 8px 12px;*/
  padding: 5px 10px;
}

</style>
