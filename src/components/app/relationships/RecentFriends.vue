<template>
  <div class="recents">
    <transition name="list">
      <div class="list">
        <FriendsTemplate
          v-for="(channel, key) of channels"
          :key="key"
          :recents="true"
          :channel-i-d="channel.channelID"
          :recipient="channel.recipients[0]"
        />
      </div>
    </transition>
  </div>
</template>

<script>
import FriendsTemplate from "./FriendsTemplate.vue";
export default {
  components: {
    FriendsTemplate
  },
  computed: {
    channels() {
      const json = this.$store.getters.channels;
      const notifications = this.$store.getters.notifications;

      const keys = Object.keys(json);
      let result = [];
      keys.forEach(function(key) {
        if (
          json[key].recipients &&
          json[key].recipients.length > 0 &&
          !json[key].servers
        )
          result.push(json[key]);
      });

      result.sort(function(a, b) {
        const notificationA = notifications.find(item => {
          return item.channelID === a.channelID;
        });
        const notificationB = notifications.find(item => {
          return item.channelID === b.channelID;
        });
        // make notifications more prority.
        if (notificationA) return -1;
        if (notificationB) return 1;
        if (a.lastMessaged === undefined) return 1;
        if (b.lastMessaged === undefined) return -1;
        return b.lastMessaged - a.lastMessaged;
      });

      // gets unopened dms
      const notificationsFiltered = notifications.filter(item => {
        if (json[item.channelID] && json[item.channelID].server_id) return;
        const find = result.find(resFind => {
          return resFind.channelID === item.channelID;
        });
        if (!find) {
          return true;
        }
      });
      for (let index in notificationsFiltered) {
        notificationsFiltered[index].creator = "dummy";
        notificationsFiltered[index].recipients = [
          notificationsFiltered[index].sender
        ];
      }
      result = notificationsFiltered.concat(result);
      return result;
    }
  }
};
</script>
<style scoped>
.list-enter-active,
.list-leave-active {
  transition: 0.3s;
}
.list-enter, .list-leave-to /* .fade-leave-active below version 2.1.8 */ {
  transform: translateY(-20px);
  opacity: 0;
}

.recents {
  background-color: rgba(0, 0, 0, 0);
  user-select: none;
  padding-bottom: 3px;
  border-radius: 5px;
  transition: 0.3s;
}
.tab {
  transition: 0.3s;
}
.tab:hover {
  background-color: rgba(0, 0, 0, 0.123);
}
</style>
