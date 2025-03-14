<template>
  <div class="w-full h-full p-8">
    <p class="uppercase text-xs font-semibold text-gray-300 mb-2">Display Settings</p>
    <div class="flex items-center py-3" @click="toggleEnableAltView">
      <div class="w-10 flex justify-center">
        <ui-toggle-switch v-model="settings.enableAltView" @input="saveSettings" />
      </div>
      <p class="pl-4">Alternative bookshelf view</p>
    </div>

    <p class="uppercase text-xs font-semibold text-gray-300 mb-2 mt-6">Playback Settings</p>
    <div v-if="!isiOS" class="flex items-center py-3" @click="toggleDisableAutoRewind">
      <div class="w-10 flex justify-center">
        <ui-toggle-switch v-model="settings.disableAutoRewind" @input="saveSettings" />
      </div>
      <p class="pl-4">Disable auto rewind</p>
    </div>
    <div class="flex items-center py-3" @click="toggleJumpBackwards">
      <div class="w-10 flex justify-center">
        <span class="material-icons text-4xl">{{ currentJumpBackwardsTimeIcon }}</span>
      </div>
      <p class="pl-4">Jump backwards time</p>
    </div>
    <div class="flex items-center py-3" @click="toggleJumpForward">
      <div class="w-10 flex justify-center">
        <span class="material-icons text-4xl">{{ currentJumpForwardTimeIcon }}</span>
      </div>
      <p class="pl-4">Jump forwards time</p>
    </div>

    <p v-if="!isiOS" class="uppercase text-xs font-semibold text-gray-300 mb-2 mt-6">Sleep Timer Settings</p>
    <div v-if="!isiOS" class="flex items-center py-3" @click="toggleDisableShakeToResetSleepTimer">
      <div class="w-10 flex justify-center">
        <ui-toggle-switch v-model="settings.disableShakeToResetSleepTimer" @input="saveSettings" />
      </div>
      <p class="pl-4">Disable shake to reset</p>
      <span class="material-icons-outlined ml-2" @click.stop="showInfo('disableShakeToResetSleepTimer')">info</span>
    </div>
  </div>
</template>

<script>
import { Dialog } from '@capacitor/dialog'

export default {
  data() {
    return {
      deviceData: null,
      settings: {
        disableAutoRewind: false,
        enableAltView: false,
        jumpForwardTime: 10,
        jumpBackwardsTime: 10,
        disableShakeToResetSleepTimer: false
      },
      settingInfo: {
        disableShakeToResetSleepTimer: {
          name: 'Disable shake to reset sleep timer',
          message: 'The sleep timer will start fading out when 30s is remaining. Shaking your device will reset the timer if it is within 30s OR has finished less than 2 mintues ago. Enable this setting to disable that feature.'
        }
      }
    }
  },
  computed: {
    isiOS() {
      return this.$platform === 'ios'
    },
    jumpForwardItems() {
      return this.$store.state.globals.jumpForwardItems || []
    },
    jumpBackwardsItems() {
      return this.$store.state.globals.jumpBackwardsItems || []
    },
    currentJumpForwardTimeIcon() {
      return this.jumpForwardItems[this.currentJumpForwardTimeIndex].icon
    },
    currentJumpForwardTimeIndex() {
      var index = this.jumpForwardItems.findIndex((jfi) => jfi.value === this.settings.jumpForwardTime)
      return index >= 0 ? index : 1
    },
    currentJumpBackwardsTimeIcon() {
      return this.jumpBackwardsItems[this.currentJumpBackwardsTimeIndex].icon
    },
    currentJumpBackwardsTimeIndex() {
      var index = this.jumpBackwardsItems.findIndex((jfi) => jfi.value === this.settings.jumpBackwardsTime)
      return index >= 0 ? index : 1
    }
  },
  methods: {
    showInfo(setting) {
      if (this.settingInfo[setting]) {
        Dialog.alert({
          title: this.settingInfo[setting].name,
          message: this.settingInfo[setting].message
        })
      }
    },
    toggleDisableShakeToResetSleepTimer() {
      this.settings.disableShakeToResetSleepTimer = !this.settings.disableShakeToResetSleepTimer
      this.saveSettings()
    },
    toggleDisableAutoRewind() {
      this.settings.disableAutoRewind = !this.settings.disableAutoRewind
      this.saveSettings()
    },
    toggleEnableAltView() {
      this.settings.enableAltView = !this.settings.enableAltView
      this.saveSettings()
    },
    toggleJumpForward() {
      var next = (this.currentJumpForwardTimeIndex + 1) % 3
      this.settings.jumpForwardTime = this.jumpForwardItems[next].value
      this.saveSettings()
    },
    toggleJumpBackwards() {
      var next = (this.currentJumpBackwardsTimeIndex + 4) % 3
      if (next > 2) return
      this.settings.jumpBackwardsTime = this.jumpBackwardsItems[next].value
      this.saveSettings()
    },
    async saveSettings() {
      const updatedDeviceData = await this.$db.updateDeviceSettings({ ...this.settings })
      console.log('Saved device data', updatedDeviceData)
      if (updatedDeviceData) {
        this.$store.commit('setDeviceData', updatedDeviceData)
        this.init()
      }
    },
    async init() {
      this.deviceData = await this.$db.getDeviceData()
      this.$store.commit('setDeviceData', this.deviceData)

      const deviceSettings = this.deviceData.deviceSettings || {}
      this.settings.disableAutoRewind = !!deviceSettings.disableAutoRewind
      this.settings.enableAltView = !!deviceSettings.enableAltView
      this.settings.jumpForwardTime = deviceSettings.jumpForwardTime || 10
      this.settings.jumpBackwardsTime = deviceSettings.jumpBackwardsTime || 10
      this.settings.disableShakeToResetSleepTimer = !!deviceSettings.disableShakeToResetSleepTimer
    }
  },
  mounted() {
    this.init()
  }
}
</script>