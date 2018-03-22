<template>
  <li :class="{ current: item.isPlay }" @click="select">
    <div class="img" ref="img" :id="item.id">
      <div class="top" v-if="item.isPlay"></div>
      <div class="right" v-if="item.isPlay"></div>
      <div class="bottom" v-if="item.isPlay"></div>
      <div class="left" v-if="item.isPlay"></div>
      <img :src="item.src" alt="search result image">
    </div>
    <p>{{item.title}}</p>
    <p>{{item.artistName}}</p>
  </li>
</template>

<script>
export default {
  props: ["item", "playOrStop"],
  data() {
    return {
      color: ""
    }
  },
  updated() {
    const canvas = document.createElement("canvas")
    const ctx = canvas.getContext('2d')
    const image = new Image()
    image.crossOrigin = 'Anonymous'
    image.onload = () => {
      const width = image.width
      const height = image.height
      const gav = this.getAspectValue(image)
      ctx.drawImage(image, 0, 0, gav.width, gav.height)
      const rgb = this.getDominantColor(ctx, gav.width, gav.height)
      this.color = `rgb(${rgb.join(",")})`
    }
    image.src = this.item.src
  },
  methods: {
    select() {
      this.playOrStop(this.item.id)
      setTimeout(() => {
        const img = document.getElementById(this.item.id)
        for(let el of img.children) {
          const c = el.classList.value
          if(c == "top"|| c == "right" || c == "bottom" || c == "left") {
            el.style.backgroundColor = this.color
          }
        }
      }, 500)
    },
    getAspectValue(img, s = 100) {
      if(img.width < img.height) {
        return {
          width: Math.ceil(img.width * ( s / img.height )),
          height: s
        }
      } else {
        return {
          width: s,
          height: Math.ceil(img.height * ( s / img.width ))
        }
      }
    },
    getDominantColor(ctx, w, h) {
      const rgb = [0, 0, 0]
      const count = w * h
      for(let i = 0; i <= w; i++) {
        for(let j = 0; j <= h; j++) {
          const [red, green, blue, alpha] = Array.slice(ctx.getImageData(i, j, 1, 1).data)
          rgb[0] += red
          rgb[1] += green
          rgb[2] += blue
        }
      }
      rgb[0] = Math.ceil(rgb[0] / count)
      rgb[1] = Math.ceil(rgb[1] / count)
      rgb[2] = Math.ceil(rgb[2] / count)
      return rgb
    }
  }
}
</script>

<style scoped>
li {
  width: 14.666%;
  margin: 1%;
}
.current {
  transform: scale(1.1);
}
.img {
  width: 100%;
  position: relative;
}
.img .top {
  position: absolute;
  top: 0px;
  left: 0;
  width: 0%;
  height: 5px;
  background-color: red;
  animation-name: top;
  animation-duration: 5s;
  animation-timing-function: linear;
  animation-fill-mode: forwards;
  animation-delay: .5s;
}

@keyframes top {
  0% {
    width: 0%;
  }
  100% {
    width: 100%;
  }
}

.img .right {
  position: absolute;
  top: 0;
  right: 0px;
  width: 5px;
  height: 0%;
  background-color: red;
  animation-name: right;
  animation-duration: 5s;
  animation-timing-function: linear;
  animation-fill-mode: forwards;
  animation-delay: 5.5s;
}

@keyframes right {
  0% {
    height: 0%;
  }
  100% {
    height: 100%;
  }
}

.img .bottom {
  position: absolute;
  bottom: 0px;
  right: 0;
  width: 0%;
  height: 5px;
  background-color: red;
  animation-name: bottom;
  animation-duration: 5s;
  animation-timing-function: linear;
  animation-fill-mode: forwards;
  animation-delay: 10.5s;
}

@keyframes bottom {
  0% {
    width: 0%;
  }
  100% {
    width: 100%;
  }
}

.img .left {
  position: absolute;
  bottom: 0;
  left: 0px;
  width: 5px;
  height: 0%;
  background-color: red;
  animation-name: left;
  animation-duration: 5s;
  animation-timing-function: linear;
  animation-fill-mode: forwards;
  animation-delay: 15.5s;
}

@keyframes left {
  0% {
    height: 0%;
  }
  100% {
    height: 100%;
  }
}

.img img {
  display: block;
  width: 100%;
}
</style>
