<template>
  <div class="actions" >
    <div>
      <i class="action icon-smile"
         title="Reaction"
         @click="onReaction"
      ></i>
      <i class="action icon-message-circle-left-speak"
         title="Reply in thread"
         v-if="message.canReply && !hideActionOpenThread"
         @click="$emit('openThreadPanel', { message })"
      ></i>
      <i class="action icon-circle-right"
         title="Go to mesage"
         v-if="!hideActionGoToMessage"
         @click="$emit('goToMessage', { message })"
      ></i>
      <font-awesome-icon
        class="action"
        icon="thumbtack"
        v-if="!hidePinning"
        :class="{pinned:message.isPinned}"
        title="Pin message for everyone to see"
        @click="$bus.$emit('message.pin', { message })"
      ></font-awesome-icon>
      <font-awesome-icon
        class="action"
        :icon="['far', 'bookmark']"
        v-if="!hideBookmarking"
        :class="{bookmarked:message.isBookmarked}"
        title="Bookmark message for personal reference"
        @click="$bus.$emit('message.bookmark', { message })"
      ></font-awesome-icon>
      <i v-if="!isContextMenuOpen && isContextMenuEnabled"
        class="action icon-plus" @click="onContextMenuOpen()"></i>
      <i v-else-if="isContextMenuEnabled"
        class="action icon-close" @click="isContextMenuOpen=false"></i>
    </div>
    <div class="context-menu" v-if="isContextMenuOpen && isContextMenuEnabled">
      <ul class="context-menu-list">
        <li v-if="message.canReply && !hideActionOpenThread"
            class="extra-action"
            @click="$emit('openThreadPanel', { message })">
            <i class="icon icon-message-circle-left-speak"></i>
            <span>Reply in thread </span>
        </li>
        <li v-if="message.canEdit"
            class="extra-action"
            @click="$emit('editMessage', { message })">
            <i class="icon icon-edit"></i>
            <span>edit message</span>
        </li>
        <li v-if="message.canDelete"
            class="extra-action"
            @click="$emit('deleteMessage', { message })">
            <i class="icon icon-trash"></i>
            <span>delete message</span>
        </li>
      </ul>
    </div>
  </div>
</template>
<script>

export default {
  props: {
    message: {
      type: Object,
      required: true,
    },

    hideActions: Boolean,
    hideActionOpenThread: Boolean,
    hideActionsMenu: Boolean,
    hidePinning: Boolean,
    hideBookmarking: Boolean,
    hideActionGoToMessage: { type: Boolean, default: false },
  },

  data () {
    return {
      isContextMenuOpen: false,
    }
  },

  computed: {
    isContextMenuEnabled: function () {
      return !this.hideActionsMenu &&
        ((this.message.canReply && !this.hideActionOpenThread) ||
          this.message.canEdit ||
          this.message.canDelete)
    },
  },

  methods: {
    onContextMenuOpen () {
      const evName = 'Messages/Message.contextMenuOpen'
      this.$bus.$emit(evName)
      this.$bus.$once(evName, () => { this.isContextMenuOpen = false })
      this.isContextMenuOpen = true
    },

    onReaction () {
      this.$bus.$emit('ui.openEmojiPicker', {
        callback: ({ colons }) => {
          // Got called back from the emoji picker, now send the reaction to this message...
          this.$bus.$emit('message.reaction', { message: this.message, reaction: colons })
        },
      })
    },
  },
}

</script>
<style scoped lang="scss">
@import '@/assets/sass/_0.commons.scss';

.actions, .context-menu {
  position: absolute;
  border-radius: 3px;
  width: 100%;
  left: 0px;
  text-align: right;
  top: -15px;
  display: inline-block;
  padding: 0 0.5em;
  z-index: 5;

  .action {
    display: inline-block;
    border: solid 1px rgba($appgrey, 0.25);
    margin: 0 5px;
    border-radius: 30px;
    line-height: 30px;
    width: 30px;
    background-color: $appwhite;
    font-size: 18px;
    text-align: center;
    box-shadow: 0 0 5px 0 rgba($appgrey, 0.5);
    cursor: pointer;
    color: $defaultlinecolor;
  }
}

.actions {
  min-width: 180px;
  .bookmarked, .pinned {
    color: $appyellow;
  }
  .svg-inline--fa{
    padding: 7px;
    width: 30px;
    height: 32px;
    margin-bottom: -6px;
  }
}

.context-menu {
  min-width: 180px;
  left: -30px;
  z-index: 6;

  &:hover {
    display: block;
  }

  .context-menu-list {
    list-style: none;
    float: right;
    background-color: white;
    box-shadow: 0 0 5px 0 rgba($appgrey, 0.5);
    border: solid 1px rgba($appgrey, 0.25);
    padding: 0;
    margin: 0;
    margin-top: -1px;

    .extra-action {
      padding: 7px 10px;
      text-align: left;

      * {
        display: inline-block;
        line-height: 18px;
        vertical-align: middle;
      }

      .icon {
        font-size: 16px;
        margin-right: 5px;
      }

      &:hover {
        background-color: $appcream;
        cursor: pointer;
      }

    }

  }

}
</style>