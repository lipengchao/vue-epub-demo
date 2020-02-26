<template>
  <div class="ebook">
    <title-bar :showFlag="showFlag"></title-bar>
    <div class="read-wrapper">
      <div id="read"></div>
      <div class="mask">
        <div class="mask">
          <div class="left" @click="prevPage"></div>
          <div class="center" @click="toggleTitleMenu"></div>
          <div class="right" @click="nextPage"></div>
        </div>
      </div>
    </div>
    <menu-bar
      ref="menuBar"
      @setFontSize="setFontSize"
      :defaultFontSize="defaultFontSize"
      :fontSizeList="fontSizeList"
      :showFlag="showFlag"
      :themeList="themeList"
      :defaultTheme="defaultTheme"
      @setTheme="setTheme"
      :bookAvailable="bookAvailable"
      @onProgressChange="onProgressChange"
      @jumpTo="jumpTo"
      :navigation="navigation"
    ></menu-bar>
  </div>
</template>

<script>
import Epub from 'epubjs'
import TitleBar from '@/components/TitleBar'
import MenuBar from '@/components/MenuBar'
const DOWNLOAD_URL = '/static/2018_Book_AgileProcessesInSoftwareEngine.epub'
global.ePub = Epub
export default {
  components: {
    TitleBar,
    MenuBar
  },
  data() {
    return {
      showFlag: false,
      // 字号数组
      fontSizeList: [
        { fontSize: 12 },
        { fontSize: 14 },
        { fontSize: 16 },
        { fontSize: 18 },
        { fontSize: 20 },
        { fontSize: 22 },
        { fontSize: 24 }
      ],
      // 默认字号大小
      defaultFontSize: 16,
      // 主题列表
      themeList: [
        {
          name: 'default',
          style: {
            body: {
              'color': '#000',
              'background': '#fff'
            }
          }
        },
        {
          name: 'eye',
          style: {
            body: {
              'color': '#000',
              'background': '#ceeaba'
            }
          }
        },
        {
          name: 'night',
          style: {
            body: {
              'color': '#fff',
              'background': '#000'
            }
          }
        },
        {
          name: 'gold',
          style: {
            body: {
              'color': '#000',
              'background': 'rgb(241, 236, 226)'
            }
          }
        }
      ],
      // 默认主题索引
      defaultTheme: 0,
      // 图书是否处于可用状态
      bookAvailable: false,
      // 目录对象
      navigation: {}
    }
  },
  mounted () {
    this.showEpub()
  },
  methods: {
    // 电子书的解析和渲染
    showEpub () {
      // 生成Book
      this.book = new Epub(DOWNLOAD_URL)
      // 生成Rendition
      this.rendition = this.book.renderTo('read', {
        width: window.innerWidth,
        height: window.innerHeight
      })
      // 通过Rendtion.display渲染电子书
      this.rendition.display()
      // 获取Theme对象
      this.themes = this.rendition.themes
      // 设置默认字号大小
      this.setFontSize(this.defaultFontSize)
      // this.themes.register(name, styles)
      // this.themes.select(name)
      // 注册主题
      this.registerTheme()
      // 设置默认样式
      this.setTheme(this.defaultTheme)
      // 获取Location对象
      // this.book.locations 因为locations对象生成是比较消耗性能的，所以默认不会生成
      // 需要通过epubjs的钩子函数来实现
      this.book.ready.then(() => {
        // 获取目录对象
        this.navigation = this.book.navigation
        return this.book.locations.generate()
      }).then(result => {
        // 获取进度对象
        this.locations = this.book.locations
        this.bookAvailable = true
      })
    },
    // 上一页
    prevPage () {
      // Rendition.prev
      this.rendition.prev()
    },
    // 下一页
    nextPage () {
      // Rendition.next
      if (this.rendition) {
        this.rendition.next()
      }
    },
    // 标题栏和菜单栏显示和隐藏
    toggleTitleMenu () {
      this.showFlag = !this.showFlag
      if (!this.showFlag) {
        this.$refs.menuBar.hideSetting()
      }
    },
    // 设置字号大小
    setFontSize (fontSize) {
      this.defaultFontSize = fontSize
      if (this.themes) {
        this.themes.fontSize(fontSize + 'px')
      }
    },
    // 注册主题
    registerTheme () {
      this.themeList.forEach(theme => {
        this.themes.register(theme.name, theme.style)
      })
    },
    // 设置主题
    setTheme (index) {
      this.themes.select(this.themeList[index].name)
      this.defaultTheme = index
    },
    // progress 进度条的数值(0-100)
    onProgressChange(progress) {
      const percentage = progress / 100
      // this.locations.cfiFromPercentage 获取位置
      const location = percentage > 0 ? this.locations.cfiFromPercentage(percentage) : 0
      this.rendition.display(location)
    },
    // 根据链接跳转到指定位置
    jumpTo (href) {
      this.rendition.display(href)
      this.hideTitleAndMenu()
    },
    hideTitleAndMenu() {
      // 隐藏标题栏和菜单栏
      this.showFlag = false
      // 隐藏菜单栏弹出的设置栏
      this.$refs.menuBar.hideSetting()
      // 隐藏目录
      this.$refs.menuBar.hideContent()
    }
  }
}
</script>

<style lang='scss' scoped>
@import 'assets/styles/global.scss';
.ebook {
  position: releative;
  .read-wrapper {
    .mask {
      position: absolute;
      display: flex;
      top: 0;
      left: 0;
      z-index: 100;
      width: 100%;
      height: 100%;
      .left {
        flex: 0 0 px2rem(100);
      }
      .center {
        flex: 1;
      }
      .right {
        flex: 0 0 px2rem(100);
      }
    }
  }
}
</style>
