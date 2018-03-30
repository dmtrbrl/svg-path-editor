<template>
  <div id="app" class="spe">
    <header class="spe-header">
      <h1 class="spe-heading">SVG Path Editor</h1>
      <div class="spe-controls">
        <label class="spe-toggle">
          <input type="checkbox" class="spe-toggle__checkbox" v-model="showGrid">
          <div class="spe-toggle__btn"></div>
          <div class="spe-toggle__label">Show grid</div>
        </label>
      </div>
    </header>
    <section class="spe-main">
      <aside class="spe-aside">
        <div class="spe-aside__box">
          <label class="spe-label" for="input">Input path</label>
          <textarea id="input" class="spe-textarea" :class="{'spe-textarea--error': inputPathError}" v-model="inputPath"></textarea>
          <div class="spe-error" v-if="inputPathError">Path is not valid</div>
        </div>
        <div class="spe-aside__box">
          <label class="spe-label" for="scale">Scale</label>
          <input id="scale" class="spe-input" type="number" min="0.1" step="0.1" v-model="options.scale">
        </div>
        <div class="spe-aside__box">
          <div class="spe-label">Translate</div>
          <div class="spe-aside__group">
            <div class="spe-aside__combo">
              <label class="spe-aside__combo-label" for="translateX">X:</label>
              <input id="translateX" class="spe-input" type="number" v-model="options.translateX">
            </div>
            <div class="spe-aside__combo">
              <label class="spe-aside__combo-label" for="translateY">Y:</label>
              <input id="translateY" class="spe-input" type="number" v-model="options.translateY">
            </div>
          </div>
        </div>
        <div class="spe-aside__box">
          <label class="spe-label" for="rotate">Rotate</label>
          <input id="rotate" class="spe-input" type="number" v-model="options.rotate">
        </div>
        <div class="spe-aside__box">
          <label class="spe-label" for="output">Output path</label>
          <textarea id="output" class="spe-textarea" v-model="outputPath" onclick="this.focus();this.select()" readonly></textarea>
        </div>
      </aside>
      <div class="spe-content">
        <svg version="1.1"
          baseProfile="full"
          width="100%" height="100%"
          xmlns="http://www.w3.org/2000/svg"
          class="spe-content__svg">
          <!-- Grid -->
          <defs>
            <pattern id="smallGrid" width="8" height="8" patternUnits="userSpaceOnUse">
              <path d="M 8 0 L 0 0 0 8" fill="none" stroke="#393939" opacity="0.4" stroke-width="0.5"/>
            </pattern>
            <pattern id="grid" width="80" height="80" patternUnits="userSpaceOnUse">
              <rect width="80" height="80" fill="url(#smallGrid)"/>
              <path d="M 80 0 L 0 0 0 80" fill="none" stroke="#393939" opacity="0.4" stroke-width="1"/>
            </pattern>
          </defs>
          <rect class="spe-content__svg-grid" :class="{'spe-content__svg-grid--visible': showGrid}" width="100%" height="100%" fill="url(#grid)" />
          <!-- Path -->
          <path :d="outputPath" fill="#393939" ref="path"></path>
        </svg>
      </div>
    </section>
  </div>
</template>

<script>
// Docs: https://github.com/fontello/svgpath
import svgpath from 'svgpath';

export default {
  name: 'app',
  data () {
    return {
      inputPath: "M164.441 44.563l53.795 53.794-136.169 136.17-53.765-53.794 136.14-136.171z m94.616-12.975l-23.991-23.99c-9.271-9.271-24.326-9.271-33.629 0l-22.981 22.98 53.795 53.795 26.806-26.805c7.191-7.192 7.191-18.789 0-25.98z m-258.907 224.757c-0.979 4.406 2.999 8.354 7.405 7.282l59.946-14.534-53.765-53.795-13.586 61.047z",
      inputPathError: false,
      outputPath: null,
      options: {
        scale: 1,
        translateX: 0,
        translateY: 0,
        rotate: 0
      },
      showGrid: false,
      centerX: 0,
      centerY: 0
    }
  },
  methods: {
    updatePath() {
      const newPath = svgpath(this.inputPath);
      if(!newPath.err && this.inputPath){
        this.outputPath = newPath
                          .rotate(this.options.rotate || 0, this.centerX, this.centerY)
                          .scale(Number(this.options.scale) || 1)
                          .translate(Number(this.options.translateX) || 0, Number(this.options.translateY) || 0)
                          .rel()
                          .round(3)
                          .toString();
        this.inputPathError = false;
      } else {
        this.inputPathError = true;
      }
    },
    resetOptions() {
      this.options = {
        scale: 1,
        translateX: 0,
        translateY: 0,
        rotate: 0
      }
    },
    setCenterCoordinates() {
      const SVGRect = this.$refs.path.getBBox();
      this.centerX = SVGRect.width / 2 + SVGRect.x;
      this.centerY = SVGRect.height / 2 + SVGRect.y;
    }
  },
  watch: {
    inputPath() {
      this.resetOptions();
      this.updatePath();
      setTimeout(() => {
        this.setCenterCoordinates();
      }, 100);
    },
    options: {
      handler(options) {
        // Check options
        if(options.scale < 0) options.scale = 1;
        // Update path
        this.updatePath();
      },
      deep: true
    }
  },
  mounted() {
    this.updatePath();
    setTimeout(() => {
      this.setCenterCoordinates();
    }, 100);
  }
}
</script>

