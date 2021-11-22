<template>
    <!-- Group List -->
    <Draggable
        v-model="$root.publicGroupList"
        :disabled="!editMode"
        item-key="id"
        :animation="100"
    >
        <template #item="group">
            <div class="mb-5 ">
                <!-- Group Title -->
                <h2 class="group-title">
                    <font-awesome-icon v-if="editMode && showGroupDrag" icon="arrows-alt-v" class="action drag me-3" />
                    <font-awesome-icon v-if="editMode" icon="times" class="action remove me-3" @click="removeGroup(group.index)" />
                    <Editable v-model="group.element.name" :contenteditable="editMode" tag="span" />
                </h2>

                <div class="shadow-box monitor-list mt-4 position-relative">
                    <div v-if="group.element.monitorList.length === 0" class="text-center no-monitor-msg">
                        {{ $t("No Monitors") }}
                    </div>

                    <!-- Monitor List -->
                    <!-- animation is not working, no idea why -->
                    <Draggable
                        v-model="group.element.monitorList"
                        class="monitor-list"
                        group="same-group"
                        :disabled="!editMode"
                        :animation="100"
                        item-key="id"
                    >
                        <template #item="monitor">
                            <div class="item">
                                <div class="row">
                                    <details>
                                        <summary class="summary">
                                            <div class="col-9 col-md-8 small-padding">
                                                <div class="info">
                                                    <font-awesome-icon v-if="editMode" icon="arrows-alt-v" class="action drag me-3" />
                                                    <font-awesome-icon v-if="editMode" icon="times" class="action remove me-3" @click="removeMonitor(group.index, monitor.index)" />

                                                    <Uptime :monitor="monitor.element" type="24" :pill="true" />
                                                    {{ monitor.element.name }}
                                                </div>
                                            </div>
                                            <div :key="$root.userHeartbeatBar" class="col-3 col-md-4">
                                                <HeartbeatBar size="small" :monitor-id="monitor.element.id" />
                                            </div>
                                        </summary>
                                        <!-- Monitor Details -->
                                        <!-- Ping Detail -->
                                        <div class="shadow-box big-padding text-center stats row">
                                            <a v-if="monitor.element.type == 'http'" :href="monitor.element.url">{{ monitor.element.url }}</a>
                                            <div class="col">
                                                <h4>Ping</h4>
                                                <p>({{ $t("Current") }})</p>
                                                <span class="num">
                                                    <a href="#" @click.prevent="showPingChartBox = !showPingChartBox">
                                                        <CountUp :value="$root.heartbeatList[monitor.element.id].at(-1).ping" />
                                                    </a>
                                                </span>
                                            </div>
                                            <!-- Uptime 24h Detail -->
                                            <div class="col">
                                                <h4>Uptime</h4>
                                                <p>({{ $t("24 Hour") }})</p>
                                                <span class="num">
                                                    <Uptime :monitor="monitor.element" type="24" />
                                                </span>
                                            </div>
                                            <!-- Uptime 30d Detail -->
                                            <div class="col">
                                                <h4>Uptime</h4>
                                                <p>({{ $t("30 Day") }})</p>
                                                <span class="num">
                                                    <Uptime :value="monitor.element" type="720" />
                                                </span>
                                            </div>
                                        </div>
                                        <!-- Ping Chart -->
                                        <div v-if="showPingChartBox" class="shadow-box big-padding text-center ping-chart-wrapper">
                                            <div class="row">
                                                <div class="col">
                                                    <PingChart :monitor-id="monitor.element.id" />
                                                </div>
                                            </div>
                                        </div>
                                    </details>
                                </div>
                            </div>
                        </template>
                    </Draggable>
                </div>
            </div>
        </template>
    </Draggable>
</template>

<script>
import { defineAsyncComponent } from "vue";
import Draggable from "vuedraggable";
import HeartbeatBar from "./HeartbeatBar.vue";
import CountUp from "./CountUp.vue";
import Uptime from "./Uptime.vue";
const PingChart = defineAsyncComponent(() => import("./PingChart.vue"));

export default {
    components: {
        Draggable,
        HeartbeatBar,
        Uptime,
        CountUp,
        PingChart,
    },
    props: {
        editMode: {
            type: Boolean,
            required: true,
        },
    },
    data() {
        return {
            showPingChartBox: true,
        };
    },
    computed: {
        showGroupDrag() {
            return (this.$root.publicGroupList.length >= 2);
        }
    },
    created() {

    },
    methods: {
        removeGroup(index) {
            this.$root.publicGroupList.splice(index, 1);
        },

        removeMonitor(groupIndex, index) {
            this.$root.publicGroupList[groupIndex].monitorList.splice(index, 1);
        },
    }
};
</script>

<style lang="scss" scoped>
@import "../assets/vars";

.summary {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 0.5rem 1rem;
    cursor: pointer;
}

.no-monitor-msg {
    position: absolute;
    width: 100%;
    top: 20px;
    left: 0;
}

.monitor-list {
    min-height: 46px;
}

.flip-list-move {
    transition: transform 0.5s;
}

.no-move {
    transition: transform 0s;
}

.drag {
    color: #bbb;
    cursor: grab;
}

.remove {
    color: $danger;
}

.group-title {
    span {
        display: inline-block;
        min-width: 15px;
    }
}

.mobile {
    .item {
        padding: 13px 0 10px 0;
    }
}

</style>
