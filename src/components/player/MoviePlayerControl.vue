<template>
  <div class="player-control" :class="{ hidePlayerControl: hidePlayerControl }">
    <div class="timeline" :class="{timelineActived:currentPos===0}">
      <div class="current-time">
        <p>{{transTime(videoStatus.currentTime).hour}}:{{transTime(videoStatus.currentTime).minute}}:{{transTime(videoStatus.currentTime).seconds}}</p>
      </div>
      <div
        class="timing"
        :style="{ width: timeline + '%' }"
        :class="{timingActived: currentPos===0}"
      ></div>
      <div
        class="timeball"
        :style="{ left: timeline*9.66 - 8 + 'px' }"
        :class="{timeballActive:currentPos===0}"
      ></div>
      <div
        class="timeline-miniVideo"
        v-show="controlTimeline.showMiniVideo"
        :style="{ left: timeline*9.66 - 120 + 'px' }"
      >
        <VideoPlayer
          class="miniVideo"
          :manifestUri="manifestUri"
          :setVideoPlay="false"
          :setVideoCurrentTime="setTimelineToSeconds"
        />
        <div class="miniVideo-timing">
          <p>{{transTime(setTimelineToSeconds).hour}}:{{videoStatus.duration===0?0:transTime(setTimelineToSeconds).minute}}:{{videoStatus.duration===0?0:transTime(setTimelineToSeconds).seconds}}</p>
        </div>
      </div>
      <div class="duration-time">
        <p>{{transTime(videoStatus.duration).hour}}:{{transTime(videoStatus.duration).minute}}:{{transTime(videoStatus.duration).seconds}}</p>
      </div>
    </div>
    <div class="button-control">
      <div class="control-sound-sub">
        <button class="style-buttom" v-show="currentPos!=1">
          <img src="../../assets/icon-control/control-sound-sub.png" alt />
        </button>
        <button class="style-buttom buttonActived" v-show="currentPos===1">
          <img src="../../assets/icon-control/control-sound-sub-actived.png" alt />
        </button>
        <p v-show="currentPos===1">Âm thanh - phụ đề</p>
      </div>
      <div class="control-speed">
        <button class="style-buttom" v-show="currentPos!=2">
          <img src="../../assets/icon-control/control-speed.png" alt />
        </button>
        <button class="style-buttom buttonActived" v-show="currentPos===2">
          <img src="../../assets/icon-control/control-speed-actived.png" alt />
        </button>
        <p :class="{ invisible: !(currentPos===2) }">Tốc độ</p>
      </div>
      <div class="control-play-pause">
        <button class="control-pause style-buttom" v-show=" videoStatus.play && (currentPos!=3)">
          <img src="../../assets/icon-control/control-stop.png" alt />
        </button>
        <button
          class="control-pause style-buttom buttonActived"
          v-show=" videoStatus.play && (currentPos===3)"
        >
          <img src="../../assets/icon-control/control-stop-actived.png" alt />
        </button>
        <button class="control-play style-buttom" v-show=" !videoStatus.play && (currentPos!=3)">
          <img src="../../assets/icon-control/control-play.png" alt />
        </button>
        <button
          class="control-play style-buttom buttonActived"
          v-show=" !videoStatus.play && (currentPos===3)"
        >
          <img src="../../assets/icon-control/control-play-actived.png" alt />
        </button>
        <p v-show=" !videoStatus.play && (currentPos===3)">Tạm Dừng</p>
      </div>
      <div class="control-episode" v-show="typeOfMovie===0">
        <button class="style-buttom" v-show="currentPos!=4">
          <img src="../../assets/icon-control/control-episode.png" alt />
        </button>
        <button class="style-buttom buttonActived" v-show="currentPos===4">
          <img src="../../assets/icon-control/control-episode-actived.png" alt />
        </button>
        <p v-show="currentPos===4">Tập phim</p>
      </div>
    </div>
  </div>
</template>

