<template>
  <div class="menu-bar">
    <transition name="slide-up">
      <div
        class="menu-wrapper"
        v-show="isShow"
        :class="{ 'hide-box-shadow': isSettingShow || !isShow }"
      >
        <div class="icon-wrapper">
          <span class="icon-menu icon" @click="showSetting(3)"></span>
        </div>
        <div class="icon-wrapper">
          <span class="icon-progress icon" @click="showSetting(2)"></span>
        </div>
        <div class="icon-wrapper">
          <span class="icon-bright icon" @click="showSetting(1)"></span>
        </div>
        <div class="icon-wrapper">
          <span class="icon-a icon" @click="showSetting(0)">A</span>
        </div>
      </div>
    </transition>
    <transition name="slide-up">
      <div class="setting-wrapper" v-show="isSettingShow">
        <div class="setting-font-size" v-if="showTag === 0">
          <div
            class="preview"
            :style="{ fontSize: fontSizeList[0].fontSize + 'px' }"
          >
            A
          </div>
          <div class="select">
            <div
              class="select-wrapper"
              v-for="(item, index) in fontSizeList"
              :key="index"
              @click="setFontSize(item.fontSize)"
            >
              <div class="line"></div>
              <div class="select-point">
                <div class="point" v-show="defaultFontSize === item.fontSize">
                  <div class="small-point"></div>
                </div>
              </div>
              <div class="line"></div>
            </div>
          </div>
          <div
            class="preview"
            :style="{
              fontSize: fontSizeList[fontSizeList.length - 1].fontSize + 'px'
            }"
          >
            A
          </div>
        </div>
        <div class="setting-theme" v-else-if="showTag === 1">
          <div
            class="setting-theme-item"
            v-for="(item, index) in themeList"
            :key="index"
            @click="setTheme(index)"
          >
            <div
              class="preview"
              :style="{ background: item.style.body.background }"
              :class="{ 'no-border': item.style.body.background !== '#fff' }"
            ></div>
            <div
              class="name"
              :class="{ selected: index === defaultThemeIndex }"
            >
              {{ item.name }}
            </div>
          </div>
        </div>
        <div class="setting-progress" v-else-if="showTag === 2">
          <div class="progress-wrapper">
            <input
              class="progress"
              type="range"
              min="0"
              max="100"
              step="1"
              :value="progress"
              :disabled="!bookAvailable"
              @change="onProgressChange($event.target.value)"
              @input="onProgressInput($event.target.value)"
              ref="progress"
            />
          </div>
          <div class="text-wrapper">
            <span>{{ bookAvailable ? progress + "%" : "加载中..." }}</span>
          </div>
        </div>
      </div>
    </transition>
    <content-view
      v-show="ifShowContent"
      :bookAvailable="bookAvailable"
      :navigation="navigation"
      @jumpTo="jumpTo"
    ></content-view>
    <transition name="fade">
      <div
        class="content-mask"
        v-show="ifShowContent"
        @click="hideContent"
      ></div>
    </transition>
  </div>
</template>

<script>
import ContentView from "./Content.vue";
export default {
  components: {
    ContentView
  },
  props: {
    isShow: {
      type: Boolean,
      default: false
    },
    fontSizeList: Array,
    defaultFontSize: Number,
    themeList: Array,
    defaultThemeIndex: Number,
    bookAvailable: {
      type: Boolean,
      default: false
    },
    navigation: Object
  },
  data() {
    return {
      isSettingShow: false,
      showTag: 0,
      progress: 0,
      ifShowContent: false
    };
  },
  methods: {
    showSetting(tag) {
      this.showTag = tag;
      if (this.showTag === 3) {
        this.isSettingShow = false;
        this.ifShowContent = true;
      } else {
        this.isSettingShow = true;
      }
    },
    hideSetting() {
      this.isSettingShow = false;
    },
    setFontSize(fontSize) {
      this.$emit("setFontSize", fontSize);
    },
    setTheme(index) {
      this.$emit("setTheme", index);
    },
    // 进度条松开后触发事件，根据进度条数值跳转到指定位置
    onProgressChange(progress) {
      this.$emit("onProgressChange", progress);
    },
    // 拖动进度条时触发事件
    onProgressInput(progress) {
      this.progress = progress;
      this.$refs.progress.style.backgroundSize = `${this.progress}% 100%`;
    },
    jumpTo(target) {
      this.$emit("jumpTo", target);
    },
    hideContent() {
      this.ifShowContent = false;
    }
  }
  // emit: ["setFontSize", "setTheme"]
};
</script>

