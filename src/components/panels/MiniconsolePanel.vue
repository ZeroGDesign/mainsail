<style scoped lang="scss">

    .consoleTable {
        border-top: 1px solid rgba(255, 255, 255, 0.12);
    }

    .gcode-command-field {
        font-family: monospace;
    }
</style>

<template>
    <panel
        v-if="socketIsConnected"
        icon="mdi-console-line"
        :title="$t('Panels.MiniconsolePanel.Headline')"
        :collapsible="true"
        card-class="miniconsole-panel"
    >
        <v-card-text :class="consoleDirection === 'table' ? 'order-1' : 'order-2'">
            <v-row>
                <v-col>
                    <v-textarea
                        v-model="gcode"
                        :items="items"
                        :label="$t('Panels.MiniconsolePanel.SendCode')"
                        solo
                        class="gcode-command-field"
                        ref="gcodeCommandField"
                        autocomplete="off"
                        no-resize
                        auto-grow
                        :rows="rows"
                        @keydown.enter.prevent.stop="doSend"
                        @keyup.up="onKeyUp"
                        @keyup.down="onKeyDown"
                        @keydown.tab="getAutocomplete"
                        hide-details
                        outlined
                        dense
                        append-icon="mdi-send"
                        @click:append="doSend"
                    ></v-textarea>
                </v-col>
                <v-col class="col-auto">
                    <command-help-modal @onCommand="gcode = $event" ></command-help-modal>
                    <v-menu :offset-y="true" :close-on-content-click="false" :title="$t('Panels.MiniconsolePanel.SetupConsole')">
                        <template v-slot:activator="{ on, attrs }">
                            <v-btn class="px-2 minwidth-0" color="grey darken-3 ml-3" v-bind="attrs" v-on="on"><v-icon>mdi-filter</v-icon></v-btn>
                        </template>
                        <v-list>
                            <v-list-item class="minHeight36">
                                <v-checkbox class="mt-0" v-model="hideWaitTemperatures" hide-details :label="$t('Panels.MiniconsolePanel.HideTemperatures')"></v-checkbox>
                            </v-list-item>
                            <v-list-item class="minHeight36" v-for="(filter, index) in customFilters" v-bind:key="index">
                                <v-checkbox class="mt-0" v-model="filter.bool" @change="toggleFilter(filter)" hide-details :label="filter.name"></v-checkbox>
                            </v-list-item>
                        </v-list>
                    </v-menu>
                </v-col>
            </v-row>
        </v-card-text>
        <v-card-text :class="(consoleDirection === 'table' ? 'order-2' : 'order-1') + ' pa-0'">
            <v-row>
                <v-col>
                    <perfect-scrollbar ref="miniConsoleScroll" :style="'height: '+consoleHeight+'px;'">
                        <console-table ref="console"
                                       :events="events"
                                       :is-mini="true"
                                       @command-click="commandClick"
                        />
                        <v-divider></v-divider>
                    </perfect-scrollbar>
                </v-col>
            </v-row>
        </v-card-text>
    </panel>
</template>

<script lang="ts">
import { strLongestEqual, reverseString} from '@/plugins/helpers'
import {Component, Mixins, Watch} from 'vue-property-decorator'
import BaseMixin from '@/components/mixins/base'
import {CommandHelp, VTextareaType} from '@/store/printer/types'
import ConsoleTable from '@/components/console/ConsoleTable.vue'
import CommandHelpModal from '@/components/CommandHelpModal.vue'
import Vue from 'vue'
import Panel from '@/components/ui/Panel.vue'

@Component({
    components: {
        Panel,
        ConsoleTable,
        CommandHelpModal
    }
})
export default class MiniconsolePanel extends Mixins(BaseMixin) {
    $refs!: {
        gcodeCommandField: VTextareaType,
        console: ConsoleTable,
        miniConsoleScroll: any
    }

    private gcode = ''
    private lastCommands: string[] = []
    private lastCommandNumber: number | null = null
    private items = [];
    private cmdListSearch: string | null = null

    get helplist(): CommandHelp[] {
        return this.$store.state.printer.helplist ?? []
    }

    get consoleDirection() {
        return this.$store.state.gui.console.direction ?? 'table'
    }

    get consoleHeight() {
        return this.$store.state.gui.console.height ?? 300
    }

    get events() {
        return this.$store.getters['server/getConsoleEvents'](this.consoleDirection === 'table', 250)
    }

    @Watch('events')
    eventsChanged() {
        if (this.consoleDirection === 'shell') this.scrollToBottom()
    }

