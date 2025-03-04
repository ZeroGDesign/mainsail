<template>
    <v-dialog
        transition="dialog-bottom-transition"
        max-width="600"
        scrollable
        v-model="isOpen"
        :fullscreen="isMobile"
    >
        <template #activator="{ on, attrs }">
            <v-btn
                class="gcode-command-btn px-2 minwidth-0"
                color="grey darken-3"
                :small="isMini"
                v-bind="attrs"
                v-on="on">
                <v-icon>mdi-help</v-icon>
            </v-btn>
        </template>
        <template #default>
            <panel :title="$t('Console.CommandList')" icon="mdi-help" card-class="command-help-dialog" :margin-bottom="false">
                <template v-slot:buttons>
                    <v-btn icon @click="isOpen = false"><v-icon>mdi-close-thick</v-icon></v-btn>
                </template>
                <v-card-title>
                    <v-row>
                        <v-col>
                            <v-text-field
                                v-model="cmdListSearch"
                                label="Search"
                                outlined
                                hide-details
                                clearable
                                dense
                            ></v-text-field>
                        </v-col>
                    </v-row>
                </v-card-title>
                <v-divider></v-divider>
                <perfect-scrollbar :class="'command-help-content '+(isMobile ? '' : 'height300')">
                    <v-card-text class="pt-0">
                        <v-row>
                            <v-col>
                                <v-list>
                                    <v-list-item
                                        v-for="cmd of helplistFiltered"
                                        :key="cmd.commandLow"
                                        class="px-0"
                                        two-line>
                                        <v-list-item-content class="px-0">
                                            <v-list-item-title class="blue--text font-weight-bold cursor-pointer" @click="$emit('onCommand', cmd.command); isOpen = false">{{ cmd.command }}</v-list-item-title>
                                            <v-list-item-subtitle class="text-wrap">{{ cmd.description }}</v-list-item-subtitle>
                                        </v-list-item-content>
                                    </v-list-item>
                                </v-list>
                            </v-col>
                        </v-row>
                    </v-card-text>
                </perfect-scrollbar>
            </panel>
        </template>
    </v-dialog>
</template>

<script lang="ts">
import BaseMixin from './mixins/base'
import {CommandHelp} from '@/store/printer/types'
import {Mixins, Prop, Watch} from 'vue-property-decorator'
import Component from 'vue-class-component'
import Panel from '@/components/ui/Panel.vue'
@Component({
    components: {Panel}
})
export default class CommandHelpModal extends Mixins(BaseMixin) {
    @Prop({ required: false, default: false }) isMini!: boolean;
    cmdListSearch = '';
    isOpen = false;

    get helplist(): CommandHelp[] {
        return this.$store.state.printer.helplist ?? []
    }

    get helplistFiltered(): CommandHelp[] {
        return this.helplist
            .filter(cmd => typeof(cmd.description) === 'string' && (!this.cmdListSearch || cmd.commandLow.includes(this.cmdListSearch.toLowerCase())))
            .sort((a, b) => a.commandLow.localeCompare(b.commandLow))
    }

    @Watch('isOpen')
    onIsOpen(val: boolean): void {
        if (!val) {
            this.cmdListSearch = ''
        }
    }
}
</script>

<style lang="scss" scoped>
    .command-help-content {
        overflow-x: hidden;

        &.height300 {
            height: 300px;
        }
    }
</style>
