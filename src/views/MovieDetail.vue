<template>
  <v-sheet
    width="100%"
    class="fill-height"
  >
    <v-toolbar
      dense
      flat
      class="movie-detail-header"
      :style="{'background-color':this.$vuetify.theme.isDark ? 'rgb(39, 39, 39, 0.6)':'rgb(255, 255, 255, 0.6)','top':this.$vuetify.breakpoint.mdAndDown?'0':'48px'}"
    >
      <v-toolbar-title>
        <v-skeleton-loader
          :loading="loading"
          type="heading"
          transition="fade-transition"
          width="50vw"
        >
          <span>{{ movie.title }} {{ movie.title_en }}</span>
        </v-skeleton-loader>
      </v-toolbar-title>
      <v-spacer />
      <v-btn
        icon
        @click="$router.back()"
      >
        <v-icon>mdi-close-circle</v-icon>
      </v-btn>
    </v-toolbar>
    <v-img
      width="100%"
      max-height="70vh"
      :src="movie.poster"
      :alt="movie.title + 'Cover'"
      gradient="to bottom,rgba(64, 64, 64, 0) 60%,rgba(30, 30, 30, 100) 100%"
      class="movie-detail-cover mt-n12"
    />
    <v-container class="py-0">
      <v-row
        justify="center"
      >
        <v-col
          cols="8"
          sm="6"
          md="5"
          lg="3"
        >
          <!--get movie skeleton-->
          <v-skeleton-loader
            :loading="loading"
            :types="{ skeleton: 'image@2' }"
            type="skeleton"
            transition="fade-transition"
          >
            <v-img
              :src="movie.poster"
              alt="Movie Poster"
              min-height="300px"
              @error="posterLoadFail"
            >
              <!--load image skeleton-->
              <template v-slot:placeholder>
                <v-skeleton-loader
                  :loading="loading"
                  :types="{ skeleton: 'image@2' }"
                  type="skeleton"
                />
              </template>
              <template v-slot:default>
                <v-row
                  v-if="error"
                >
                  <v-col
                    cols="12"
                    class="text-center"
                  >
                    <p class="title font-weight-bold mb-0">
                      {{ $t('posterLoadFail') }}
                    </p>
                  </v-col>
                </v-row>
              </template>
            </v-img>
          </v-skeleton-loader>
        </v-col>
      </v-row>
      <v-row
        no-gutters
        justify="space-around"
        class="movie-detail-index mx-2 mx-sm-8 mx-lg-12"
      >
        <v-col
          cols="12"
          class="d-flex justify-center mt-2"
        >
          <v-skeleton-loader
            :loading="loading"
            transition="fade-transition"
            type="list-item"
            width="50vw"
          >
            <div>
              <span class="body-1 mx-2 font-weight-bold">{{ movie.year }}</span>
              <span class="body-1 mx-2 font-weight-bold">{{ movie.info.genre }}</span>
              <span class="body-1 mx-2 font-weight-bold">{{ movie.info.duration }}</span>
            </div>
          </v-skeleton-loader>
        </v-col>
        <v-col
          cols="12"
          class="text-center"
        >
          <movie-detail-video
            :trailers="movie.trailers"
            :btn-loading="btnLoading"
          />
        </v-col>
        <v-col cols="12">
          <v-skeleton-loader
            :loading="loading"
            type="paragraph"
            :class="{'my-6 mx-4' : loading}"
          >
            <v-expansion-panels flat>
              <v-expansion-panel style="background-color: transparent !important;">
                <v-expansion-panel-header class="mt-4 py-0 text-justify body-2 px-0">
                  {{ movie.info.summary.slice(0,textLength).trim() }}
                  {{ movie.info.summary.length > textLength ? '......' : '' }}
                </v-expansion-panel-header>
                <v-expansion-panel-content class="text-justify body-2">
                  <div class="mx-n6">
                    {{ movie.info.summary.substring(textLength).trim() }}
                  </div>
                </v-expansion-panel-content>
              </v-expansion-panel>
            </v-expansion-panels>
          </v-skeleton-loader>
        </v-col>
        <v-col
          cols="12"
        >
          <span class="caption grey--text">{{ $t('actors') }}: {{ movie.info.actors.split('/').slice(0,7).join('/') }}</span>
        </v-col>
        <v-col
          cols="12"
        >
          <div class="d-flex justify-space-around">
            <v-btn
              text
              x-large
              class="px-0"
              @click="addMovieToUser(movie._id,'list')"
            >
              <div>
                <v-icon
                  class="d-flex"
                  :color="userStore.list.includes(movie._id) ? 'red': undefined"
                >
                  mdi-plus
                </v-icon>
                <span class="caption grey--text">
                  {{ userStore.list.includes(movie._id) ? $t('removeList') : $t('addList') }}
                </span>
              </div>
            </v-btn>

            <movie-detail-play
              :movie-id="movie._id"
              :movie-title="movie.title"
            />

            <v-btn
              text
              x-large
              class="px-0"
              @click="addMovieToUser(movie._id,'like')"
            >
              <div>
                <v-icon
                  class="d-flex"
                  :color="userStore.like.includes(movie._id) ? 'red': undefined"
                >
                  mdi-heart
                </v-icon>
                <span class="caption grey--text">
                  {{ userStore.like.includes(movie._id) ? $t('removeLike') : $t('addLike') }}
                </span>
              </div>
            </v-btn>
          </div>
        </v-col>
      </v-row>
    </v-container>
    <movie-detail-comment
      :movie="movie"
    />
    <movie-list
      :title="movie.recs ? $t('relatedMovies') : $t('mayLike')"
      :ids="movie.recs ? movie.recs : null"
      :genre="movie.recs ? null : movie.info.genre.split('/')[0]"
    />
  </v-sheet>
