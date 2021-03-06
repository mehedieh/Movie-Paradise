<template>
  <v-sheet width="100%">
    <div class="home-video-grid">
      <!--todo: elegant transition-->
      <v-img
        v-show="paused"
        :src="backdrop"
        class="home-video-grid-img"
      />
      <div class="home-video-grid-video">
        <video
          ref="video"
          autoplay
          muted
          playsinline
          x5-playsinline
          webkit-playsinline
          class="home-video-video"
          @durationchange="duration = timeTranslate(videoDom.duration)"
          @timeupdate="onTimeUpdate(videoDom.currentTime)"
          @playing="paused = false"
          @pause="paused = true"
          @ended="paused = true"
        >
          <source
            :src="videoUrl"
            type="video/mp4"
          >
          {{ $t('noVideoSupport') }}
        </video>
      </div>
      <div class="home-video-grid-layer">
        <div class="home-video-info">
          <span
            v-class="['display-1','display-2']"
            class="white--text font-weight-bold"
          >
            {{ movie.title }}
          </span>
          <div class="white--text mt-2">
            <span class="d-block">{{ movie.info.region }}
              <span v-if="movie.rating.douban_score">
                / {{ $t('douban') }} {{ movie.rating.douban_score }}</span>
              <span v-if="movie.rating.imdb_score">
                / IMDb {{ movie.rating.imdb_score }} </span>
            </span>
            <span class="d-block">{{ movie.info.genre }}</span>
          </div>
          <v-btn
            class="home-video-info-button mt-2 font-weight-bold"
            @click="$router.push({ path: `/movie/${movie.path}` })"
          >
            <v-icon left>
              mdi-information-outline
            </v-icon>
            {{ $t('moreInfo') }}
          </v-btn>
          <v-btn
            class="home-video-info-button mt-2 ml-2"
            @click="shuffle"
          >
            <v-icon center>
              mdi-shuffle-variant
            </v-icon>
          </v-btn>
        </div>
        <div class="home-video-control">
          <v-btn
            icon
            class="home-video-control-button white--text"
            @click="statusChange"
          >
            <v-icon>
              {{ icon }}
            </v-icon>
          </v-btn>
          <div
            class="home-video-control-container"
            @click="progressChange($event)"
          >
            <span
              ref="progress"
              class="home-video-control-progress"
            />
          </div>
          <span class="home-video-control-text white--text">{{ currentTime }} / {{ duration }}</span>
        </div>
      </div>
    </div>
  </v-sheet>
</template>

<script>
import { undefinedMovie } from '../utils'
import { getTodayMovie } from '../api/movie'

export default {
  name: 'HomeVideo',
  data () {
    return {
      movie: undefinedMovie(),
      paused: true,
      videoDom: undefined,
      videoUrl: '',
      icon: 'mdi-volume-off',
      duration: '00:00',
      currentTime: '00:00'
    }
  },
  computed: {
    backdrop: function () {
      if (this.movie && this.movie.backdrops !== null && this.movie.backdrops.length !== 0) {
        return 'https://image.tmdb.org/t/p/w1280/' + this.movie.backdrops[0].file_path
      } else {
        return ''
      }
    }
  },
  watch: {
    paused: function (val) {
      if (val) {
        this.icon = this.videoDom.currentTime === this.videoDom.duration ? 'mdi-restart' : 'mdi-play'
      } else {
        this.icon = this.videoDom.muted ? 'mdi-volume-off' : 'mdi-volume-high'
      }
    }
  },
  async mounted () {
    await this.init()
  },
  methods: {
    async init () {
      this.movie = await getTodayMovie()
      this.videoUrl = this.movie.trailers[0].play_url
      this.videoDom = this.$refs.video
      this.paused = true
      this.videoDom.load()
    },
    async shuffle () {
      // avoid loading cache
      this.$store.commit('SET_MOVIE_CACHE', { today: null })
      await this.init()
      this.$store.commit('SET_MOVIE_CACHE', { today: this.movie })
    },
    statusChange () {
      if (this.paused) {
        this.videoDom.currentTime = this.videoDom.currentTime === this.videoDom.duration ? 0 : this.videoDom.currentTime
        this.videoDom.play()
      } else {
        this.videoDom.muted = !this.videoDom.muted
      }
      this.icon = this.videoDom.muted ? 'mdi-volume-off' : 'mdi-volume-high'
    },
    onTimeUpdate (currentTime) {
      this.currentTime = this.timeTranslate(currentTime)
      const percentage = 100 * this.videoDom.currentTime / this.videoDom.duration
      this.$refs.progress.style.width = percentage + '%'
    },
    progressChange (e) {
      this.paused = false
      this.videoDom.currentTime = e.offsetX / e.currentTarget.offsetWidth * this.videoDom.duration
      this.videoDom.play()
      this.$refs.progress.style.width = 100 * e.offsetX / e.currentTarget.offsetWidth + '%'
    },
    timeTranslate (t) {
      let m = Math.floor(t / 60)
      m < 10 && (m = '0' + m)
      return m + ':' + (t % 60 / 100).toFixed(2).slice(-2)
    }
  }
}
</script>

<style lang="sass">
  $progress-width: 135px
  $progress-height: 36px
  .home-video
    &-video
      object-fit: cover
      width: 100%
      max-height: calc(100vh - 48px)

    &-grid
      display: grid
      grid-template-columns: 1fr
      grid-template-areas: "overflow"

      &-video
        grid-area: overflow
        overflow: hidden
        display: block

      &-img
        display: grid
        grid-area: overflow
        z-index: 1

      &-layer
        display: grid
        align-content: center
        grid-area: overflow
        grid-template-columns: 3fr 1fr
        z-index: 1

    &-info
      padding-left: 5vw

      &-button
        background-color: rgba(51, 51, 51, .6) !important
        color: white !important
        padding: 0 24px !important

        &:hover
          color: black !important
          background-color: white !important

    &-control
      display: flex
      flex-direction: row
      align-items: flex-end
      justify-content: flex-end

      &-button
        margin-right: 5px
        background-color: rgba(51, 51, 51, .6)
        border: 1px solid rgba(255, 255, 255, .5)

      &-text
        width: $progress-width
        height: $progress-height
        border: 3px white
        border-style: none none none solid
        background-color: rgba(51, 51, 51, .6)
        padding: 6px 30px 6px 6px

      &-container
        position: absolute
        width: $progress-width
        height: $progress-height
        cursor: pointer
        z-index: 1

      &-progress
        position: absolute
        height: 100%
        background-color: rgba(255, 255, 255, .5)
        transition: all .2s
</style>