<script>
import VideoPlayer from "./VideoPlayer";
export default {
  name: "MoviePlayerControl",
  data() {
    return {
      currentPos: 3,
      maxPos: 4,
      prePos: null,
      hidePlayerControl: true,
      controlTimeline: {
        isControl: false,
        showMiniVideo: false,
        setTimeline: 0,
      },
      keypressed: false,
    };
  },
  components: {
    VideoPlayer,
  },
  props: {
    manifestUri: String,
    typeOfMovie: Number,
    keyCode: Number,
    eventKey: Boolean,
    videoStatus: Object,
    setHidePlayerControl: Boolean,
  },
  methods: {
    transTime(time) {
      return {
        hour: Math.floor(time / 3600),
        minute: Math.floor((time % 3600) / 60),
        seconds: Math.floor(time % 60),
      };
    },
    enter() {
      if (this.hidePlayerControl) {
        this.hidePlayerControl = false;
        this.currentPos = 3;
      } else
        switch (this.currentPos) {
          case 0:
            if (this.controlTimeline.isControl || this.currentPos === 0) {
              this.$emit("setVideoCurrentTime", this.setTimelineToSeconds);
              if (
                this.setTimelineToSeconds != this.videoStatus.duration &&
                !this.videoStatus.play
              )
                this.$emit("playVideo");
              this.controlTimeline.isControl = false;
              this.controlTimeline.showMiniVideo = false;
              this.currentPos = 3;

              this.hidePlayerControl = true;
            }
            break;
          case 1:
            this.$emit("openSoundSubMenu");
            this.hidePlayerControl = true;
            break;
          case 2:
            this.$emit("openSpeedMenu");
            this.hidePlayerControl = true;
            break;
          case 3:
            if (!this.hidePlayerControl) this.$emit("playVideo");
            break;
          case 4:
            this.$emit("openEpisodeMenu");
            this.hidePlayerControl = true;
            break;
        }
      if (this.controlTimeline.isControl)
        this.controlTimeline.isControl = false;
    },
  },
  watch: {
    setHidePlayerControl: function (vNew) {
      if (!vNew) {
        this.$set(this, "hidePlayerControl", false);
        this.$emit("hideMoviePlayerControl");
      }
    },
    eventKey: function (vNew, vOld) {
      if (vNew != null) {
        switch (this.keyCode) {
          case 37:
            //Left key pressed
            if (this.currentPos > 1)
              this.$set(this, "currentPos", this.currentPos - 1);
            else if (this.currentPos === 0) {
              if (this.videoStatus.play) this.$emit("playVideo");
              this.controlTimeline.isControl = true;
              this.controlTimeline.showMiniVideo = true;

              this.controlTimeline.setTimeline - 1 >= 0
                ? (this.controlTimeline.setTimeline -= 1)
                : (this.controlTimeline.setTimeline = 0);
            }
            break;
          case 38:
            //Up key pressed
            if (!this.hidePlayerControl) {
              if (this.currentPos > 0)
                this.$set(this, "prePos", this.currentPos);
              this.$set(this, "currentPos", 0);
            } else {
              this.hidePlayerControl = false;
              this.currentPos = 3;
            }

            break;
          case 39:
            //Right key pressed
            if (this.currentPos < this.maxPos && this.currentPos > 0)
              this.$set(this, "currentPos", this.currentPos + 1);
            else if (this.currentPos === 0) {
              if (this.videoStatus.play) this.$emit("playVideo");
              this.controlTimeline.isControl = true;
              this.controlTimeline.showMiniVideo = true;

              this.controlTimeline.setTimeline + 1 <= 100
                ? (this.controlTimeline.setTimeline += 1)
                : (this.controlTimeline.setTimeline = 100);
            }
            break;
          case 40:
            //Down key pressed
            if (this.hidePlayerControl) {
              this.hidePlayerControl = false;
              this.currentPos = 3;
            } else if (this.currentPos === 0) {
              this.$set(this, "currentPos", this.prePos);
              this.controlTimeline.showMiniVideo = false;
            }
            break;
          case 13:
            //Enter key pressed
            this.enter();
            break;
        }
      }

      // auto hide PlayerControl after 3s
      var self = this;

      if (self.hidePlayerControl === false) {
        self.keypressed = true;
        let i = 0;
        let t_hideControlPlayer = setInterval(function () {
          if (!self.controlTimeline.showMiniVideo) {
            if (i === 0) self.keypressed = false;
            ++i;
            if (self.keypressed) {
              clearInterval(t_hideControlPlayer);
            } else if (i === 30 && !self.hidePlayerControl) {
              self.hidePlayerControl = true;
              self.currentPos = 3;
              if (self.controlTimeline.isControl)
                self.controlTimeline.isControl = false;
              clearInterval(t_hideControlPlayer);
            }
          }
        }, 100);
      }
    },
  },
  computed: {
    timeline: function () {
      if (!this.controlTimeline.isControl && this.videoStatus.duration != 0) {
        this.controlTimeline.setTimeline =
          (this.videoStatus.currentTime / this.videoStatus.duration) * 100;
        return this.controlTimeline.setTimeline;
      } else {
        return this.controlTimeline.setTimeline;
      }
    },
    setTimelineToSeconds: function () {
      return this.videoStatus.duration === 0
        ? 0
        : (this.controlTimeline.setTimeline / 100) * this.videoStatus.duration;
    },
  },
  mounted() {
    if (this.typeOfMovie === 0) this.maxPos = 4;
    else if (this.typeOfMovie === 1) this.maxPos = 3;
  },
};
</script>

