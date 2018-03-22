<template>
  <section>
    <h1>好きなアーティストの名前を検索してみよう！</h1>
    <div class="input">
      <input type="text" placeholder="米津玄師" v-model="input">
    </div>

    <div v-if="items.length >= 1">
      <h2>検索結果: 「{{input}}」</h2>
      <ul>
        <item :playOrStop="playOrStop" :item="item" :key="index" v-for="(item, index) in items" ></item>
      </ul>
    </div>

    <div class="noresult" v-else>
      <h2>結果がありません</h2>
    </div>
  </section>
</template>

<script>
import item from '~/components/item.vue'
import Rx from 'rxjs'
import axios from 'axios'

export default {
  components: {
    item
  },
  data() {
    return {
      input: "daoko",
      items: [],
      someMusicPlaying: false,
      context: null,
      source: null
    }
  },
  created() {
    let input = this.input
    Rx.Observable
    .interval(10)
    .throttleTime(2000)
    .subscribe(() => {
      if(this.input != input) {
        this.getArtists(this.input)
      }
      input = this.input
    })
  },
  mounted() {
    this.context = new AudioContext()
    this.getArtists(this.input)
  },
  methods: {
    presenter(results) {
      return results.map((result) => {
        return {
          id: result.trackId,
          isPlay: false,
          src: result.artworkUrl100.split("100x100").join("500x500"),
          musicSrc: result.previewUrl,
          artistName: result.artistName,
          title: result.trackName
        }
      })
    },
    getArtists(artistName) {
      axios.get(`https://itunes.apple.com/search?term=${artistName}&media=music&entity=musicTrack&country=jp&lang=ja_jp&limit=50`)
      .then((res) => {
        this.items = this.presenter(res.data.results)
      })
    },
    playOrStop(id) {
      if(this.someMusicPlaying) {
        this.stop()
      }
      for(let item of this.items) {
        if(id == item.id) {
          if(item.isPlay) {
            this.stop()
            item.isPlay = false
          } else {
            this.play(item.musicSrc)
            item.isPlay = true
          }
        } else {
          item.isPlay = false
        }
      }
      setTimeout(() => {
        for(let item of this.items) {
          if(id == item.id) {
            item.isPlay = false
          }
        }
        this.stop()
      }, 20 * 1000)
    },
    play(url) {
      this.someMusicPlaying = true
      this.source = this.context.createBufferSource()
      axios({ url, method: "GET", responseType: "arraybuffer" }).then((res) => {
        this.context.decodeAudioData(res.data, (buffer) => {
          this.source.buffer = buffer
          this.source.connect(this.context.destination)
          setTimeout(() => {
            this.source.start(0)
          }, 500)
        })
      })
    },
    stop() {
      this.someMusicPlaying = false
      this.source.stop(0)
    }
  }
}
</script>

<style>
body, html {
  width: 100%;
  height: 100%;
}
* {
  padding: 0;
  margin: 0;
  box-sizing: border-box;
}
.input {
  width: 100%;
  display: flex;
  padding: 30px 0;
  justify-content: center;
}
.input input {
    width: 30%;
    font-size: 1.5rem;
    padding: 10px;
    font-weight: bold;
    outline: 0;
  }
h1 {
  text-align: center;
  margin: 20px 0;
}
h2 {
  text-align: center;
  font-size: 2rem;
  margin: 30px 0;
}
ul {
  display: flex;
  flex-wrap: wrap;
  list-style: none;
  justify-content: flex-start;
}
</style>
