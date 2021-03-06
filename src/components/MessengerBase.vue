<template>
  <div>
    <div class="flex-grid">
      <!-- Left panel -->
      <base-side-panel
        key="leftPanel"
        orientation="left"
        :width="250"
        :pinned="isChannelPanelPinned"
        :hidden.sync="uiHideChannelSidePanel"
        :disable-gestures="isLeftPanelDisableGestures"
      >
        <channels-panel
          :channel="channel"
          @close="toggleChannelSidePanel(true)"
          @openQuickSearch="openQuickSearch"
          @searchSubmit="searchSubmit"
        />
      </base-side-panel>

      <!-- Content -->
      <slot />

      <!-- Right panel -->
      <base-side-panel
        v-if="uiRightSidePanelContent"
        key="rightPanel"
        orientation="right"
        :hidden.sync="uiHideRightSidePanel"
        :width="400"
        :pinned="uiPinRightSidePanel"
      >
        <members-panel
          v-if="isMembersPanel"
          :channel="channel"
          @close="switchRightSidePanel()"
        />

        <thread-panel
          v-if="isThreadPanel"
          :message="uiRightSidePanelThreadMessage"
          @close="switchRightSidePanel()"
        />

        <pinned-messages-panel
          v-if="isPinnedMessagesPanel"
          :channel="channel"
          @openThreadPanel="switchRightSidePanel('thread', $event)"
          @close="switchRightSidePanel()"
        />

        <bookmarked-messages-panel
          v-if="isBookmarkedMessagesPanel"
          :channel="channel"
          @openThreadPanel="switchRightSidePanel('thread', $event)"
          @close="switchRightSidePanel()"
        />
      </base-side-panel>
    </div>
  </div>
</template>

<script>
import BaseSidePanel from 'corteza-webapp-messaging/src/components/Panel/Base'
import ChannelsPanel from 'corteza-webapp-messaging/src/components/Panel/Channels'
import MembersPanel from 'corteza-webapp-messaging/src/components/Panel/Rightside/Members'
import ThreadPanel from 'corteza-webapp-messaging/src/components/Panel/Rightside/Thread'
import BookmarkedMessagesPanel from 'corteza-webapp-messaging/src/components/Panel/Rightside/BookmarkedMessages'
import PinnedMessagesPanel from 'corteza-webapp-messaging/src/components/Panel/Rightside/PinnedMessages'

export default {
  components: {
    BaseSidePanel,
    ChannelsPanel,
    MembersPanel,
    ThreadPanel,
    BookmarkedMessagesPanel,
    PinnedMessagesPanel,
  },

  props: {
    channel: {
      type: Object,
      default: () => ({}),
    },
  },

  data () {
    return {
      // UI control
      uiRightSidePanelThreadMessage: null,
      uiRightSidePanelContent: null,
      uiPinChannelSidePanel: true,
      uiHideChannelSidePanel: false,
      uiPinRightSidePanel: true,
      hideRightSidePanel: true,

      uiWide: undefined,
    }
  },

  computed: {
    uiHideRightSidePanel: {
      get () {
        return this.hideRightSidePanel
      },
      set (v) {
        this.hideRightSidePanel = v
        if (v) {
          this.uiRightSidePanelContent = null
        }
      },
    },

    isChannelPanelPinned () {
      return this.uiPinChannelSidePanel || this.$route.name === 'landing'
    },

    isLeftPanelDisableGestures () {
      return !this.uiHideRightSidePanel || this.isChannelPanelPinned
    },
    isMembersPanel () {
      return this.uiRightSidePanelContent === 'members'
    },
    isThreadPanel () {
      return this.uiRightSidePanelContent === 'thread'
    },
    isPinnedMessagesPanel () {
      return this.channel && this.uiRightSidePanelContent === 'pinnedMessages'
    },
    isBookmarkedMessagesPanel () {
      return this.uiRightSidePanelContent === 'bookmarkedMessages'
    },
  },

  created () {
    this.$bus.$on('Messenger/toggleChannelSidePanel', this.toggleChannelSidePanel)
    this.$bus.$on('Messenger/switchRightSidePanel', ({ type, e }) => this.switchRightSidePanel(type, e))

    this.$bus.$on('Messenger/uiWide', this.windowResizeHandler)
  },
  beforeDestroy () {
    this.$bus.$off('Messenger/toggleChannelSidePanel', this.toggleChannelSidePanel)
    this.$bus.$off('Messenger/switchRightSidePanel', ({ type, e }) => this.switchRightSidePanel(type, e))

    this.$bus.$off('Messenger/uiWide', this.windowResizeHandler)
  },

  methods: {
    windowResizeHandler (wide) {
      // We want to pin side panels when screen is wide enough.
      this.uiWide = wide
      this.uiPinChannelSidePanel = wide
      this.uiPinRightSidePanel = wide
    },

    switchRightSidePanel (panel = null, $event = {}) {
      if (panel !== 'thread' && panel === this.uiRightSidePanelContent) {
        this.uiRightSidePanelContent = null
      } else {
        this.uiRightSidePanelContent = panel
      }

      if (this.uiRightSidePanelContent && !this.uiWide) {
        // Close channels when opening right panel and screen is not wide enough
        this.uiHideChannelSidePanel = true
      }

      this.uiHideRightSidePanel = !this.uiRightSidePanelContent

      switch (panel) {
        case 'thread':
          this.uiRightSidePanelThreadMessage = $event.message
      }
    },

    toggleChannelSidePanel (state = null) {
      this.uiHideChannelSidePanel = state === null ? !this.uiHideChannelSidePanel : state

      if (!this.uiHideChannelSidePanel && !this.uiWide) {
        // Close right side when screen is not wide enough
        this.switchRightSidePanel(false)
      }
    },

    openQuickSearch () {
      this.$emit('openQuickSearch')
    },
    searchSubmit (e) {
      this.$emit('searchSubmit', e)
    },
  },

}
</script>

<style scoped lang="scss">

div.flex-grid {
  z-index: 1000;
  background-color: $light;
  height: 100vh;
  width: 100vw;
  display: flex;
  flex-flow: row nowrap;
}

</style>