<style lang="scss" scoped>
@import "../assets/styles/global.scss";
.menu-bar {
  .menu-wrapper {
    position: absolute;
    display: flex;
    bottom: 0;
    left: 0;
    width: 100%;
    height: px2rem(48);
    z-index: 101;
    background-color: white;
    box-shadow: 0 px2rem(-8) px2rem(8) rgba(0, 0, 0, 0.15);
    &.hide-box-shadow {
      box-shadow: none;
    }
    .icon-wrapper {
      flex: 1;
      @include center;
      .icon-progress {
        font-size: px2rem(26);
      }
      .icon-bright {
        font-size: px2rem(22);
      }
    }
  }
  .setting-wrapper {
    position: absolute;
    bottom: px2rem(48);
    left: 0;
    z-index: 101;
    width: 100%;
    height: px2rem(60);
    background-color: white;
    box-shadow: 0 px2rem(-8) px2rem(8) rgba(0, 0, 0, 0.15);
    .setting-font-size {
      display: flex;
      height: 100%;
      .preview {
        flex: 0 0 px2rem(40);
        @include center;
      }
      .select {
        flex: 1;
        display: flex;
        .select-wrapper {
          flex: 1;
          display: flex;
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
          @include center;
          .line {
            height: 0;
            border-top: px2rem(1) solid #ccc;
            flex: 1;
          }
          .select-point {
            position: relative;
            // flex: 0 0 0;
            width: 0;
            height: px2rem(7);
            border-left: px2rem(1) solid #ccc;
            .point {
              position: absolute;
              top: px2rem(-5);
              left: px2rem(-9);
              width: px2rem(15);
              height: px2rem(15);
              background-color: #fff;
              border-radius: 50%;
              border: px2rem(1) solid #ccc;
              box-shadow: 0 px2rem(4) px2rem(4) rgba(0, 0, 0, 0.15);
              @include center;
              .small-point {
                width: px2rem(3);
                height: px2rem(3);
                background-color: black;
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
          &.no-border {
            border: none;
          }
        }
        .name {
          flex: 0 0 px2rem(20);
          font-size: px2rem(14);
          color: #ccc;
          @include center;
          &.selected {
            color: #333;
            font-weight: 700;
          }
        }
      }
    }
    .setting-progress {
      position: relative;
      height: 100%;
      width: 100%;
      .progress-wrapper {
        width: 100%;
        height: 100%;
        padding: 0 px2rem(30);
        box-sizing: border-box;
        .progress {
          width: 100%;
          height: px2rem(2);
          -webkit-appearance: none;
          background: -webkit-linear-gradient(#999, #999) no-repeat, #ddd;
          background-size: 0 100%;
          &:focus {
            outline: none;
          }
          &::-webkit-slider-thumb {
            -webkit-appearance: none;
            height: px2rem(20);
            width: px2rem(20);
            border-radius: 50%;
            background: white;
            box-shadow: 0 4px 4px 0 rgba(0, 0, 0, 0.15);
            border: px2rem(1) solid #ddd;
          }
        }
      }
      .text-wrapper {
        position: absolute;
        left: 0;
        bottom: 0;
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
    width: 100%;
    height: 100%;
    background: rgba(51, 51, 51, 0.8);
  }
}
</style>
