<template>
  <div class="item">
    <div class="top">
      <div
        @click="bannerImageClicked"
        class="background-dark"
        :style="{
          backgroundImage: `url(${bannerDomain +
            theme.screenshot}${'?type=webp'})`
        }"
      />
    </div>
    <div class="bottom">
      <div class="name">
        <div class="name-container">
          <span class="inner-name">{{ theme.theme.name }}</span>
        </div>
      </div>
      <div class="description">{{ theme.description }}</div>
      <div class="buttons">
        <!-- <div class="stars-button">
          <div class="material-icons">star</div>
          {{theme.stars}}
        </div> -->
        <div
          class="button un-apply"
          v-if="appliedTheme === theme.id"
          @click="unApplyButton"
        >
          <span>Unapply</span>
        </div>
        <div v-else class="button" @click="applyButton">
          <span>Apply</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import config from "@/config.js";
import exploreService from "@/services/exploreService";
export default {
  props: ["theme"],
  data() {
    return {
      joinClicked: false,
      bannerDomain: config.domain + "/media/",
      appliedTheme: null
    };
  },
  methods: {
    async applyButton() {
      // get css
      const { ok, result } = await exploreService.applyTheme(this.theme.id);
      if (ok) {
        const css = result.data.css;
        const id = result.data.id;
        // save to local storage.
        localStorage.setItem("appliedThemeId", id);

        const styleEl = document.createElement("style");
        styleEl.classList.add("theme-" + id);
        styleEl.id = "theme";
        styleEl.innerHTML = css;

        const currentStyle = document.getElementById("theme");
        if (currentStyle) {
          currentStyle.outerHTML = styleEl.outerHTML;
        } else {
          document.head.innerHTML += styleEl.outerHTML;
        }
        this.appliedTheme = id;
      }
    },
    bannerImageClicked() {
      this.$store.dispatch(
        "setImagePreviewURL",
        this.bannerDomain + this.theme.screenshot + "?type=webp"
      );
    },
    unApplyButton() {
      this.appliedTheme = null;
      localStorage.removeItem("appliedThemeId");
      document.getElementById("theme").outerHTML = "";
    }
  },
  mounted() {
    this.appliedTheme = localStorage.getItem("appliedThemeId");
  },
  computed: {}
};
</script>

<style lang="scss" scoped>
.item {
  position: relative;
  width: 250px;
  height: 350px;
  background: #024253;
  opacity: 0.9;
  margin: 5px;
  border-radius: 4px;
  flex-shrink: 0;
  transition: 0.3s;
  display: flex;
  flex-shrink: 0;
  flex-direction: column;
  overflow: hidden;
  &:hover {
    opacity: 1;
  }
  .top {
    display: flex;
    flex-direction: column;
    width: 100%;
    height: 150px;
    justify-content: center;
    align-content: center;
    align-items: center;
    flex-shrink: 0;
    position: relative;
    .background-dark {
      position: absolute;
      top: 0;
      bottom: 0;
      left: 0;
      right: 0;
      background: rgba(0, 0, 0, 0.6);
      background-position: center;
      background-size: cover;
      cursor: pointer;
    }
    .avatar {
      background: rgb(26, 133, 255);
      height: 80px;
      width: 80px;
      border-radius: 50%;
      margin-bottom: 10px;
    }
  }
  .bottom {
    display: flex;
    flex-direction: column;
    background: #04333f;
    flex: 1;
    height: 100%;
    overflow: auto;
    flex-shrink: 0;
    .name {
      margin-top: 9px;
      font-size: 21px;
      white-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
      width: 100%;
      text-align: center;
      display: flex;
      z-index: 999;
      .name-container {
        display: flex;
        margin: auto;
      }
      .inner-name {
        max-width: 200px;
        overflow: hidden;
        text-overflow: ellipsis;
      }
      .material-icons {
        color: #66e0ff;
        margin-left: 5px;
      }
    }

    .description {
      margin: 10px;
      flex: 1;
      opacity: 0.9;
      overflow: auto;
      font-size: 14px;
      word-break: break-word;
      white-space: pre-wrap;
      overflow-wrap: anywhere;
      flex-shrink: 0;
    }

    .buttons {
      display: flex;
      width: 100%;
      flex-direction: row;
      margin-bottom: 10px;
    }
    .stars-button {
      display: flex;
      align-items: center;
      align-content: center;
      padding-left: 5px;
      padding-right: 5px;
      margin-left: 10px;
      margin-top: 0px;
      border-radius: 4px;
      margin-right: 10px;
      transition: 0.2s;
      flex: 1;
      cursor: pointer;
      background: #ff501bd7;
      &:hover {
        background: #ff501b;
      }
      .material-icons {
        margin-right: 5px;
      }
    }
    .button {
      display: flex;
      align-items: center;
      justify-content: center;
      width: 100%;
      height: 40px;
      border-radius: 4px;
      background: rgba(0, 179, 219, 0.8);
      transition: 0.2s;
      margin-right: 10px;
      margin-left: 10px;

      cursor: pointer;
      &:hover {
        background: #00b4db;
      }
      &.selected {
        background: grey;
      }
      &.un-apply {
        background: rgba(255, 53, 53, 0.808);
        &:hover {
          background: rgb(255, 53, 53);
        }
      }
    }
  }
}
</style>
