//*****************************************
// 创建人：泰多
// 创建时间：2019/1/6
// 功能说明：该组件主要显示当前章节发布次数列表

//***********

<template>
    <div class="v_chapterList">

        <div v-show="!isChapterDetail">
            <!-- 头部  BEGIN -->
            <pb-header :isRight="true">
                <span slot="left" @click="closeFn">返回</span>
                <span slot="title" v-text="'['+ chapterName +']发布列表'"></span>
                <span slot="right" @click="sortFn" v-text="sortNum ? '倒序'  : '正序'">正序</span>
            </pb-header>
            <!-- 头部  END -->

            <section class="preview-chapterList">
                <div class="preview-chapterList-item" v-for="(chapterItem,chapterIndex) in chapterList" @click="skipChapterDetailFn(chapterItem)">
                    <div>
                        <span v-text="chapterType[chapterItem.chapter_type]">普通</span>
                        <span v-text="chapterItem.chapter_name">章节1</span>
                        <span class="fr">发布版本:<font style="color:red">{{[chapterItem.sortIndex]}}</font></span>
                    </div>
                    <div :class="chapterItem.chapter_expression.s_id == 2 ? 'chapter-bsx' : chapterItem.chapter_expression.s_id == 3 ? 'chapter-xx' : '' ">
                        <span v-text="chapterMode[chapterItem.chapter_expression.s_id]">网聊</span>
                        <span v-text="chapterItem.chapter_words  + '字'">4字</span>
                    </div>
                    <el-button @click.stop="restoreDataFn(chapterItem)" type="danger" size="mini">数据恢复</el-button>
                </div>
            </section>
        </div>


        <!-- 章节内容详情 -->
        <chapter-detail :chapter="chapter" v-if="isChapterDetail"></chapter-detail>
        
    </div>

</template>

<script>
    import pbHeader from '../../../components/pb_header'
    import chapterDetail from './chapterDetail'
    import {mapGetters} from 'vuex'
    //封装接口请求
    import {axiosGetPublishProject,axiosSaveStructures}  from '../../../service/requestConfig.js'
    export default {
        created(){
            this.init();
        },
        components:{
            pbHeader,
            chapterDetail
        },
        props:['chapterItem'],
        computed: {
            ...mapGetters({
                projectNumber:'getProjectNumber',
                projectId:'getProjectId'
            })
        },
        data(){
            return{
                componentName:"",
                //章节列表
                chapterList:[],
                 //章节类型
                chapterType:['初始','普通','结局'],
                //章节模式
                chapterMode:['网聊','半身像','胸像模式'],
                //章节id
                chapterId:0,
                //章节名称
                chapterName:"",
                //章节详情显示
                isChapterDetail:false,
                //章节内容详情
                chapter:{},
                //顺序
                sortNum:0,
            }
        },
        methods: {
            //初始化
            init(){
                let _query = this.chapterItem;
                this.chapterId = _query.chapterId;
                this.chapterName = _query.chapterName;
                this.initRequest();
            },
            //初始化请求数据
            async initRequest(){
                //存储所有的请求
                let _requestArr = [];
                //当前bin文件全程
                let _curBinName = this.activeName+'.bin';
                //遍历总发布文件次数  并添加到数组
                for(let i =1;i <= this.projectNumber;i++){
                    await axiosGetPublishProject(this.projectId,i,`chapter_${this.chapterId}.bin`).then((item)=>{
                                //给当前章节绑定索引
                                item.data.sortIndex = i;
                                //添加当前章节数据
                                _requestArr.push(item.data);
                                //当请求的章节数据是最后一次的话则渲染页面
                                if(i == this.projectNumber){
                                    this.chapterList = _requestArr;
                                }     
                            }).catch(err=>{
                                    
                            })
                }
            },
            //关闭页面
            closeFn(){
                this.$parent.isChapterItemList = false;
            },
            //数据恢复
            restoreDataFn(data){
                //恢复数据结构
                axiosSaveStructures(this.projectId,`chapter_${this.chapterId}.bin`,data).then(resp=>{
                    if(resp.data.isOk){
                        this.ut_showMessage('success','dataRenewSuccess')
                    }else{
                        this.ut_showMessage('error',resp.data.message.context);
                    }
                }).catch(err=>{
                    this.ut_showMessage('error','dataRenewFailure');
                    console.log("数据恢复失败原因",err);
                })
            },
            //跳转章节内容详情
            skipChapterDetailFn(chapter){
                this.chapter = chapter;
                this.isChapterDetail = true;
            },
            //排序
            sortFn(){
                this.sortNum = this.sortNum ? 0 : 1;
                this.chapterList.reverse();
            }
        }
    }
</script>

<style scoped>
    .v_chapterList{
        /* position: absolute;
        left:0;
        top:0; */
        width:100%;
        height: 100%;
        background-color:#f5f5f5;
    }
    .preview-chapterList-item{
        height: 110px;
    }
    .preview-chapterList-item>div{
        overflow: hidden;
    }
    .preview-chapterList-item>button{
        float:right;
    }
</style>
