<template>
  <div class="menu-bar">
    <transition name="slide-up">
      <div class="menu-wrapper" :class="{'hide-box-shadow': isSetting || !showFlag}" v-show="showFlag">
        <div class="icon-wrapper" @click="showSetting(3)">
          <span class="icon-menu icon"></span>
        </div>
        <div class="icon-wrapper" @click="showSetting(2)">
          <span class="icon-progress icon"></span>
        </div>
        <div class="icon-wrapper" @click="showSetting(1)">
          <span class="icon-bright icon"></span>
        </div>
        <div class="icon-wrapper" @click="showSetting(0)">
          <span class="icon-a icon">A</span>
        </div>
      </div>
    </transition>
    <transition name="slide-up">
      <div class="setting-wrapper" v-show="isSetting">
        <div class="setting-font-size" v-if="showTag === 0">
          <div class="preview" :style="{fontSize: fontSizeList[0].fontSize + 'px'}">A</div>
          <div class="select">
            <div class="select-wrapper" v-for="(item, index) in fontSizeList" :key="index" @click="setFontSize(item.fontSize)">
              <div class="line"></div>
              <div class="point-wrapper">
                <div class="point" v-show="defaultFontSize === item.fontSize">
                  <div class="small-point"></div>
                </div>
              </div>
              <div class="line"></div>
            </div>
          </div>
          <div class="preview" :style="{fontSize: fontSizeList[fontSizeList.length - 1].fontSize + 'px'}">A</div>
      </div>
        <div class="setting-theme" v-else-if="showTag === 1">
          <div class="setting-theme-item" @click="setTheme(index)" v-for="(item, index) in themeList" :key="index">
            <div class="preview" :class="{'no-boder': item.style.body.background !== '#fff'}" :style="{background: item.style.body.background}"></div>
            <div class="text" :class="{'selected': index === defaultTheme}">{{ item.name }}</div>
          </div>
        </div>
        <div class="setting-progress" v-else-if="showTag === 2">
          <div class="progress-wrapper">
            <input
              class="progress"
              type="range"
              max="100"
              min="0"
              step="1"
              @change="onProgressChange($event.target.value)"
              @input="onProgressInput($event.target.value)"
              :value="progress"
              :disabled="!bookAvailable"
              ref="progress"
            >
          </div>
          <div class="text-wrapper">
            <span>{{ bookAvailable ? progress + '%' : '加载中...'}}</span>
          </div>
        </div>
      </div>
    </transition>
    <content-view
      :ifShowContent="ifShowContent"
      v-show="ifShowContent"
      :navigation="navigation"
      :bookAvailable="bookAvailable"
      @jumpTo="jumpTo"></content-view>
     <transition name="fade">
      <div
        class="content-mask"
        v-show="ifShowContent"
        @click="hideContent"></div>
    </transition>
  </div>
</template>

<script>
import ContentView from '@/components/ContentView'
export default {
  name: 'MenuBar',
  components: {
    ContentView
  },
  props: {
    showFlag: {
      type: Boolean,
      default: false
    },
    fontSizeList: {
      type: Array,
      default () {
        return []
      }
    },
    defaultFontSize: {
      type: Number,
      default: 16
    },
    themeList: {
      type: Array,
      default () {
        return []
      }
    },
    defaultTheme: {
      type: Number,
      default: 0
    },
    // 图书是否处于可用状态
    bookAvailable: {
      type: Boolean,
      default: false
    },
    navigation: {
      type: Object,
      default () {
        return {}
      }
    }
  },
  data() {
    return {
      isSetting: false,
      showTag: 0,
      progress: 0,
      ifShowContent: false
    }
  },
  methods: {
    // 设置字号大小
    setFontSize (fontSize) {
      this.$emit('setFontSize', fontSize)
    },
    // 显示设置
    showSetting (tag) {
      this.isSetting = true
      this.showTag = tag
      if (this.showTag === 3) {
        this.isSetting = false
        this.ifShowContent = true
      } else {
        this.isSetting = true
      }
    },
    // 隐藏设置
    hideSetting () {
      this.isSetting = false
    },
    // 设置主题
    setTheme (index) {
      this.$emit('setTheme', index)
    },
    // 进度条松开后触发事件
    onProgressChange (progress) {
      this.$emit('onProgressChange', progress)
    },
    // 拖动进度条触发事件
    onProgressInput (progress) {
      this.progress = progress
      this.$refs.progress.style.backgroundSize = `${this.progress}% 100%`
    },
    // 跳转方法，调用父组件方法
    jumpTo(target) {
      this.$emit('jumpTo', target)
    },
    // 隐藏目录
    hideContent () {
      this.ifShowContent = false
    }
  }
}
</script>

