<template>
  <div class="me-view-body" v-title :data-title="title">
    <el-container class="me-view-container">
      <!--<el-aside class="me-area">-->
        <!--<ul class="me-operation-list">-->
          <!--<li class="me-operation-item">-->
            <!--<el-button type="primary" icon="el-icon-edit"></el-button>-->
          <!--</li>-->
        <!--</ul>-->
      <!--</el-aside>-->
      <el-main>

        <div class="me-view-card">
          <h1 class="me-view-title">{{article.title}}</h1>
          <zan></zan>
          <div class="me-view-author">
            <a class="">
              <!-- <img class="me-view-picture" :src="article.author.avatar"></img> -->
            </a>
            <div class="me-view-info">
              <!-- <span>{{nickName}}</span>
               -->
               <div id="nameAndGender" >
                 <a>
							<div style="height: 30px;line-height: 30px;float: left;font-size: 20px;" @click="gohishome">
								{{userName}}
							</div>
              </a>
							<!-- <div v-if="userGender == 'male'" style="float: left;"><img src="../../assets/male.png" style="height: 20px;margin-top: 5px;margin-left: 5px;"/></div> -->
							<!-- <div v-else style="float: left;height: 40px;"><img src="../../assets/female.png" style="height: 20px;margin-top: 5px;margin-left: 5px;"/></div> -->
							
							
						</div>
              <div class="userSpaceUserInfos" style="border-left: rgb(158, 228, 50) 1px solid;padding-left: 5px;">
							<div v-for="u in userInfos" style="height: 30px;float: left;border-right: rgb(158, 228, 50) 1px solid;margin-right: 5px;padding-right: 5px;">
								<div style="height: 15px;font-size:15px;">{{u.partNum}}</div>
								<div style="height: 8px;margin-top: 2px;font-size: 10px;">{{u.partName}}</div>
							</div>
						</div>
              <div class="me-view-meta">
                <span>{{article.publishDate | format}}</span>
                <span>阅读   {{article.viewNumber}}</span>
                <span>评论   {{article.commentCount}}</span>
              </div>

            </div>
            <el-button
              v-if="this.article.userId == this.$store.state.id"
              @click="editArticle()"
              style="position: absolute;left: 60%;"
              size="mini"
              round
              icon="el-icon-edit">编辑</el-button>
              <el-button
              v-if="this.article.userId == this.$store.state.id"
              @click="deleteArticle()"
              style="position: absolute;left: 66%;"
              size="mini"
              round
              icon="el-icon-delete">删除</el-button>
          </div>
          <div class="me-view-content">
            <markdown-editor :editor=article.editor></markdown-editor>
          </div>

          <div class="me-view-end">
            <el-alert
              title="文章End..."
              type="success"
              center
              :closable="false">
            </el-alert>
          </div>

          <div class="me-view-tag">
            标签：
            <!-- <el-tag v-for="t in article.tagInfos" :key="t.tagId" class="me-view-tag-item" size="mini" type="success">{{t.tagName}}</el-tag> -->
            <el-button v-for="t in article.tagInfos" @click="tagOrCategory('tag', t.tagId)" size="mini" type="primary" :key="t.tagId" round plain>{{t.tagName}}</el-button>
          </div>

          <!-- <div class="me-view-tag">
            文章分类：
            <span style="font-weight: 600">{{article.category.categoryname}}</span>
            <el-button @click="tagOrCategory('category', article.category.id)" size="mini" type="primary" round plain>{{article.category.categoryname}}</el-button>
          </div> -->

          <div class="me-view-comment">
            <div class="me-view-comment-write">
              <el-row :gutter="20">
                <el-col :span="2">
                  <a class="">
                    <img class="me-view-picture" :src="avatar"></img>
                  </a>
                </el-col>
                <el-col :span="22">
                  <el-input
                    type="textarea"
                    :autosize="{ minRows: 2}"
                    placeholder="你的评论..."
                    class="me-view-comment-text"
                    v-model="comment.content"
                    resize="none">
                  </el-input>
                </el-col>
              </el-row>

              <el-row :gutter="20">
                <el-col :span="2" :offset="22">
                  <el-button type="text" @click="publishComment()">评论</el-button>
                </el-col>
              </el-row>
            </div>

            <div class="me-view-comment-title">
              <span>{{article.commentCount}} 条评论</span>
            </div>

            <commment-item
              v-for="(c,index) in comments"
              :comment="c"
              :articleId="article.articleId"
              :index="index"
              :rootCommentCounts="comments.length"
              @commentCountIncrement="commentCountIncrement"
              @commentCountDecrement="commentCountDecrement"
              @getCommentsByArticle="getCommentsByArticle"
              :key="c.id">
            </commment-item>

          </div>

        </div>
      </el-main>

    </el-container>
  </div>