<style>
.player-control {
  width: 100%;
  height: 100%;
  background: -webkit-gradient(
    linear,
    left top,
    left bottom,
    color-stop(0.3, rgba(0, 0, 0, 0)),
    color-stop(0.7, rgba(0, 0, 0, 0.9))
  );

  position: absolute;
  bottom: 0;

  text-align: center;

  transition: 0.1s ease;
}
.hidePlayerControl {
  height: 0;
  overflow: hidden;
}

/* ----- Css Timeline ------- */
.player-control .timeline {
  background-color: rgba(255, 255, 255, 0.3);
  width: 966px;
  height: 3px;
  position: absolute;
  left: 157px;
  bottom: 230px;
  border-radius: 2px;
}

.player-control .timeline .timing {
  background-color: #ffffff;
  width: 966px;
  height: 3px;
  position: absolute;
  bottom: 0;
  border-radius: 2px;
}
.player-control .timeline .timeball {
  width: 12px;
  height: 12px;
  border-radius: 6px;
  background-color: #ffffff;
  position: absolute;
  bottom: -4.5px;
}
.player-control .timelineActived,
.timingActived {
  height: 8px !important;
  border-radius: 4px !important;
}
.timeballActive {
  height: 20px !important;
  width: 20px !important;
  border-radius: 10px !important;
  display: block;
  bottom: -6.5px !important;
}
.player-control .timeline-miniVideo,
.miniVideo {
  background-color: #000000;
  width: 236px;
  height: 135px;
  border: 1px solid #000000;
  border-radius: 12px;
}

.player-control .timeline-miniVideo {
  position: relative;
  bottom: 160px;
  overflow: hidden;
}
.player-control .timeline-miniVideo .miniVideo {
  width: 240px;
  height: 135px;
  position: relative;
  right: 3px;
}

.player-control .timeline-miniVideo .miniVideo-timing {
  width: 240px;
  height: 25px;
  background-color: rgba(0, 0, 0, 0.85);
  position: relative;
  bottom: 27px;
  color: #ffffff;
}
.player-control .timeline-miniVideo .miniVideo-timing p {
  position: relative;
  top: 4px;
}

.player-control .timeline .current-time {
  position: absolute;
  bottom: -8px;
  left: -80px;
  font-size: 20px;
  color: #ffffff;
}
.player-control .timeline .duration-time {
  position: absolute;
  bottom: -8px;
  right: -92px;
  font-size: 20px;
  color: #ffffff;
}

/* ------ Css Player Control ----------- */
.player-control .button-control {
  position: absolute;
  bottom: 112px;
  display: flex;
  color: #ffffff;
}

.player-control .style-buttom {
  background-color: rgba(255, 255, 255, 0.2);
  border: none;
}

.player-control .button-control .buttonActived {
  background-color: #ffffff;
}
.player-control .button-control .control-sound-sub,
.control-speed,
.control-play-pause,
.control-episode {
  position: absolute;
  bottom: 0;
  width: max-content;
}
.player-control .button-control .control-sound-sub {
  left: 112px;
}
.player-control .button-control .control-speed {
  left: 225px;
}
.player-control .button-control .control-episode {
  left: 1083px;
}
.player-control .button-control .control-play-pause {
  left: 604px;
  bottom: -5px;
}
.player-control .button-control .control-episode p {
  left: -13px;
}
.player-control .button-control p {
  position: absolute;
  bottom: 0;
  width: max-content;
}
.player-control .button-control .control-sound-sub button,
.control-speed button,
.control-episode button {
  width: 40px;
  height: 40px;
  border-radius: 20px;
  padding: 0;
  position: absolute;
  bottom: 40px;
}
.player-control .button-control .control-sound-sub button {
  left: 45px;
}
.player-control .button-control .control-speed button {
  left: 5px;
}

.player-control .control-play-pause button {
  width: 56px;
  height: 56px;
  border-radius: 28px;
  padding: 0;
  position: absolute;
  bottom: 32px;
  left: 10px;
}
</style>