<script setup lang="ts">
import { ref } from 'vue'
import { useI18n } from 'vue-i18n'
import { useToast } from 'vue-toast-notification'
import { readConfigurationFile } from '@/shared/core/reader'
import Dropdown from '@components/dropdown.vue'
import Page from '@components/page.vue'
import PreferenceGroup from '@components/settings/preference-group.vue'
import SwitchPreference from '@components/settings/switch-preference.vue'
import predefinedConfigurations from '@shared/const/config'
import useConfiguration from '@stores/config'
import useSettingsStore from '@stores/settings'
import { message, open } from '@tauri-apps/api/dialog'

const $toast = useToast()
const { t } = useI18n()
const settingsStore = useSettingsStore()
const configurationStore = useConfiguration()

const config = ref(predefinedConfigurations[0].value)

const onCustomConfigurationChange = (enabled: boolean) => {
  if (!enabled) {
    configurationStore.revert()
    $toast.success(t('status.config-removed'))
  } else handleConfigurationChange(config.value)
}

const handleConfigurationChange = async (value: string) => {
  if (value !== 'custom') {
    settingsStore.updatePreference('predefinedConfiguration', value)
    configurationStore.change(value)
    config.value = value

    $toast.success(t('status.config-updated'))
  } else {
    const filePath = await open({
      filters: [
        {
          name: '*',
          extensions: ['json', 'txt']
        }
      ]
    })

    if (filePath !== null && !Array.isArray(filePath)) {
      try {
        const config = await readConfigurationFile(filePath)
        await configurationStore.replace(config)
      } catch (e) {
        await message(String(e))
      }
    } else {
      await message(t('no-file-selected-config'))
      settingsStore.updatePreference('customConfig', false)
    }
  }
}
</script>

<template>
  <page>
    <div>
      <h1 class="header">{{ t('settings.header') }}</h1>
    </div>
    <div class="mt-4 space-y-4">
      <preference-group title="settings.character.name">
        <switch-preference
          label="settings.character.position-prevention"
          preference-key="positionPrevention" />
        <switch-preference
          label="settings.character.uses-ideology-suffix"
          preference-key="usesIdeologySuffixOnToken" />
      </preference-group>
      <preference-group title="settings.logging.name">
        <switch-preference label="settings.logging.option-logging" preference-key="optionLogging" />
      </preference-group>
      <preference-group title="settings.configuration.name">
        <switch-preference
          label="settings.configuration.custom-config"
          preference-key="customConfig"
          @change="onCustomConfigurationChange" />
        <div className="flex items-center gap-4" v-if="settingsStore.getPreference('customConfig')">
          <legend class="text-sm font-medium shrink-0">
            {{ t('settings.configuration.predefined-configs') }}
          </legend>
          <div class="flex-1 space-y-2">
            <dropdown
              localise
              :options="predefinedConfigurations"
              v-model="config"
              @update:model-value="handleConfigurationChange" />
          </div>
        </div>
      </preference-group>
      <preference-group title="settings.interface.name">
        <switch-preference
          label="settings.interface.advisor-trait-input"
          preference-key="useInputForAdvisorTraitBox" />
      </preference-group>
    </div>
  </page>
</template>
@/shared/const/config
