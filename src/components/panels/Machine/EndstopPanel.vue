<template>
    <panel :title="$t('Machine.EndstopPanel.Endstops')" icon="mdi-arrow-expand-vertical" card-class="machine-endstop-panel" :collapsible="true">
        <v-card-text class="pb-0">
            <v-container px-0 py-0>
                <template v-if="Object.keys(endstops).length">
                    <v-row v-for="key of Object.keys(endstops)" v-bind:key="key">
                        <v-col class="py-1">
                            <label class="mt-1 d-inline-block">{{ $t('Machine.EndstopPanel.Endstop')}} <b>{{ key.toUpperCase() }}</b></label>
                            <v-chip class="float-right" small :color="endstops[key] === 'open' ? 'green' : 'red' " text-color="white">
                                <template v-if="endstops[key] === 'open'">
                                    {{ $t('Machine.EndstopPanel.open')}}
                                </template>
                                <template v-else>
                                    {{ $t('Machine.EndstopPanel.TRIGGERED')}}
                                </template>
                            </v-chip>
                        </v-col>
                    </v-row>
                    <v-row v-if="existProbe">
                        <v-col class="py-1">
                            <label class="mt-1 d-inline-block">Probe</label>
                            <v-chip class="float-right" small :color="probe ? 'red' : 'green' " text-color="white">
                            <template v-if="probe">
                                {{ $t('Machine.EndstopPanel.TRIGGERED')}}
                            </template>
                            <template v-else>
                                {{ $t('Machine.EndstopPanel.open')}}
                            </template>
                            </v-chip>
                        </v-col>
                    </v-row>
                </template>
                <template v-else>
                    <v-row>
                        <v-col>
                            <p>{{ $t('Machine.EndstopPanel.EndstopInfo')}}</p>
                        </v-col>
                    </v-row>
                </template>
            </v-container>
        </v-card-text>
        <v-card-actions class="pt-3">
            <v-spacer></v-spacer>
            <v-btn icon @click="syncEndstops" :loading="loadings.includes('queryEndstops')">
                <v-icon>mdi-sync</v-icon>
            </v-btn>
        </v-card-actions>
    </panel>
</template>

<script lang="ts">

import {Component, Mixins} from 'vue-property-decorator'
import BaseMixin from '../../mixins/base'
import Panel from '@/components/ui/Panel.vue'
@Component({
    components: {Panel}
})
export default class EndstopPanel extends Mixins(BaseMixin) {
    public sortEndstops: any = {}

    get endstops() {
        return this.$store.state.printer.endstops ?? {}
    }

    get existProbe () {
        return ('probe' in this.$store.state.printer.configfile.settings)
    }

    get probe () {
        if (
            'probe' in this.$store.state.printer &&
            'last_query' in this.$store.state.printer.probe
        ) return this.$store.state.printer.probe.last_query

        return false
    }

    syncEndstops() {
        this.$socket.emit('printer.query_endstops.status', { }, { action: 'printer/getEndstopStatus', loading: 'queryEndstops' })
        if (this.existProbe) {
            window.console.log('exist probe')
            this.$store.dispatch('server/addEvent', { message: 'QUERY_PROBE', type: 'command' })
            this.$socket.emit('printer.gcode.script', { script: 'QUERY_PROBE' })
        }
    }
}

/*    import { mapState } from 'vuex'

    export default {
        created() {
            this.getEndstops();
        },
        methods: {
            ,
            getEndstops() {
                this.sortEndstops = {};

                let keys = Object.keys(this.endstops);
                keys.sort();

                for (let i = 0; i < keys.length; i++) {
                    let k = keys[i];
                    this.sortEndstops[k] = this.endstops[k];
                }
            }
        },
        watch: {
            endstops: function() {
                this.getEndstops();
            }
        }
    }*/
</script>