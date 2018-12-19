<template>
    <el-dialog title=""
               :close-on-click-modal="false"
               :close-on-press-escape="false"
               :before-close="closeFirstDialog"
               class="alam_dialog"
               style="overflow: hidden"
               :visible.sync="showDialog">
      <el-tabs class="alam_tabs" v-model="activeName" @tab-click="handleClick">
        <el-tab-pane label="报警详情" name="0">
          <div class="alam_img_box">
            <div class="face-photos-title">
              <div class="title-box clear">
                <p>{{$t('alarmsHistory.face')}}</p>
                <p>{{$t('alarmsHistory.match')}}（{{activeIndex + 1}} / {{ dialogInfo.photos.length }}）</p>
              </div>
              <div class="compare_img_box">
                <div class="img-bg compare_img1" @mouseover="jumpOptionOver" @mouseleave="jumpOptionLeave">
                  <jumpFigure :jumpOptionVisible="jumpOptionVisible" :jumpImg="dialogInfo&&dialogInfo.face"></jumpFigure>
                  <img v-lazy="dialogInfo.face"
                       :key="dialogInfo.face"
                       alt="" v-imgEasy:bottom="dialogInfo.face" style="margin-right: 0!important;"
                       @click.stop="secondDialog.showZoomImg=true;zoomImgSrc=dialogInfo.face ">
                </div>
                <div class="img-bg compare_img2"
                     @click.stop="secondDialog.showZoomImg=true;zoomImgSrc=matchInfo[activeIndex].path ">
                  <alam-slide ref='slide_' :slides="this.dialogInfo.photos" :activeIndex="activeIndex"
                              @activeIndexChange="indexChange"></alam-slide>
                </div>
              </div>
            </div>
            <div class="body-title" v-if="dialogInfo.body&&!dialogInfo.full">
              <p>{{$t('alarmsHistory.body')}}</p>
              <div class="body-photo">
                <div class="img-bg">
                  <drawFace :src="dialogInfo.body"
                            :imgPositionInfo="dialogInfo.bodyPositionInfo"
                            :imgInfo="dialogInfo.facePositionInfo"></drawFace>
                </div>
              </div>
            </div>

            <div class="body-title" v-if="dialogInfo.full">
              <p>{{$t('alarmsHistory.full')}}</p>
              <div class="body-photo">
                <drawFace :src="dialogInfo.full"
                          :imgPositionInfo="dialogInfo.fullPositionInfo"
                          :imgInfo="dialogInfo.bodyPositionInfo"></drawFace>
              </div>
            </div>
          </div>
          <div class="info-box clear">
            <p>
              <i class="icon-user" :title="$t('alarm_dialog.name')"></i>
              <span>{{matchInfo[activeIndex].subject.subjectName?matchInfo[activeIndex].subject.subjectName:$t('alarm_dialog.unknow')}}</span>
              <i class="iconfont icon-nv" v-if="matchInfo[activeIndex].gender == 0"></i>
              <i class="iconfont icon-nan" v-if="matchInfo[activeIndex].gender == 1"></i>
            </p>
            <p>
              <i class="iconfont icon-zhuanshufangandibucaidantubiao"
                 :title="$t('alarm_dialog.criminalRecord')"></i>
              <span>
                    {{matchInfo[activeIndex].subject.criminalRecord==1?$t('alarm_dialog.have'):matchInfo[activeIndex].subject.criminalRecord==2?$t('alarm_dialog.without'):$t('alarm_dialog.unknow')}}
                </span>
            </p>
            <p>
              <i class="el-icon-tickets"
                 :title="$t('alarm_dialog.album')"></i>
              <span>{{matchInfo[activeIndex].group.groupName}}</span>
            </p>
            <p>
              <i class="el-icon-location"
                 :title="$t('alarm_dialog.location')"></i>
              <span>{{dialogInfo.camera_location}}</span>
            </p>
            <p>
              <i class="el-icon-time"
                 :title="$t('alarm_dialog.timeStamp')"></i>
              <span>{{dialogInfo.timestamp}}</span>
            </p>
            <p>
              <i class="iconfont icon-huzhao"
                 v-if="matchInfo[activeIndex].subject.certType === 2"
                 :title="$t('cert_type.passport')"></i>
              <i class="iconfont icon-shenfenzheng1"
                 v-if="matchInfo[activeIndex].subject.certType === 1"
                 :title="$t('cert_type.passport.card')"></i>
              <i class="iconfont icon-shen"
                 v-if="matchInfo[activeIndex].subject.certType === 3"
                 :title="$t('cert_type.passport.officer')"></i>
              <span v-if="matchInfo[activeIndex].subject.certId"
                    :title="$t('albumUser.id_num')">
                    {{matchInfo[activeIndex].subject.certId}}
                </span>
            </p>
            <p>
              <i class="el-icon-tickets" :title="$t('alarm_dialog.deal_state')"></i>
              <span v-if="dealState===1">
                    {{$t('alarm_dialog.already_deal')}}
                </span>
              <span v-else>
                     {{$t('alarm_dialog.not_deal')}}
                </span>
            </p>
          </div>
          <div class="alam-button">
            <el-button-group>
              <div @click.stop="secondDialog.dialogAdd=true">
                <i class="iconfont icon-ic_person_add_px"></i>
                <p>{{$t('button.add_to_warehouse')}}</p>
              </div>
              <div @click.stop="deleteAlam" v-if="dialogInfo.status">
                <i class="iconfont icon-shanchu"></i>
                <p>{{$t('button.delete_alarm')}}</p>
              </div>
              <div @click.stop="backoutDelete(dialogInfo.id)" v-if="!dialogInfo.status">
                <i class="iconfont icon-shanchu"></i>
                <p>{{$t('button.cancel_delete')}}</p>
              </div>
              <div @click="secondDialog.showReportDialog=true" v-if="dealState!=1">
                <i class="iconfont icon-zhuanshufangandibucaidantubiao"></i>
                <p>{{$t('button.results_deal')}}</p>
              </div>
              <div @click="secondDialog.showReportDialog=true;reportType=true" v-if="dealState===1">
                <i class="iconfont icon-zhuanshufangandibucaidantubiao"></i>
                <p>{{$t('alarm_dialog.police_detail')}}</p>
              </div>
            </el-button-group>
          </div>

        </el-tab-pane>
        <el-tab-pane label="视频回放" name="1" v-if=" dialogInfo.type ===0 && videoSrc">
          <el-row>
            <el-col :span="8" style="text-align: center">
              <p style="padding-bottom: 10px">{{$t('alarmsHistory.face')}}</p>
              <div class="all-img-containe" style="position: relative;"  @mouseover="jumpOptionOver" @mouseleave="jumpOptionLeave">
                <jumpFigure :jumpOptionVisible="jumpOptionVisible" :jumpImg="dialogInfo&&dialogInfo.face"></jumpFigure>
                <img v-lazy="dialogInfo.face"
                     :key="dialogInfo.face"
                     @click.stop="secondDialog.showZoomImg=true;zoomImgSrc=dialogInfo.face ">
              </div>
              <!--<el-button :disabled="!videoSrc" class="downLoad" plain icon="el-icon-download ">-->
                <!--<a :href="videoSrc+'?Filename=抓拍视频.mp4'" download>下载视频</a>-->
                <!--&lt;!&ndash;<a href="http://10.199.1.75:8080/v4/clips/bweed-&#45;&#45;170-_41b0acf36056fc_meta/data?downloadFilename=抓拍视频.mp4" download="抓拍视频.mp4">下载视频</a>&ndash;&gt;-->
              <!--</el-button>-->
            </el-col>
            <el-col :span="16"  style="text-align: center">
              <PlayVideo  :video-src='videoSrc' :video-ready="videoReady"></PlayVideo>
            </el-col>
          </el-row>
          <div class="alam-button" style="margin-top: 44px">
            <el-button-group>
              <div @click.stop="secondDialog.dialogAdd=true">
                <i class="iconfont icon-ic_person_add_px"></i>
                <p>{{$t('button.add_to_warehouse')}}</p>
              </div>
              <div @click.stop="deleteAlam" v-if="dialogInfo.status">
                <i class="iconfont icon-shanchu"></i>
                <p>{{$t('button.delete_alarm')}}</p>
              </div>
              <div @click.stop="backoutDelete(dialogInfo.id)" v-if="!dialogInfo.status">
                <i class="iconfont icon-shanchu"></i>
                <p>{{$t('button.cancel_delete')}}</p>
              </div>
              <div @click="secondDialog.showReportDialog=true" v-if="dealState!=1">
                <i class="iconfont icon-zhuanshufangandibucaidantubiao"></i>
                <p>{{$t('button.results_deal')}}</p>
              </div>
              <div @click="secondDialog.showReportDialog=true;reportType=true" v-if="dealState===1">
                <i class="iconfont icon-zhuanshufangandibucaidantubiao"></i>
                <p>{{$t('alarm_dialog.police_detail')}}</p>
              </div>
            </el-button-group>
          </div>
        </el-tab-pane>
      </el-tabs>
        <!--查看大图-->
        <zoom-img v-if="secondDialog.showZoomImg"
                  :show-dialog="secondDialog.showZoomImg"
                  :img-src="zoomImgSrc"
                  :append-to-body="true"
                  @closeDialog="closeSecondDialog">
        </zoom-img>

        <!--大图对比-->
        <zoom-big v-if="secondDialog.showMatchBigImg"
                  :title="$t('alarm_dialog.img_compare')"
                  :show-dialog="secondDialog.showMatchBigImg"
                  :img-detail="dialogInfo"
                  :active-index="activeIndex"
                  @closeDialog="closeSecondDialog">
        </zoom-big>

        <!--弹窗-新建-->
        <dialog-add v-if="secondDialog.dialogAdd"
                    :dialogAdd="secondDialog.dialogAdd"
                    :dialogInfo="dialogInfo"
                    :append-to-body="true"
                    @closeDialog="closeSecondDialog"
                    type="alarm">
        </dialog-add>

        <report-dialog :append-to-body="true"
                       v-if="secondDialog.showReportDialog"
                       :show-dialog="secondDialog.showReportDialog"
                       @alarmDealAlarm="alarmDealAlarm"
                       @closeDialog="closeSecondDialog"
                       :type="reportType"
                       :reportType="'black'"
                       :idNo="idNo">
        </report-dialog>
    </el-dialog>
