<style scoped>

</style>

<template>
    <div v-if="displayMenuPoint">
        <li :class="currentPage === '/allPrinters' ? 'nav-item active' : 'nav-item '">
            <div
                class="nav-link "
                @click.prevent
                @click="switchToPrinters"
                role="button"
            >
                <v-icon>mdi-view-dashboard-outline</v-icon>
                <span class="nav-title">{{ $t("App.Printers")}}</span>

                <v-menu bottom :offset-x="true">
                    <template v-slot:activator="{ on, attrs }">
                        <v-icon class="nav-arrow right" v-bind="attrs" v-on="on" >mdi-chevron-down</v-icon>
                    </template>

                    <v-list dense>
                        <v-list-item two-line v-for="printer in printers" v-bind:key="printer._namespace" @click="changePrinter(printer)" :disabled="!printer.socket.isConnected" link>
                            <v-list-item-content>
                                <v-list-item-title>{{ getPrinterName(printer._namespace) }}</v-list-item-title>
                                <v-list-item-subtitle>{{ getPrinterDescription(printer)}}</v-list-item-subtitle>
                            </v-list-item-content>
                        </v-list-item>
                    </v-list>
                </v-menu>
            </div>
            <v-item-group class="v-btn-toggle mx-4 d-block row" name="printers" v-if="false">
                <v-btn class="col" color="primary" @click="switchToPrinters">{{ $t("App.Printers")}}</v-btn>
            </v-item-group>
        </li>
        <v-divider class="my-4"></v-divider>
    </div>
</template>

<script lang="ts">

import {Component, Mixins} from 'vue-property-decorator'
import BaseMixin from './mixins/base'
import router from '@/plugins/router'
import {FarmPrinterState} from '@/store/farm/printer/types'

@Component
export default class TheSidebarPrinterMenu extends Mixins(BaseMixin) {

    get displayMenuPoint() {
        return (this.remoteMode && this.countPrinters > 1) || (!this.remoteMode && this.countPrinters)
    }

    get printers() {
        return this.$store.getters['farm/getPrinters']
    }

    get countPrinters() {
        return this.$store.getters['farm/countPrinters']
    }

    get currentPage() {
        return this.$route.fullPath
    }

    switchToPrinters() {
        router.push('/allPrinters')
    }

    getPrinterName(namespace: string) {
        return this.$store.getters['farm/'+namespace+'/getPrinterName']
    }

    getPrinterDescription(printer: FarmPrinterState) {
        return this.$store.getters['farm/'+printer._namespace+'/getStatus']
    }

    changePrinter(printer: FarmPrinterState) {
        if (printer.socket.isConnected) {
            this.$store.dispatch('changePrinter', { printer: printer._namespace })
        }
    }
}
</script>