</template>

<script>
import MovieList from '../components/global/MovieList'
import MovieDetailVideo from '../components/MovieDetailVideo'
import MovieDetailComment from '../components/MovieDetailComment'
import MovieDetailPlay from '../components/MovieDetailPlay'
import userMixin from '../mixins/userMixin'
import appMixin from '../mixins/appMixin'
import { getMovieByPath } from '../api/movie'
import { undefinedMovie } from '../utils'
import fallbackPoster from '../utils/fallbackPoster'
import { patchTrailers, patchBackdrops, headPoster, patchPoster } from '../api/improvement'

export default {
  name: 'MovieDetail',
  components: {
    'movie-list': MovieList,
    'movie-detail-video': MovieDetailVideo,
    'movie-detail-comment': MovieDetailComment,
    'movie-detail-play': MovieDetailPlay
  },
  mixins: [userMixin, appMixin],
  props: {
    path: {
      type: String,
      default: undefined
    }
  },
  data () {
    return {
      movie: undefinedMovie(),
      loading: true,
      btnLoading: false,
      error: false
    }
  },
  computed: {
    textLength: function () {
      const lengthSwitcher = {
        xs: 80,
        sm: 150,
        md: 200,
        lg: 250,
        xl: 300
      }
      return lengthSwitcher[this.$vuetify.breakpoint.name]
    }
  },
  // exit component and enter again
  async beforeRouteEnter (to, from, next) {
    next(async vm => {
      await vm.init(vm, to.params.path)
    })
  },
  // same component update data
  async beforeRouteUpdate (to, from, next) {
    next()
    await this.init(this, to.params.path)
  },
  methods: {
    async init (_this, path) {
      _this.$vuetify.goTo(0)
      this.loading = true
      this.btnLoading = true
      _this.movie = await getMovieByPath(path)
      this.loading = false
      const allow = _this.$store.state.allowImprove.allow
      const id = _this.movie._id
      // trailers = [] means no trailers, null means need patch
      if (_this.movie.trailers === null) {
        this.movie.trailers = await patchTrailers(id)
      }
      this.btnLoading = false
      if (_this.movie.backdrops === null && allow) {
        await patchBackdrops(_this.movie.path)
      }
      if (allow) {
        try {
          await headPoster(id)
        } catch (e) {
          // eslint-disable-next-line no-console
          console.log('404 not found, adding this poster to our oss. thanks for improvement')
          await patchPoster(id)
        }
      }
    },
    posterLoadFail () {
      this.error = fallbackPoster(this.movie)
    }
  }
}
</script>

<style lang="scss">
  .movie-detail {
    &-header {
      z-index: 2;
      position: sticky;
      -webkit-backdrop-filter: saturate(180%) blur(20px);
      backdrop-filter: saturate(180%) blur(20px);
    }

    &-cover {
      position: absolute;
      filter: blur(20px);
    }

    &-index div {
      z-index: 1;
    }
  }
</style>