<style lang="scss">
// Fonts
@import url('https://fonts.googleapis.com/css?family=Roboto+Mono:400,500');
@import url('https://fonts.googleapis.com/css?family=Roboto:400');

// Common
*{
  box-sizing: border-box;
}
body{
  padding: 0;
  margin: 0;
  background: #FFFCEF;
  font-family: 'Roboto', sans-serif;
  color: #393939;
}
input, textarea{
  min-width: 0;
  font-family: 'Roboto Mono', sans-serif;
  font-size: 13px;
  color: #393939;
}

.spe{
  // Header
  &-header{
    display: flex;
    align-items: center;
    height: 50px;
    border-bottom: 1px solid rgba(#393939, 0.2);
  }
    &-heading{
      width: 280px;
      margin: 0;
      padding: 15px;
      font-family: "Roboto Mono", sans-serif;
      font-weight: 500;
      font-size: 16px;
      line-height: 20px;
    }
    &-controls{
      padding: 10px;
    }
  // Main
  &-main{
    display: flex;
    height: calc(100vh - 50px);
  }
    // Aside
    &-aside{
      padding: 15px;
      width: 280px;
      flex-shrink: 0;
      border-right: 1px solid rgba(#393939, 0.2);
      overflow: auto;
      &__box{
        &:not(:first-child){
          margin-top: 15px;
        }
      }
      &__group{
        display: flex;
        & > *{
          &:not(:first-child){
            margin-left: 10px;
          }
        }
      }
      &__combo{
        display: flex;
        align-items: center;
        &-label{
          display: flex;
          align-items: center;
          height: 100%;
          padding: 5px 4px 5px 8px;
          background: rgba(#393939, 0.2);
          font-family: 'Roboto Mono', sans-serif;
          font-size: 12px;
        }
      }
    }
    // Content
    &-content{
      width: 100%;
      padding: 10px 0 0 10px;
      overflow: hidden;
      &__svg{
        background: rgba(#393939, 0.05);
        &-grid{
          opacity: 0;
          transition: opacity 0.5s ease;
          &--visible{
            opacity: 1;
          }
        }
      }
    }
  // Form elements
  &-toggle{
    display: flex;
    align-items: center;
    cursor: pointer;
    &__checkbox{
      display: none;
    }
    &__btn{
      position: relative;
      height: 20px;
      width: 35px;
      border-radius: 10px;
      border: 1px solid #393939;
      opacity: 0.3;
      transition: opacity 0.5s ease, background-color 0.5s ease;
      &::before{
        content: "";
        display: block;
        position: absolute;
        top: 2px;
        left: 2px;
        width: 14px;
        height: 14px;
        border-radius: 50%;
        background: #393939;
        transition: transform 0.5s ease, background-color 0.5s ease;
      }
    }
    &__checkbox:checked + &__btn{
      opacity: 1;
      background-color: #393939;
      &::before{
        transform: translateX(14.5px);
        background-color: #FFFCEF;
      }
    }
    &__label{
      display: block;
      margin-left: 5px;
      margin-top: 2px;
      font-size: 11px;
      text-transform: uppercase;
    }
  }
  &-label{
    display: block;
    text-transform: uppercase;
    font-size: 11px;
    line-height: 20px;
  }
  &-textarea{
    display: block;
    width: 100%;
    height: 100px;
    padding: 5px 7px;
    border: 1px solid rgba(#393939, 0.2);
    resize: none;
    background: transparent;
    word-break: break-all;
    &--error{
      border-color: #FF7F5B;
    }
  }
  &-input{
    display: block;
    width: 100%;
    padding: 5px 7px;
    border: 1px solid rgba(#393939, 0.2);
    background: transparent;
  }
  &-error{
    margin-top: 5px;
    font-size: 11px;
    color: #FF7F5B;
  }
}
</style>
