<template>
  <div class="content server-general">
    <errors-list-template :errors="errors" v-if="errors" />
    <div class="content-inner" :key="key">
      <div class="top">
        <div class="avatar">
          <profile-picture
            class="server-avatar"
            size="100px"
            :url="update.avatar || `${avatarDomain}/${server.avatar}`"
          />
          <div class="button" @click="$refs.avatarBrowser.click()">
            Edit Avatar
          </div>
          <input
            ref="avatarBrowser"
            type="file"
            accept=".jpeg, .jpg, .png, .gif"
            class="hidden"
            @change="imageChangeEvent('avatar', $event)"
          />
        </div>
        <div class="banner">
          <div
            class="banner-image"
            :style="{
              backgroundImage: `url(${update.banner ||
                `${bannerDomain}${server.banner}`})`
            }"
          >
            <div class="banner-text"></div>
          </div>
          <div class="button" @click="$refs.bannerBrowser.click()">
            Edit Banner
          </div>
          <input
            ref="bannerBrowser"
            type="file"
            accept=".jpeg, .jpg, .png, .gif"
            class="hidden"
            @change="imageChangeEvent('banner', $event)"
          />
        </div>
      </div>
      <div class="details">
        <div class="options">
          <div class="input">
            <div class="input-title">Server Name</div>
            <input
              type="text"
              ref="name"
              placeholder="Channel Name"
              :default-value.prop="server.name"
              @input="inputEvent('name', $event)"
            />
          </div>
          <drop-down
            :items="channels"
            :selected-item="defaultChannel"
            name="Default Channel"
            @change="changeEvent('default_channel_id', $event.channelID)"
          />
        </div>
        <div
          class="button save-button"
          :class="{ disabled: requestSent }"
          v-if="changed"
          @click="updateServer()"
        >
          {{ requestSent ? "Saving..." : "Save Changes" }}
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import config from "@/config.js";
import ProfilePicture from "@/components/ProfilePictureTemplate.vue";
import ErrorsListTemplate from "@/components/app/errorsListTemplate";
import ServerService from "@/services/ServerService";
import DropDown from "./DropDownMenu";
import path from "path";

export default {
  components: { DropDown, ProfilePicture, ErrorsListTemplate },
  data() {
    return {
      requestSent: false,
      changed: false,
      errors: null,
      avatarDomain: config.domain + "/avatars/",
      bannerDomain: config.domain + "/media/",
      update: {},
      key: 1
    };
  },
  methods: {
    inputEvent(name, event) {
      this.$set(this.update, name, event.target.value);
    },
    changeEvent(name, value) {
      this.$set(this.update, name, value);
    },
    async updateServer() {
      if (this.requestSent) return;
      this.requestSent = true;
      const { ok, error } = await ServerService.updateServer(
        this.server.server_id,
        this.update
      );
      if (!ok) {
        if (error.response) {
          if (error.response.data.message)
            this.errors = [{ msg: error.response.data.message }];
          else this.errors = error.response.data.errors;
        } else {
          this.errors = [{ msg: "Something went wrong." }];
        }
        this.requestSent = false;
        return;
      }
      this.key = Math.random();
      this.update = {};
      this.requestSent = false;
    },
    //type: avatar || banner
    imageChangeEvent(type) {
      if (!this.googleDriveLinked) {
        event.target.value = "";
        return this.$store.dispatch("setPopoutVisibility", {
          name: "GDLinkMenu",
          visibility: true
        });
      }
      const file = event.target.files[0];
      const _this = this;
      const maxSize = 2092000;
      event.target.value = "";
      if (file.size > maxSize) {
        return this.$store.dispatch(
          "setGenericMessage",
          "Image is larger than 2MB"
        );
      }
      const allowedFormats = [".png", ".jpeg", ".gif", ".jpg"];
      if (!allowedFormats.includes(path.extname(file.name).toLowerCase())) {
        return this.$store.dispatch(
          "setGenericMessage",
          "That file format is not allowed!"
        );
      }
      let reader = new FileReader();
      reader.readAsDataURL(file);

      reader.onload = function() {
        _this.$set(_this.update, type, reader.result);
      };
      reader.onerror = function(error) {
        console.log("Error: ", error);
        return this.$store.dispatch(
          "setGenericMessage",
          "Something went wrong. Try again later."
        );
      };
    }
  },
  watch: {
    update(obj) {
      if (Object.keys(obj).length === 0) {
        return (this.changed = false);
      }
      this.changed = true;
    }
  },
  computed: {
    googleDriveLinked() {
      return this.$store.getters["settingsModule/settings"].GDriveLinked;
    },
    server() {
      const serverID = this.$store.state.popoutsModule.serverSettings.serverID;
      return this.$store.getters["servers/servers"][serverID];
    },
    defaultChannel() {
      const channels = this.$store.getters.channels;
      if (this.update.default_channel_id) {
        return channels[this.update.default_channel_id];
      }
      return channels[this.server.default_channel_id];
    },
    channels() {
      const serverID = this.$store.state.popoutsModule.serverSettings.serverID;
      const channelsIds = this.$store.getters["servers/channelsIDs"][serverID];
      if (channelsIds) {
        let channels = [];
        for (let channelID of channelsIds) {
          channels.push(this.$store.getters.channels[channelID]);
        }
        return channels;
      } else {
        return false;
      }
    }
  }
};
</script>

<style scoped lang="scss">
.content-inner {
  height: 100%;
  overflow: auto;
  display: flex;
  flex-direction: column;
}
.errors {
  align-self: center;
}
.server-avatar {
  align-self: center;
  margin: 10px;
}
.top {
  display: flex;
  align-self: center;
  margin-top: 10px;
  justify-content: center;
  background-color: #044050;
  flex-shrink: 0;

  .avatar {
    display: flex;
    flex-direction: column;
    height: 100%;
    flex-shrink: 0;
    .server-avatar {
      height: 100%;
      margin-top: 35px;
    }
    .button {
      margin-bottom: 16px;
    }
  }
  .banner {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-self: center;
    flex-shrink: 0;
    margin-top: 10px;
    padding: 5px;
    margin-left: 2px;
    .banner-image {
      position: relative;
      width: 250px;
      height: 130px;
      background: rgba(0, 0, 0, 0.4);
      background-position: center;
      background-size: cover;
      border-radius: 4px;
      box-shadow: 0px 2px 12px rgba(0, 0, 0, 0.13);
      .banner-text {
        position: absolute;
        bottom: 0;
        left: 0;
        right: 0;
        width: 100%;
        height: 35px;
        background: rgba(0, 0, 0, 0.5);
      }
    }
  }
}

.details {
  display: flex;
  flex-direction: column;
  width: 100%;
  height: 100%;
}
.options {
  width: 230px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-self: center;
  flex-shrink: 0;
}
.server-name {
  align-self: center;
  margin-left: 10px;
}
.input {
  display: flex;
  flex-direction: column;
  background-color: #044050;
  padding: 10px;
  align-self: center;
  margin: 10px;
  flex-shrink: 0;
}
.input input {
  margin-top: 2px;
  margin-bottom: 0;
  width: 190px;
}
.button {
  background: #033442;
  padding: 10px;
  align-self: center;
  margin: 5px;
  cursor: pointer;
  user-select: none;
  transition: 0.3s;
}
.button:hover {
  background: #022831;
}
.save-button {
  margin-top: 10px;
}
.save-button.disabled {
  background: rgba(0, 0, 0, 0.692);
}

.hidden {
  display: none;
}
@media (max-width: 390px) {
  .top {
    flex-direction: column;
    .avatar {
      height: initial;
    }
  }
}
</style>
