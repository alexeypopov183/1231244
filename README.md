<template>
  <section class="section-player">
    <div class="video-player">
      <video
        ref="videoPlayer"
        class="video-js vjs-fluid vjs-big-play-centered vjs-show-big-play-button-on-pause"
      ></video>
      <div class="vjs-playlist"></div>
    </div>
    <div class="button"></div>
  </section>
</template>

<script>
import "video.js/dist/video-js.css";
import videojs from "video.js";
import "videojs-playlist";
import "videojs-playlist-ui";
import "videojs-contrib-quality-levels";
import "videojs-http-source-selector";
import "videojs-playlist-ui/dist/videojs-playlist-ui.css";

// import "videojs-resolution-switcher";
import "@silvermine/videojs-quality-selector";

export default {
  name: "VideoPlayer",
  props: {
    options: {
      type: Object,
      default() {
        return {};
      },
    },
  },
  methods: {
    // quality() {
    //   // console.log(this.player.qualityLevels());
    //   this.player.qualityLevels().on("addqualitylevel", (event) => {
    //     const qualityLevel = event.qualityLevel;
    //     qualityLevel.width = 240;
    //     qualityLevel.height = 240;
    //     qualityLevel.bitrate = 1000;
    //     qualityLevel.enabled = this.player.qualityLevels.length < 2;
    //     console.log(qualityLevel);
    //     // qualityLevel.enabled = qualityLevel.bitrate > 496000;
    //   });
    // },
  },
  data: () => ({
    player: null,
    url: null,
    playlistArray: [
      {
        id: "http://localhost:8000/live/123/index.m3u8",
        name: "Stream OBS",
        sources: [
          {
            src: "http://localhost:8000/live/123_720/index.m3u8",
            type: "application/x-mpegURL",
          },
        ],
        poster: "http://media.w3.org/2010/05/sintel/poster.png",
      },
      {
        id: "https://vz-b4f1e97e-483.b-cdn.net/65c65840-de66-4c27-afd0-a3b5a904b768/playlist.m3u8",
        name: "Stream Random",
        sources: [
          {
            src: "http://amssamples.streaming.mediaservices.windows.net/69fbaeba-8e92-4740-aedc-ce09ae945073/AzurePromo.ism/manifest(format=m3u8-aapl)",
            type: "application/x-mpegURL",
          },
        ],
        poster: "http://media.w3.org/2010/05/sintel/poster.png",
      },
      {
        id: "http://m.live.net.sa:1935/live/sunnah/playlist.m3u8",
        name: "Stream Random",
        sources: [
          {
            src: "https://nmxlive.akamaized.net/hls/live/529965/Live_1/index.m3u8",
            type: "application/x-mpegURL",
          },
        ],
        poster: "http://media.w3.org/2010/05/sintel/poster.png",
      },
    ],
  }),
  mounted() {
    try {
      this.player = videojs(this.$refs.videoPlayer, this.options, () => {
        this.player.log("onPlayerReady", this);
      });
      this.player.on("error", () => {
        this.player.createModal("Retrying connection");
        if (this.player.error()) {
          this.player.retryLock = setTimeout(() => {
            this.player.playlist.next();
            this.player.play();
          }, 3000);
        }
      });

      this.player.tech().on("retryplaylist", () => {
        this.playlistArray = this.playlistArray.filter(
          (el) => el.id !== this.player.currentSrc()
        );

        this.player.playlist(this.playlistArray);
        this.player.playlist.autoadvance(0);
        this.player.playlistUi();
      });

      // this.player.playlist(this.playlistArray);
      // this.player.playlist.autoadvance(0);
      // this.player.playlistUi();

      this.player.src([
        {
          src: "http://localhost:8000/live/123_360/index.m3u8",
          type: "application/x-mpegURL",
          label: '360'
        },
        {
          src: "http://localhost:8000/live/123_720/index.m3u8",
          type: "application/x-mpegURL",
          label: '720'
        },
      ]);

      let qualityLevels = this.player.qualityLevels();
      // qualityLevels.addQualityLevel({
      //   id: 0,
      //   width: 240,
      //   height: 240,
      //   bitrate: 1000,
      //   enabled: function() {return this.player.qualityLevels.length < 2},
      // });
      console.log(qualityLevels);
    } catch (error) {
      console.log("catch error>>>", error);
    }
  },
  beforeDestroy() {
    if (this.player) {
      this.player.dispose();
    }
  },
};
</script>

<style scoped>
.video-player {
  display: flex;
}
.video-js {
  width: 70%;
}
.vjs-playlist {
  width: 30%;
}
</style>