<style lang="scss" scoped>
@import '../assets/styles/global.scss';
.menu-bar {
  .menu-wrapper {
    position: absolute;
    display: flex;
    bottom: 0;
    left: 0;
    z-index: 101;
    width: 100%;
    height: px2rem(48);
    background: white;
    box-shadow: 0 px2rem(-8) px2rem(8) rgba(0, 0, 0, .15);
    &.hide-box-shadow {
      box-shadow: none;
    }
    .icon-wrapper {
      flex: 1;
      @include center;
      .icon-progress {
        font-size: px2rem(28);
      }
      .icon-bright {
        font-size: px2rem(24);
      }
    }
  }
  .setting-wrapper {
    position: absolute;
    bottom: px2rem(48);
    left: 0;
    width: 100%;
    height: px2rem(60);
    background: white;
    z-index: 101;
    box-shadow: 0 px2rem(-8) px2rem(8) rgba(0, 0, 0, .15);
    .setting-font-size {
      display: flex;
      height: 100%;
      .preview {
        flex: 0 0 px2rem(40);
        @include center;
      }
      .select {
        display: flex;
        flex: 1;
        .select-wrapper {
          flex: 1;
          display: flex;
          align-items: center;
          &:first-child {
            .line {
              &:first-child {
                border-top: none;
              }
            }
          }
          &:last-child {
            .line {
              &:last-child {
                border-top: none;
              }
            }
          }
          .line {
            flex: 1;
            height: 0;
            border-top: px2rem(1) solid #ccc;
          }
          .point-wrapper {
            position: relative;
            flex: 0 0 0;
            width: 0;
            height: px2rem(7);
            border-left: px2rem(1) solid #ccc;
            .point {
              position: absolute;
              top: px2rem(-8);
              left: px2rem(-10);
              width: px2rem(20);
              height: px2rem(20);
              border-radius: 50%;
              background: white;
              border: px2rem(1) solid #ccc;
              box-shadow: 0 px2rem(4) px2rem(4) rgba(0, 0, 0, .15);
              @include center;
              .small-point {
                width: px2rem(5);
                height: px2rem(5);
                background: black;
                border-radius: 50%;
              }
            }
          }
        }
      }
    }
    .setting-theme {
      display: flex;
      height: 100%;
      .setting-theme-item {
        flex: 1;
        display: flex;
        flex-direction: column;
        padding: px2rem(5);
        box-sizing: border-box;
        .preview {
          flex: 1;
          border: px2rem(1) solid #ccc;
          box-sizing: border-box;
          &.no-boder {
            border: none;
          }
        }
        .text {
          flex: 0 0 px2rem(20);
          font-size: px2rem(14);
          color: #ccc;
          @include center;
          &.selected {
            color: #333;
          }
        }
      }
    }
    .setting-progress {
      position: relative;
      width: 100%;
      height: 100%;
      .progress-wrapper {
        width: 100%;
        height: 100%;
        @include center;
        padding: 0 px2rem(30);
        box-sizing: border-box;
        .progress {
          width: 100%;
          -webkit-appearance: none;
          height: px2rem(2);
          background: -webkit-linear-gradient(#999, #999) no-repeat, #ddd;
          background-size: 0 100%;
          &:focus {
            outline: none;
          }
          &::-webkit-slider-thumb {
            -webkit-appearance: none;
            width: px2rem(20);
            height: px2rem(20);
            border-radius: 50%;
            background: white;
            box-shadow: 0 4px 4px rgba(0, 0, 0, .15);
            border: px2rem(1) solid #ddd;
          }
        }
      }
      .text-wrapper {
        position: absolute;
        bottom: 0;
        left: 0;
        width: 100%;
        color: #333;
        font-size: px2rem(12);
        text-align: center;
      }
    }
  }
  .content-mask {
    position: absolute;
    top: 0;
    left: 0;
    z-index: 101;
    display: flex;
    width: 100%;
    height: 100%;
    background: rgba(51, 51, 51, .8);
  }
}
</style>