</template>

<script>
    import zoomImg from './dialog_zoom_img.vue'
    import zoomBig from './dialog_zoom_big.vue'
    import PlayVideo from '@/components/comments/play_video.vue'
    import reportDialog from '@/components/comments/report_dialog';
    import {monitoringSrv} from '../monitoring/monitoring.service';
    import alamSlide from './alam_slide';
    import dialogAdd from './dialog_add.vue';
    import drawFace from '@/components/comments/faceDraw';
    import jumpFigure from '@/components/comments/jumpFigure.vue';
    export default {
        name: "alam_history_dialog",
        props: {
            dialogInfo: Object, //报警详情/抓拍详情
            type: String,
            showDialog: {
                type: Boolean,
                default: false
            }
        },
        components: {
            zoomImg,
            zoomBig,
            PlayVideo,
            alamSlide,
            dialogAdd,
            reportDialog,
            drawFace,
          jumpFigure
        },
        computed: {
            idNo: function () {
                return this.dialogInfo.id;
            }
        },

        data() {
            return {
              jumpOptionVisible:false,
              activeName: '0',
                activeIndex: 0,//对比照活动下标
                secondDialog: {
                    'showZoomImg': false, //查看大图
                    'openVedio': false,
                    'dialogAdd': false,
                    'showReportDialog': false,//警情填报
                },
                zoomImgSrc: '', //传递要缩放的图片地址
                title: '',//弹窗标题,
                matchInfo: this.dialogInfo.photos,
                dealState: this.dialogInfo.dealFlag,
                reportType: false,
                videoSrc: this.dialogInfo.video_path,//视频回放src，通过接口请求获取
                queryNum: 0,//查询次数
                videoReady: false,
            }
        },
        methods: {
          jumpOptionLeave(){
            this.jumpOptionVisible=false;
          },
          jumpOptionOver(){
            this.jumpOptionVisible=true;
          },
          handleClick(tab, event) {
            if(tab.index==='0'){
              this.activeName='0'
            }else {
              this.activeName='1'
            }
          },
            /**
             * 关闭一级菜单
             * */
            closeFirstDialog() {
                this.$emit('closeDialog');
            },
            /**
             * 关闭二级弹窗
             * */
            closeSecondDialog() {
                let obj = this.secondDialog
                Object.keys(obj).forEach(key => {
                    obj[key] = false
                })
            },
            /**
             * 轮播组件改变下标对应展示数据
             */
            indexChange(res) {
                this.activeIndex = res;
            },
            /**
             * 删除报警**/
            deleteAlam() {
                let deleteId = this.dialogInfo.id;
                this.$emit('deleteAlam', deleteId)
            },
            /**
//             * 视频播放**/
//            playVideo() {
//                if (this.videoSrc && this.videoReady) {
//                    this.secondDialog.openVedio = true;
//                } else {
//                    return false;
//                }
//            },
            /**
             * 战果填报方法
             * */
            alarmDealAlarm(val) {
                let params = val;
                params.idNo = this.idNo;
                monitoringSrv.alarmDealAlarm(params).then(res => {
                    if (res.code === '0') {
                        this.dealState = 1;//改变处理状态
                        this.$emit('changeFlag', this.dealState);
                        this.closeSecondDialog();

                    } else {
                        this.$notify.warning({
                            title: this.$t('mes_title.fail'),
                            message: res.msg,
                            offset: 50
                        })
                        console.log('战果填报接口业务错误', res.msg)
                    }
                }).catch(err => {
                    console.log('战果填报接口网络故障', err);
                })
            },

            /**
             * 视频回放src查询接口,如果查询失败，则延迟2s查询，直到查询成功或查询20次为止**/
            getVideoPlaySrc() {
                this.queryNum++;
                let params = {};
                params.id = this.idNo;
                monitoringSrv.getVideoSrc(params).then(res => {
                    if (res.code === '0') {
                        //状态0位生成失败
                        if (res.data.status === 1) {
                            this.videoSrc = res.data.videoPath;
                            this.videoReady = true;
                        } else {
//                            this.videoSrc = '';
                            if (this.queryNum < 20 && !this.videoReady) {
                                setTimeout(() => {
                                    this.getVideoPlaySrc()
                                }, 2000);
                            }
                        }
                    } else {
                        console.log('获取报警回放视频src接口报错');
                    }

                }).catch(err => {
                    console.log('获取报警回放视频src网络故障', err)
                })

            },

            backoutDelete(id) {
                this.$emit('backoutDelete', id);
            }
        },
        created() {
            // this.title = this.$t('dialog_title.alarms_dialog');
            this.getVideoPlaySrc();
        },
        beforeDestroy() {
            this.videoReady = false;
        }
    }
