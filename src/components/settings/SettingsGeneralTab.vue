<template>
    <div>
        <v-card flat>
            <v-card-text>
                <settings-row :title="$t('Settings.GeneralTab.PrinterName')">
                    <v-text-field v-model="printerName" hide-details outlined dense ></v-text-field>
                </settings-row>
                <v-divider class="my-2"></v-divider>
                <settings-row :title="$t('Settings.GeneralTab.Language')">
                    <v-select v-model="currentLanguage" :items="availableLanguages" hide-details outlined dense></v-select>
                </settings-row>
                <v-divider class="my-2"></v-divider>
                <settings-row :title="$t('Settings.GeneralTab.CalcEstimateTime')" :sub-title="$t('Settings.GeneralTab.CalcEstimateTimeDescription')">
                    <v-select v-model="calcEstimateTime" :items="calcEstimateItems" multiple hide-details dense outlined></v-select>
                </settings-row>
                <v-divider class="my-2"></v-divider>
                <settings-row :title="$t('Settings.GeneralTab.CalcEtaTime')" :sub-title="$t('Settings.GeneralTab.CalcEtaTimeDescription')">
                    <v-select v-model="calcEtaTime" :items="calcEtaTimeItems" multiple hide-details dense outlined></v-select>
                </settings-row>
                <v-divider class="my-2"></v-divider>
                <settings-row :title="$t('Settings.GeneralTab.BoolBigThumbnail')" :sub-title="$t('Settings.GeneralTab.BoolBigThumbnailDescription')" :dynamicSlotWidth="true">
                    <v-switch v-model="boolBigThumbnail" hide-details class="mt-0"></v-switch>
                </settings-row>
                <v-divider class="my-2"></v-divider>
                <settings-row :title="$t('Settings.GeneralTab.DisplayCANCEL_PRINT')" :sub-title="$t('Settings.GeneralTab.DisplayCANCEL_PRINTDescription')" :dynamicSlotWidth="true">
                    <v-switch v-model="displayCancelPrint" hide-details class="mt-0"></v-switch>
                </settings-row>
                <v-divider class="my-2"></v-divider>
                <settings-row :title="$t('Settings.GeneralTab.DisplayZOffset')" :sub-title="$t('Settings.GeneralTab.DisplayZOffsetDescription')" :dynamicSlotWidth="true">
                    <v-switch v-model="displayZOffsetStandby" hide-details class="mt-0"></v-switch>
                </settings-row>
                <v-divider class="my-2"></v-divider>
                <settings-row :title="$t('Settings.GeneralTab.ConfirmOnEmergencyStop')" :sub-title="$t('Settings.GeneralTab.ConfirmOnEmergencyStopDescription')" :dynamicSlotWidth="true">
                    <v-switch v-model="confirmOnEmergencyStop" hide-details class="mt-0"></v-switch>
                </settings-row>
                <v-divider class="my-2"></v-divider>
                <settings-row :title="$t('Settings.GeneralTab.ConfirmOnPowerDeviceChange')" :sub-title="$t('Settings.GeneralTab.ConfirmOnPowerDeviceChangeDescription')" :dynamicSlotWidth="true">
                    <v-switch v-model="confirmOnPowerDeviceChange" hide-details class="mt-0"></v-switch>
                </settings-row>
                <v-divider class="my-2"></v-divider>
                <settings-row :title="$t('Settings.GeneralTab.FactoryReset')" :dynamicSlotWidth="true">
                    <v-btn @click="dialogResetMainsail=true" color="error" small>{{ $t('Settings.GeneralTab.FactoryReset') }}</v-btn>
                </settings-row>
            </v-card-text>
        </v-card>
        <v-dialog v-model="dialogResetMainsail" persistent :width="300">
            <v-card dark>
                <v-toolbar flat dense color="primary">
                    <v-toolbar-title>
                    <span class="subheading">
                        <v-icon class="mdi mdi-help-circle" left></v-icon> {{ $t('Settings.GeneralTab.FactoryReset') }}
                    </span>
                    </v-toolbar-title>
                    <v-spacer></v-spacer>
                    <v-btn small class="minwidth-0" @click="dialogResetMainsail = false"><v-icon small>mdi-close-thick</v-icon></v-btn>
                </v-toolbar>
                <v-card-text class="pt-3">
                    <v-container class="pb-0">

                        <v-row>
                            <v-col>
                                <p class="text-center mb-0">{{ $t('Settings.GeneralTab.FactoryInfo') }}</p>
                            </v-col>
                        </v-row>

                        <v-row>
                            <v-col class="text-center">
                                <v-btn
                                    color="red"
                                    @click="resetMainsail"
                                >
                                    {{ $t('Settings.GeneralTab.ResetMainsail') }}
                                </v-btn>
                            </v-col>
                        </v-row>
                    </v-container>
                </v-card-text>
            </v-card>
        </v-dialog>
    </div>