</template>

<script>
  import MarkdownEditor from '@/components/markdown/MarkdownEditor'
  import CommmentItem from '@/components/comment/CommentItem'
  import {viewArticle,deleteArticleById} from '@/api/article'
  import {getCommentsByArticle, publishComment} from '@/api/comment'
  import {getToken} from '@/request/token'
  import {getUserById,favor,unfavor,checkfavor} from '@/api/login'
  import default_avatar from '@/assets/img/default_avatar.png'
  import zan from '../../components/blogComponents/zan_article.vue'

  export default {
    name: 'BlogView',
    created() {
      this.getArticle()
    },
    watch: {
      '$route': 'getArticle'
    },
    data() {
      return {
        author_info:{
	            nickname:"Eddie Peng",
              avatarUrl:"static/avatar1.png",
              articleCount:13,
              userFollowerCount:10,
              userFollowingCount:8,
          },
        userName: "hhd",
				  userGender: "male",
				  userIntro: "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
			  	userInfos:[
					{partName:"关注",partNum:0},
					{partName:"粉丝",partNum:1},
					{partName:"文章",partNum:2},
				],
				nickName:"nmsl", 
        article: {
          articleId: '',
          title: '',
          commentCount: 0,
          viewNumber: 0,
          summary: '',
          userId:'',
          tagInfos: [],
          category:{},
          publishDate: '',
          editor: {
            value: '',
            toolbarsFlag: false,
            subfield: false,
            defaultOpen: 'preview'
          }
        },
        comments: [],
        comment: {
          article: {},
          content: ''
        }
      }
    },
    computed: {
      avatar() {
        let avatar = this.$store.state.avatar

        if (avatar.length > 0) {
          return avatar
        }
        return default_avatar
      },
      title() {
        return `${this.article.title} - 文章 - `
      }
    },
    methods: {
      userSpaceFollow() {

			},
      tagOrCategory(type, id) {
        this.$router.push({path: `/${type}/${id}`})
      },
      editArticle() {
        this.$router.push({path: `/write/${this.article.articleId}`})
      },
      getArticle() {
        let that = this
        // alert(that.$route.params.id)
        viewArticle(that.$route.params.id).then(data => {
          // alert("ok");
          
          Object.assign(that.article, data.data.data)
          
          // alert(JSON.stringify(data.data.data.tagInfos))
          that.article.editor.value = data.data.data.content
          
          that.getCommentsByArticle()
          
          that.getUser()
        }).catch(error => {
          // alert("bad");
          if (error !== 'error') {
            that.$message({type: 'error', message: '文章加载失败', showClose: true})
          }
        })
      },
      getUser() {
				let that = this
        // alert(that.$route.params.id)
        getUserById(that.article.userId).then(data => {
          // alert("ok");
          Object.assign(that.author_info, data.data.data)
					// alert(JSON.stringify(data.data.data.tagInfos))
					if (!this.author_info.avatarUrl || typeof(this.author_info.avatarUrl)=="undefined") 
					{ 
					this.author_info.avatarUrl="static/avatar1.png"
          }　
          this.userInfos[0].partNum=this.author_info.userFollowingCount
          this.userInfos[1].partNum=this.author_info.userFollowerCount
          this.userInfos[2].partNum=this.author_info.articleCount
          // alert()
          that.userName=that.author_info.nickname
					// alert(this.avatarUrl)
        }).catch(error => {
          // alert("bad");
          if (error !== 'error') {
            that.$message({type: 'error', message: '用户加载失败', showClose: true})
          }
        })
			},
      deleteArticle(){
        
        this.$confirm('文章将会从地球上消失，是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          // alert("ok")
          let loading = this.$loading({
              lock: true,
              text: ' 删除中，请稍后...'
            })
          deleteArticleById(getToken(),this.article.articleId).then((data) => {
              loading.close();   
              this.$message({message: '删除成功啦', type: 'success', showClose: true})
              this.$router.push('/')

            }).catch((error) => {
              loading.close();
              if (error !== 'error') {
                that.$message({message: error, type: 'error', showClose: true});
              }
            })
          
        })
      },
      publishComment() {
        let that = this
        if (!that.comment.content) {
          return;
        }
        that.comment.article.articleId = that.article.articleId

        publishComment(that.article.articleId,that.comment.content,getToken()).then(data => {
          that.$message({type: 'success', message: '评论成功', showClose: true})
          // that.comments.unshift(data)
          //  getCommentsByArticle(that.article.articleId).then(data => {
          //   that.comments = data.data.data
          //   // alert(JSON.stringify(that.comments))
          // }).catch(error => {
          //   // alert("no")
          //   if (error !== 'error') {
          //     that.$message({type: 'error', message: '评论加载失败', showClose: true})
          //   }
          // })
          that.getCommentsByArticle()
          that.commentCountIncrement()
          that.comment.content = ''
        }).catch(error => {
          if (error !== 'error') {
            that.$message({type: 'error', message: '评论失败', showClose: true})
          }
        })
      },
      getCommentsByArticle() {
        // alert("in")
        let that = this
        getCommentsByArticle(that.article.articleId).then(data => {
          that.comments = data.data.data
          // alert(JSON.stringify(that.comments))
        }).catch(error => {
          // alert("no")
          if (error !== 'error') {
            that.$message({type: 'error', message: '评论加载失败', showClose: true})
          }
        })
      },
      commentCountIncrement() {
        this.article.commentCount += 1
      },
      commentCountDecrement() {
        this.article.commentCount -= 1
      },
      gohishome(){
        this.$router.push('/userspace/'+this.article.userId)
      }
    },
    components: {
      'markdown-editor': MarkdownEditor,
      CommmentItem,
      zan
    },
    //组件内的守卫 调整body的背景色
    beforeRouteEnter(to, from, next) {
      window.document.body.style.backgroundColor = '#fff';
      next();
    },
    beforeRouteLeave(to, from, next) {
      window.document.body.style.backgroundColor = '#f5f5f5';
      next();
    }
  }