</script>
<style>
  .alam_dialog .el-dialog__headerbtn .el-dialog__close{
    position: relative;
    z-index: 99;
  }
  .alam_dialog .el-dialog__header {
    padding: 0 !important;
  }
    .alam_tabs .el-tabs__item{
    width:115px;
      text-align: center;
  }

</style>
<style scoped>
.compare_img1{
  position: relative;
}
  .downLoad{
    margin-top: 42px;
    background: #fff;
    border-color: #409EFF;
    color: #409EFF;
  }
  .all-img-containe{
    height: 270px;
    width: 270px;
    padding: 10px;
    overflow: hidden;
    margin: 0 auto;
  }
  .all-img-containe img{
    width: auto;
    height: 100%;
  }
    .alam_img_box {
        display: flex;
        justify-content: space-around;
    }

    .alam_img_box p {
        text-align: center;
    }

    .compare_img_box {
        width: 560px;
        height: 280px;
        padding: 10px;
        display: flex;
        background: url("../../../static/assets/images/svg/details_bottomleft.svg"), url("../../../static/assets/images/svg/details_bottomright.svg"), url("../../../static/assets/images/svg/details_topleft.svg"), url("../../../static/assets/images/svg/details_topright.svg");
        background-repeat: no-repeat;
        background-position: bottom left, bottom right, top left, top right;
        background-size: 6px;
    }

    .img-bg {
        width: 280px;
        height: 100%;
    }

    .img-bg img {
        display: block;
        width: 100%;
    }

    .body-photo {
        width: 346px;
        height: 280px;
        box-sizing: border-box;
        padding: 10px;
        background: url("../../../static/assets/images/svg/details_bottomleft.svg"), url("../../../static/assets/images/svg/details_bottomright.svg"), url("../../../static/assets/images/svg/details_topleft.svg"), url("../../../static/assets/images/svg/details_topright.svg");
        background-repeat: no-repeat;
        background-position: bottom left, bottom right, top left, top right;
        background-size: 6px;
        margin-left: 10px;
    }

    .body-photo img {
        width: 100%;
    }

    .face-photos-title .title-box p {
        width: 50%;
        float: left;
    }

    .info-box {
        margin: 24px 0;
    }

    .info-box p {
        display: flex;
        align-items: center;
        float: left;
        line-height: 32px;
        margin: 0 20px;
    }

    .info-box p i {
        font-size: 20px;
        margin-right: 8px;
        display: inline !important;
    }

    .alam-button {
        display: flex;
        justify-content: center;
        align-items: center;
    }

    .alam-button .el-button-group a {
        color: #606266;
    }

    .alam-button .el-button-group div {
        float: left;
        cursor: pointer;
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        font-size: 12px;
        margin: 0 20px;
    }

    .alam-button .el-button-group div:hover {
        color: #7db6ec;
    }

    .alam-button .el-button-group div i {
        font-size: 18px;
        margin-bottom: -4px;
    }

    .play-btn-disabled {
        color: lightgray !important;
        cursor: no-drop !important;
    }
</style>