</template>

<script lang="ts">

import Component from 'vue-class-component'
import { Mixins } from 'vue-property-decorator'
import BaseMixin from '@/components/mixins/base'
import SettingsRow from '@/components/settings/SettingsRow.vue'
@Component({
    components: {SettingsRow}
})
export default class SettingsGeneralTab extends Mixins(BaseMixin) {
    private dialogResetMainsail = false

    get printerName() {
        return this.$store.state.gui.general.printername
    }

    set printerName(newVal) {
        this.$store.dispatch('gui/saveSetting', {name: 'general.printername', value: newVal})
    }

    get currentLanguage() {
        return this.$store.state.gui.general.language
    }

    set currentLanguage(newVal) {
        this.$store.dispatch('gui/saveSetting', {name: 'general.language', value: newVal })
    }

    get availableLanguages() {
        // eslint-disable-next-line no-undef
        const locales = require.context('@/locales', true, /[A-Za-z0-9-_,\s]+\.json$/i)
        const languages: any = []

        locales.keys().map((key: string) => {
            const langKey = key.slice(2, key.lastIndexOf('.'))

            languages.push({
                text: locales(key).title,
                value: langKey
            })
        })

        return languages
    }

    get calcEstimateItems() {
        return [
            { value: 'file', text: 'File' },
            { value: 'filament', text: 'Filament' },
        ]
    }

    get calcEstimateTime() {
        return this.$store.state.gui.general.calcEstimateTime
    }

    set calcEstimateTime(newVal) {
        this.$store.dispatch('gui/saveSetting', {name: 'general.calcEstimateTime', value: newVal })
    }

    get calcEtaTimeItems() {
        return [
            { value: 'file', text: 'File' },
            { value: 'filament', text: 'Filament' },
            { value: 'slicer', text: 'Slicer' },
        ]
    }

    get calcEtaTime() {
        return this.$store.state.gui.general.calcEtaTime
    }

    set calcEtaTime(newVal) {
        this.$store.dispatch('gui/saveSetting', {name: 'general.calcEtaTime', value: newVal })
    }

    get boolBigThumbnail() {
        return this.$store.state.gui.dashboard.boolBigThumbnail
    }

    set boolBigThumbnail(newVal) {
        this.$store.dispatch('gui/saveSetting', {name: 'dashboard.boolBigThumbnail', value: newVal })
    }

    get displayCancelPrint() {
        return this.$store.state.gui.general.displayCancelPrint
    }

    set displayCancelPrint(newVal) {
        this.$store.dispatch('gui/saveSetting', {name: 'general.displayCancelPrint', value: newVal })
    }

    get displayZOffsetStandby() {
        return this.$store.state.gui.general.displayZOffsetStandby
    }

    set displayZOffsetStandby(newVal) {
        this.$store.dispatch('gui/saveSetting', {name: 'general.displayZOffsetStandby', value: newVal })
    }

    get confirmOnEmergencyStop() {
        return this.$store.state.gui.general.confirmOnEmergencyStop
    }

    set confirmOnEmergencyStop(newVal) {
        this.$store.dispatch('gui/saveSetting', {name: 'general.confirmOnEmergencyStop', value: newVal })
    }

    get confirmOnPowerDeviceChange() {
        return this.$store.state.gui.general.confirmOnPowerDeviceChange
    }

    set confirmOnPowerDeviceChange(newVal) {
        this.$store.dispatch('gui/saveSetting', {name: 'general.confirmOnPowerDeviceChange', value: newVal })
    }

    resetMainsail() {
        this.$store.dispatch('gui/resetMoonrakerDB')
        this.dialogResetMainsail = false
    }
}
</script>