</script>

<style>
  #nameAndGender:after {
		content: "";
		display: block;
		clear: both;
		
	}
  .userSpaceUserInfos:after {
		content: "";
		display: block;
		clear: both;
	}


	#userSpaceFollow {
		float: left;
		margin-top: 30px;
		margin-left: 10px;
		border: white 1px solid;
		border-radius: 5px;
		font-size: 15px;
		color: white;
	}

	#userSpaceFollow:hover {
		background-color: #00D1B2;
	}
  .me-view-body {
    margin: 100px auto 140px;
  }

  .me-view-container {
    width: 700px;
  }

  .el-main {
    overflow: hidden;
  }

  .me-view-title {
    font-size: 34px;
    font-weight: 700;
    line-height: 1.3;
  }

  .me-view-author {
    /*margin: 30px 0;*/
    margin-top: 30px;
    vertical-align: middle;
  }

  .me-view-picture {
    width: 40px;
    height: 40px;
    border: 1px solid #ddd;
    border-radius: 50%;
    vertical-align: middle;
    background-color: #5fb878;
  }

  .me-view-info {
    display: inline-block;
    vertical-align: middle;
    margin-left: 8px;
  }

  .me-view-meta {
    font-size: 12px;
    color: #969696;
  }

  .me-view-end {
    margin-top: 20px;
  }

  .me-view-tag {
    margin-top: 20px;
    padding-left: 6px;
    border-left: 4px solid #c5cac3;
  }

  .me-view-tag-item {
    margin: 0 4px;
  }

  .me-view-comment {
    margin-top: 60px;
  }

  .me-view-comment-title {
    font-weight: 600;
    border-bottom: 1px solid #f0f0f0;
    padding-bottom: 20px;
  }

  .me-view-comment-write {
    margin-top: 20px;
  }

  .me-view-comment-text {
    font-size: 16px;
  }

  .v-show-content {
    padding: 8px 25px 15px 0px !important;
  }

  .v-note-wrapper .v-note-panel {
    box-shadow: none !important;
  }

  .v-note-wrapper .v-note-panel .v-note-show .v-show-content, .v-note-wrapper .v-note-panel .v-note-show .v-show-content-html {
    background: #fff !important;
  }


</style>
