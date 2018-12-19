<template>
  <div class="video-wrapper">
    <video-player class="video-player vjs-custom-skin"
                  ref="videoPlayer"
                  :playsinline="true"
                  :options="playerOptions"
                  @ready="playerReadied">
    </video-player>
  </div>
</template>
<script>
  import { videoPlayer } from 'vue-video-player';
  import 'videojs-hotkeys'

  export default {
    components: {
      videoPlayer
    },
    props: {
      videoSrc: String,
      vedioTime: '',
      videoReady: {
        type: Boolean,
        default: false
      },
    },
    watch: {
      videoReady(cur,old){
        if (cur){ //如果是视频资源可用，则重新播放视频
          console.log('cur===', cur);
          console.log('player===', this.$refs.videoPlayer.player);
          this.playerOptions.sources[0].src = this.videoSrc;
        }
      },
    },
    data() {
      return {
        playerOptions: {
//                    playbackRates: [0.5, 1.0, 2.0,3.0], //播放速度
          height: '360',
          muted: false, // 默认情况下将会消除任何音频。
          preload: 'auto', // 建议浏览器在<video>加载元素后是否应该开始下载视频数据。auto浏览器选择最佳行为,立即开始加载视频（如果浏览器支持）
          aspectRatio: '16:9', // 将播放器置于流畅模式，并在计算播放器的动态大小时使用该值。值应该代表一个比例 - 用冒号分隔的两个数字（例如"16:9"或"4:3"）
          fluid: true, // 当true时，Video.js player将拥有流体大小。换句话说，它将按比例缩放以适应其容器。
          sources: [{
            type: "video/mp4",
            src: this.videoSrc
          }],
          notSupportedMessage: '此视频暂无法播放，请稍后再试', //允许覆盖Video.js无法播放媒体源时显示的默认信息。
        },
        initVedioTime: 0,//初始播放时间
      }
    },
    methods: {

      playerReadied(player) {
        let ct = this.initVedioTime;
        player.currentTime(ct || 0);
        player.hotkeys({
          volumeStep: 0.1,
          seekStep: 5,
          enableModifiersForNumbers: false,
          fullscreenKey: function (event, player) {
            // override fullscreen to trigger when pressing the F key or Ctrl+Enter
            return ((event.which === 70) || (event.ctrlKey && event.which === 13));
          }
        })
      }

    },
    created: function () {
      if (this.vedioTime) {
        let arr = this.vedioTime.split(':')
        let h = Number(arr[0])
        let m = Number(arr[1])
        let s = Number(arr[2])
        this.initVedioTime = s + m * 60 + h * 60 * 60
      } else {
        this.initVedioTime = 0
      }
    }

  }
</script>
<style scoped="">
  .video-wrapper {
    width: 606px;
    height: 340px;
    margin: 0 auto;
    overflow: hidden;
  }

</style>
<style>
  .vjs-error .vjs-error-display::before {
    font-size: 24px;
    content: "视频生成中，请稍后...";
  }

  .vjs-error .vjs-error-display .vjs-modal-dialog-content {
    display: none;
  }

  .vjs-custom-skin > .vjs-error .vjs-big-play-button {
    display: none;
  }
</style>