    get hideWaitTemperatures(): boolean {
        return this.$store.state.gui.console.hideWaitTemperatures
    }

    set hideWaitTemperatures(newVal) {
        this.$socket.emit('server.database.post_item', { namespace: 'mainsail', key: 'console.hideWaitTemperatures', value: newVal }, { action: 'gui/updateDataFromDB' })
    }

    get customFilters(): any[] {
        return this.$store.state.gui.console.customFilters
    }

    get rows(): number {
        return this.gcode?.split('\n').length ?? 1
    }

    commandClick(msg: string): void {
        this.gcode = msg

        this.$nextTick(() => {
            this.$refs.gcodeCommandField.focus()
        })
    }

    doSend(cmd: KeyboardEvent) {
        if (!cmd.shiftKey) {
            if (this.gcode !== '') {
                this.$store.dispatch('printer/sendGcode', this.gcode)
                this.lastCommands.push(this.gcode)
                this.gcode = ''
                this.lastCommandNumber = null
                setTimeout(() => {
                    this.$refs.console.$el.scroll({
                        top: 0,
                        left: 0,
                        behavior: 'smooth'
                    })
                }, 20)
            }
        } else {
            this.gcode += '\n'
        }
    }

    onKeyUp(): void {
        if (this.lastCommandNumber === null && this.lastCommands.length) {
            this.lastCommandNumber = this.lastCommands.length - 1
            this.gcode = this.lastCommands[this.lastCommandNumber]
        } else if (this.lastCommandNumber && this.lastCommandNumber > 0) {
            this.lastCommandNumber--
            this.gcode = this.lastCommands[this.lastCommandNumber]
        }
    }

    onKeyDown(): void {
        if (this.lastCommandNumber !== null && this.lastCommandNumber < (this.lastCommands.length - 1)) {
            this.lastCommandNumber++
            this.gcode = this.lastCommands[this.lastCommandNumber]
        } else if (this.lastCommandNumber !== null && this.lastCommandNumber === (this.lastCommands.length - 1)) {
            this.lastCommandNumber = null
            this.gcode = ''
        }
    }

    getAutocomplete(e: Event): void {
        e.preventDefault()
        if (this.gcode.length) {
            let check = this.gcode.toLowerCase()
            const textarea = this.$refs.gcodeCommandField.$refs.input
            const sentence = textarea.value
            const len = sentence.length
            const pos = textarea.selectionStart
            const currentLinePos = len - reverseString(sentence).indexOf('\n', len - pos)
            const currentEndPos = sentence.indexOf('\n', currentLinePos) > -1 ? sentence.indexOf('\n', currentLinePos) - 1 : Number.MAX_SAFE_INTEGER
            if (this.rows > 1) {
                check = sentence.substr(currentLinePos, currentEndPos - currentLinePos)
            }
            let commands = this.helplist.filter((element) => element.commandLow.startsWith(check.toLowerCase()))
            if (commands?.length === 1) {
                if (this.rows > 1) {
                    this.gcode = this.gcode.replace(check, commands[0].command)
                } else {
                    this.gcode = commands[0].command
                }
            } else if(commands?.length > 1) {
                let commands = this.helplist.filter((element) => element.commandLow.startsWith(check.toLowerCase()))
                if (this.rows > 1) {
                    this.gcode = this.gcode.replace(check, commands.reduce((acc, val) => {
                        return strLongestEqual(acc, val.command)
                    }, commands[0].command))
                } else {
                    this.gcode = commands.reduce((acc, val) => {
                        return strLongestEqual(acc, val.command)
                    }, commands[0].command)
                }
                if (commands && commands.length) {
                    let output = ''
                    commands.forEach(command => output += '<a class="command blue--text font-weight-bold">'+command.command+'</a>: '+command.description+'<br />')

                    this.$store.dispatch('server/addEvent', { message: output, type: 'autocomplete' })
                }
            }
        }
        this.$refs.gcodeCommandField.focus()
    }

    toggleFilter(filter: string) {
        this.$store.dispatch('gui/updateConsoleFilter',  filter)
    }

    mounted() {
        if (this.consoleDirection === 'shell') this.scrollToBottom()
    }

    scrollToBottom() {
        this.$nextTick(() => {
            if (this.$refs.miniConsoleScroll) {
                const perfectScroll = ((this.$refs.miniConsoleScroll as Vue).$el as HTMLDivElement)
                perfectScroll.scrollTop = perfectScroll.scrollHeight
            }
        })
    }
}
</script>
