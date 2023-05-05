<template>
  <div class="ebook">
    <title-bar :isShow="isShow"></title-bar>
    <div class="read-wrapper">
      <div id="read"></div>
      <div class="mask">
        <div class="left" @click="prevPage"></div>
        <div class="center" @click="toggleShow"></div>
        <div class="right" @click="nextPage"></div>
      </div>
    </div>
    <menu-bar
      :isShow="isShow"
      :fontSizeList="fontSizeList"
      :defaultFontSize="defaultFontSize"
      :themeList="themeList"
      :defaultThemeIndex="defaultThemeIndex"
      ref="menuBar"
      @setFontSize="setFontSize"
      @setTheme="setTheme"
      :bookAvailable="bookAvailable"
      @onProgressChange="onProgressChange"
      :navigation="navigation"
      @jumpTo="jumpTo"
    ></menu-bar>
  </div>
</template>

<script>
import Epub from "epubjs";
import TitleBar from "./components/TitleBar.vue";
import MenuBar from "./components/MenuBar.vue";

const DOWNLOAD_URL = "/static/2018_Book_AgileProcessesInSoftwareEngine.epub";
global.ePub = Epub;
export default {
  components: {
    TitleBar,
    MenuBar
  },
  data() {
    return {
      isShow: false,
      fontSizeList: [
        { fontSize: 12 },
        { fontSize: 14 },
        { fontSize: 16 },
        { fontSize: 18 },
        { fontSize: 20 },
        { fontSize: 22 },
        { fontSize: 24 }
      ],
      defaultFontSize: 16,
      themeList: [
        {
          name: "default",
          style: {
            body: {
              color: "#000",
              background: "#fff"
            }
          }
        },
        {
          name: "eye",
          style: {
            body: {
              color: "#000",
              background: "#ceeaba"
            }
          }
        },
        {
          name: "night",
          style: {
            body: {
              color: "#fff",
              background: "#000"
            }
          }
        },
        {
          name: "gold",
          style: {
            body: {
              color: "#000",
              background: "rgb(241, 236, 226)"
            }
          }
        }
      ],
      defaultThemeIndex: 0,
      bookAvailable: false,
      navigation: {}
    };
  },
  methods: {
    showEpub() {
      this.book = new Epub(DOWNLOAD_URL);
      this.rendition = this.book.renderTo("read", {
        width: window.innerWidth,
        height: window.innerHeight
      });
      this.rendition.display();
      this.themes = this.rendition.themes;
      this.setFontSize(this.defaultFontSize);
      // 注册样式theme
      this.registerTheme();
      // 设置样式
      this.setTheme(this.defaultThemeIndex);
      // 获取location对象
      this.book.ready
        .then(() => {
          this.navigation = this.book.navigation;
          return this.book.locations.generate();
        })
        .then(result => {
          this.locations = this.book.locations;
          this.bookAvailable = true;
        });
    },
    // 上一页
    prevPage() {
      if (this.rendition) {
        this.rendition.prev();
      }
    },
    // 下一页
    nextPage() {
      if (this.rendition) {
        this.rendition.next();
      }
    },
    toggleShow() {
      this.isShow = !this.isShow;
      this.$refs.menuBar.hideSetting();
    },
    setFontSize(fontSize) {
      this.defaultFontSize = fontSize;
      if (this.themes) {
        this.themes.fontSize(fontSize + "px");
      }
    },
    registerTheme() {
      this.themeList.forEach(theme => {
        this.themes.register(theme.name, theme.style);
      });
    },
    setTheme(index) {
      this.themes.select(this.themeList[index].name);
      this.defaultThemeIndex = index;
    },
    // progress进度条数值，0-100
    onProgressChange(progress) {
      const percentage = progress / 100;
      const location =
        percentage > 0 ? this.locations.cfiFromPercentage(percentage) : 0;
      this.rendition.display(location);
    },
    jumpTo(href) {
      this.rendition.display(href);
      this.hideTitleAndMenu();
    },
    hideTitleAndMenu() {
      // 隐藏标题栏和菜单栏
      this.isShow = false;
      // 隐藏菜单栏弹出的设置栏
      this.$refs.menuBar.hideSetting();
      // 隐藏目录
      this.$refs.menuBar.hideContent();
    }
  },
  mounted() {
    this.showEpub();
  }
};
</script>

<style lang="scss" scoped>
@import "assets/styles/global.scss";
.ebook {
  position: relative;
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
