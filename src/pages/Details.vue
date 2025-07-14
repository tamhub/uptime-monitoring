<template>
    <transition name="slide-fade" appear>
        <div v-if="monitor" class="printable-content">
            <router-link v-if="group !== ''" :to="monitorURL(monitor.parent)"> {{ group }}</router-link>
            <h1>
                {{ monitor.name }}
                <div class="monitor-id">
                    <div class="hash">#</div>
                    <div>{{ monitor.id }}</div>
                </div>
            </h1>
            <p v-if="monitor.description" v-html="descriptionHTML"></p>
            <div class="d-flex">
                <div class="tags">
                    <Tag v-for="tag in monitor.tags" :key="tag.id" :item="tag" :size="'sm'" />
                </div>
            </div>
            <p class="url">
                <a v-if="monitor.type === 'http' || monitor.type === 'keyword' || monitor.type === 'json-query' || monitor.type === 'mp-health' || monitor.type === 'real-browser' " :href="monitor.url" target="_blank" rel="noopener noreferrer">{{ filterPassword(monitor.url) }}</a>
                <span v-if="monitor.type === 'port'">TCP Port {{ monitor.hostname }}:{{ monitor.port }}</span>
                <span v-if="monitor.type === 'ping'">Ping: {{ monitor.hostname }}</span>
                <span v-if="monitor.type === 'keyword'">
                    <br>
                    <span>{{ $t("Keyword") }}: </span>
                    <span class="keyword">{{ monitor.keyword }}</span>
                    <span v-if="monitor.invertKeyword" alt="Inverted keyword" class="keyword-inverted"> ↧</span>
                </span>
                <span v-if="monitor.type === 'json-query'">
                    <br>
                    <span>{{ $t("Json Query") }}:</span> <span class="keyword">{{ monitor.jsonPath }}</span>
                    <br>
                    <span>{{ $t("Expected Value") }}:</span> <span class="keyword">{{ monitor.expectedValue }}</span>
                </span>
                <span v-if="monitor.type === 'dns'">[{{ monitor.dns_resolve_type }}] {{ monitor.hostname }}
                    <br>
                    <span>{{ $t("Last Result") }}:</span> <span class="keyword">{{ monitor.dns_last_result }}</span>
                </span>
                <span v-if="monitor.type === 'docker'">Docker container: {{ monitor.docker_container }}</span>
                <span v-if="monitor.type === 'gamedig'">Gamedig - {{ monitor.game }}: {{ monitor.hostname }}:{{ monitor.port }}</span>
                <span v-if="monitor.type === 'grpc-keyword'">gRPC - {{ filterPassword(monitor.grpcUrl) }}
                    <br>
                    <span>{{ $t("Keyword") }}:</span> <span class="keyword">{{ monitor.keyword }}</span>
                </span>
                <span v-if="monitor.type === 'mongodb'">{{ filterPassword(monitor.databaseConnectionString) }}</span>
                <span v-if="monitor.type === 'mqtt'">MQTT: {{ monitor.hostname }}:{{ monitor.port }}/{{ monitor.mqttTopic }}</span>
                <span v-if="monitor.type === 'mysql'">{{ filterPassword(monitor.databaseConnectionString) }}</span>
                <span v-if="monitor.type === 'postgres'">{{ filterPassword(monitor.databaseConnectionString) }}</span>
                <span v-if="monitor.type === 'push'">Push: <a :href="pushURL" target="_blank" rel="noopener noreferrer">{{ pushURL }}</a></span>
                <span v-if="monitor.type === 'radius'">Radius: {{ filterPassword(monitor.hostname) }}</span>
                <span v-if="monitor.type === 'redis'">{{ filterPassword(monitor.databaseConnectionString) }}</span>
                <span v-if="monitor.type === 'sqlserver'">SQL Server: {{ filterPassword(monitor.databaseConnectionString) }}</span>
                <span v-if="monitor.type === 'steam'">Steam Game Server: {{ monitor.hostname }}:{{ monitor.port }}</span>
            </p>

            <div class="functions">
                <div class="btn-group" role="group">
                    <button v-if="monitor.active" class="btn btn-normal" @click="pauseDialog">
                        <font-awesome-icon icon="pause" /> {{ $t("Pause") }}
                    </button>
                    <button v-if="! monitor.active" class="btn btn-primary" :disabled="monitor.forceInactive" @click="resumeMonitor">
                        <font-awesome-icon icon="play" /> {{ $t("Resume") }}
                    </button>
                    <router-link :to=" '/edit/' + monitor.id " class="btn btn-normal">
                        <font-awesome-icon icon="edit" /> {{ $t("Edit") }}
                    </router-link>
                    <router-link :to=" '/clone/' + monitor.id " class="btn btn-normal">
                        <font-awesome-icon icon="clone" /> {{ $t("Clone") }}
                    </router-link>
                    <button class="btn btn-normal text-danger" @click="deleteDialog">
                        <font-awesome-icon icon="trash" /> {{ $t("Delete") }}
                    </button>
                </div>

                <button class="btn btn-primary" @click="downloadPDF">
                    <font-awesome-icon icon="file-pdf" /> {{ $t("Download PDF") }}
                </button>
            </div>

            <div class="shadow-box heartbeat-bar">
                <div class="row heartbeat-bar-row">
                    <div class="col-md-8 heartbeat-bar-col-1">
                        <HeartbeatBar :monitor-id="monitor.id" />
                        <span class="word">{{ $t("checkEverySecond", [ monitor.interval ]) }}</span>
                    </div>
                    <div class="col-md-4 text-center heartbeat-bar-col-2">
                        <span
	                        class="badge rounded-pill ${status.color}"
	                        :style="status?.color === 'primary'
		                        ? { backgroundColor: '#5cdd8b' }
		                        : status?.color === 'danger'
			                        ? { backgroundColor: '#dc3545' }
			                        : {}"
	                        style="font-size: 30px;"
	                        data-testid="monitor-status"
                        >
	                        {{ status.text }}
                         </span>
                    </div>
                </div>
            </div>

            <div v-if="monitor.type === 'push'" class="shadow-box big-padding">
                <a href="#" @click="pushMonitor.showPushExamples = !pushMonitor.showPushExamples">{{ $t("pushViewCode") }}</a>

                <transition name="slide-fade" appear>
                    <div v-if="pushMonitor.showPushExamples" class="mt-3">
                        <select id="push-current-example" v-model="pushMonitor.currentExample" class="form-select">
                            <optgroup :label="$t('programmingLanguages')">
                                <option value="csharp">C#</option>
                                <option value="go">Go</option>
                                <option value="java">Java</option>
                                <option value="javascript-fetch">JavaScript (fetch)</option>
                                <option value="php">PHP</option>
                                <option value="python">Python</option>
                                <option value="typescript-fetch">TypeScript (fetch)</option>
                            </optgroup>
                            <optgroup :label="$t('pushOthers')">
                                <option value="bash-curl">Bash (curl)</option>
                                <option value="powershell">PowerShell</option>
                                <option value="docker">Docker</option>
                            </optgroup>
                        </select>

                        <prism-editor v-model="pushMonitor.code" class="css-editor mt-3" :highlight="pushExampleHighlighter" line-numbers readonly></prism-editor>
                    </div>
                </transition>
            </div>

            <div class="shadow-box big-padding text-center stats">
                <div class="row">
                    <div v-if="monitor.type !== 'group'" class="col-12 col-sm col row d-flex align-items-center d-sm-block">
                        <h4 class="col-4 col-sm-12">{{ pingTitle() }}</h4>
                        <p class="col-4 col-sm-12 mb-0 mb-sm-2">({{ $t("Current") }})</p>
                        <span class="col-4 col-sm-12 num">
                            <a href="#" @click.prevent="showPingChartBox = !showPingChartBox">
                                <CountUp :value="ping" />
                            </a>
                        </span>
                    </div>
                    <div v-if="monitor.type !== 'group'" class="col-12 col-sm col row d-flex align-items-center d-sm-block">
                        <h4 class="col-4 col-sm-12">{{ pingTitle(true) }}</h4>
                        <p class="col-4 col-sm-12 mb-0 mb-sm-2">(24{{ $t("-hour") }})</p>
                        <span class="col-4 col-sm-12 num">
                            <CountUp :value="avgPing" />
                        </span>
                    </div>

                    <div class="col-12 col-sm col row d-flex align-items-center d-sm-block">
                        <h4 class="col-4 col-sm-12">{{ $t("Uptime") }}</h4>
                        <p class="col-4 col-sm-12 mb-0 mb-sm-2">(24{{ $t("-hour") }})</p>
                        <span class="col-4 col-sm-12 num">
                            <Uptime :monitor="monitor" type="24" />
                        </span>
                    </div>

                    <div class="col-12 col-sm col row d-flex align-items-center d-sm-block">
                        <h4 class="col-4 col-sm-12">{{ $t("Uptime") }}</h4>
                        <p class="col-4 col-sm-12 mb-0 mb-sm-2">(30{{ $t("-day") }})</p>
                        <span class="col-4 col-sm-12 num">
                            <Uptime :monitor="monitor" type="720" />
                        </span>
                    </div>

                    <div class="col-12 col-sm col row d-flex align-items-center d-sm-block">
                        <h4 class="col-4 col-sm-12">{{ $t("Uptime") }}</h4>
                        <p class="col-4 col-sm-12 mb-0 mb-sm-2">(1{{ $t("-year") }})</p>
                        <span class="col-4 col-sm-12 num">
                            <Uptime :monitor="monitor" type="1y" />
                        </span>
                    </div>

                    <div v-if="tlsInfo" class="col-12 col-sm col row d-flex align-items-center d-sm-block">
                        <h4 class="col-4 col-sm-12">{{ $t("Cert Exp.") }}</h4>
                        <p class="col-4 col-sm-12 mb-0 mb-sm-2">(<Datetime :value="tlsInfo.certInfo.validTo" date-only />)</p>
                        <span class="col-4 col-sm-12 num">
                            <a href="#" @click.prevent="toggleCertInfoBox = !toggleCertInfoBox">{{ tlsInfo.certInfo.daysRemaining }} {{ $tc("day", tlsInfo.certInfo.daysRemaining) }}</a>
                        </span>
                    </div>
                </div>
            </div>

            <transition name="slide-fade" appear>
                <div v-if="showCertInfoBox" class="shadow-box big-padding text-center">
                    <div class="row">
                        <div class="col">
                            <certificate-info :certInfo="tlsInfo.certInfo" :valid="tlsInfo.valid" />
                        </div>
                    </div>
                </div>
            </transition>

            <div v-if="showPingChartBox" class="shadow-box big-padding text-center ping-chart-wrapper">
                <div class="row">
                    <div class="col chart-wrapper">
                        <PingChart :monitor-id="monitor.id" />
                    </div>
                </div>
            </div>

            <div v-if="monitor.type === 'real-browser'" class="shadow-box">
                <div class="row">
                    <div class="col-md-6 zoom-cursor">
                        <img :src="screenshotURL" style="width: 100%;" alt="screenshot of the website" @click="showScreenshotDialog">
                    </div>
                    <ScreenshotDialog ref="screenshotDialog" :imageURL="screenshotURL" />
                </div>
            </div>

            <div class="shadow-box table-shadow-box">
                <div class="dropdown dropdown-clear-data">
                    <button class="btn btn-sm btn-outline-danger dropdown-toggle" type="button" data-bs-toggle="dropdown">
                        <font-awesome-icon icon="trash" /> {{ $t("Clear Data") }}
                    </button>
                    <ul class="dropdown-menu dropdown-menu-end">
                        <li>
                            <button type="button" class="dropdown-item" @click="clearEventsDialog">
                                {{ $t("Events") }}
                            </button>
                        </li>
                        <li>
                            <button type="button" class="dropdown-item" @click="clearHeartbeatsDialog">
                                {{ $t("Heartbeats") }}
                            </button>
                        </li>
                    </ul>
                </div>
                <table class="table table-borderless table-hover">
                    <thead>
                        <tr>
                            <th>{{ $t("Status") }}</th>
                            <th>{{ $t("DateTime") }}</th>
                            <th>{{ $t("Message") }}</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr v-for="(beat, index) in displayedRecords" :key="index" style="padding: 10px;">
                            <td><Status :status="beat.status" /></td>
                            <td :class="{ 'border-0':! beat.msg}"><Datetime :value="beat.time" /></td>
                            <td class="border-0">{{ beat.msg }}</td>
                        </tr>

                        <tr v-if="importantHeartBeatListLength === 0">
                            <td colspan="3">
                                {{ $t("No important events") }}
                            </td>
                        </tr>
                    </tbody>
                </table>

                <div class="d-flex justify-content-center kuma_pagination">
                    <pagination
                        v-model="page"
                        :records="importantHeartBeatListLength"
                        :per-page="perPage"
                        :options="paginationConfig"
                    />
                </div>
            </div>

            <Confirm ref="confirmPause" :yes-text="$t('Yes')" :no-text="$t('No')" @yes="pauseMonitor">
                {{ $t("pauseMonitorMsg") }}
            </Confirm>

            <Confirm ref="confirmDelete" btn-style="btn-danger" :yes-text="$t('Yes')" :no-text="$t('No')" @yes="deleteMonitor">
                {{ $t("deleteMonitorMsg") }}
            </Confirm>

            <Confirm ref="confirmClearEvents" btn-style="btn-danger" :yes-text="$t('Yes')" :no-text="$t('No')" @yes="clearEvents">
                {{ $t("clearEventsMsg") }}
            </Confirm>

            <Confirm ref="confirmClearHeartbeats" btn-style="btn-danger" :yes-text="$t('Yes')" :no-text="$t('No')" @yes="clearHeartbeats">
                {{ $t("clearHeartbeatsMsg") }}
            </Confirm>
        </div>
    </transition>
</template>

<script>
import { defineAsyncComponent } from "vue";
import { useToast } from "vue-toastification";
const toast = useToast();
import Confirm from "../components/Confirm.vue";
import HeartbeatBar from "../components/HeartbeatBar.vue";
import Status from "../components/Status.vue";
import Datetime from "../components/Datetime.vue";
import CountUp from "../components/CountUp.vue";
import Uptime from "../components/Uptime.vue";
import Pagination from "v-pagination-3";
const PingChart = defineAsyncComponent(() => import("../components/PingChart.vue"));
import Tag from "../components/Tag.vue";
import CertificateInfo from "../components/CertificateInfo.vue";
import { getMonitorRelativeURL } from "../util.ts";
import { URL } from "whatwg-url";
import DOMPurify from "dompurify";
import { marked } from "marked";
import { getResBaseURL } from "../util-frontend";
import { highlight, languages } from "prismjs/components/prism-core";
import "prismjs/components/prism-clike";
import "prismjs/components/prism-javascript";
import "prismjs/components/prism-css";
import { PrismEditor } from "vue-prism-editor";
import "vue-prism-editor/dist/prismeditor.min.css";
import ScreenshotDialog from "../components/ScreenshotDialog.vue";

export default {
    components: {
        Uptime,
        CountUp,
        Datetime,
        HeartbeatBar,
        Confirm,
        Status,
        Pagination,
        PingChart,
        Tag,
        CertificateInfo,
        PrismEditor,
        ScreenshotDialog
    },
    data() {
        return {
            page: 1,
            perPage: 25,
            heartBeatList: [],
            toggleCertInfoBox: false,
            showPingChartBox: true,
            paginationConfig: {
                hideCount: true,
                chunksNavigation: "scroll",
            },
            cacheTime: Date.now(),
            importantHeartBeatListLength: 0,
            displayedRecords: [],
            pushMonitor: {
                showPushExamples: false,
                currentExample: "javascript-fetch",
                code: "",
            },
        };
    },
    computed: {
        monitor() {
            let id = this.$route.params.id;
            return this.$root.monitorList[id];
        },

        lastHeartBeat() {
            // Also trigger screenshot refresh here
            // eslint-disable-next-line vue/no-side-effects-in-computed-properties
            this.cacheTime = Date.now();

            if (this.monitor.id in this.$root.lastHeartbeatList && this.$root.lastHeartbeatList[this.monitor.id]) {
                return this.$root.lastHeartbeatList[this.monitor.id];
            }

            return {
                status: -1,
            };
        },

        ping() {
            if (this.lastHeartBeat.ping || this.lastHeartBeat.ping === 0) {
                return this.lastHeartBeat.ping;
            }

            return this.$t("notAvailableShort");
        },

        avgPing() {
            if (this.$root.avgPingList[this.monitor.id] || this.$root.avgPingList[this.monitor.id] === 0) {
                return this.$root.avgPingList[this.monitor.id];
            }

            return this.$t("notAvailableShort");
        },

        status() {
            if (this.$root.statusList[this.monitor.id]) {
                return this.$root.statusList[this.monitor.id];
            }

            return { };
        },

        tlsInfo() {
            if (this.$root.tlsInfoList[this.monitor.id] && this.$root.tlsInfoList[this.monitor.id].certInfo) {
                return this.$root.tlsInfoList[this.monitor.id];
            }

            return null;
        },

        showCertInfoBox() {
            return this.tlsInfo != null && this.toggleCertInfoBox;
        },

        group() {
            return this.monitor.path.slice(0, -1).join(" / ");
        },

        pushURL() {
            return this.$root.baseURL + "/api/push/" + this.monitor.pushToken + "?status=up&msg=OK&ping=";
        },

        screenshotURL() {
            return getResBaseURL() + this.monitor.screenshot + "?time=" + this.cacheTime;
        },

        descriptionHTML() {
            if (this.monitor.description != null) {
                return DOMPurify.sanitize(marked(this.monitor.description));
            } else {
                return "";
            }
        }
    },

    watch: {
        page(to) {
            this.getImportantHeartbeatListPaged();
        },

        monitor(to) {
            this.getImportantHeartbeatListLength();
        },
        "monitor.type"() {
            if (this.monitor && this.monitor.type === "push") {
                this.loadPushExample();
            }
        },
        "pushMonitor.currentExample"() {
            this.loadPushExample();
        },
    },

    mounted() {
        this.getImportantHeartbeatListLength();

        this.$root.emitter.on("newImportantHeartbeat", this.onNewImportantHeartbeat);

        if (this.monitor && this.monitor.type === "push") {
            if (this.lastHeartBeat.status === -1) {
                this.pushMonitor.showPushExamples = true;
            }
            this.loadPushExample();
        }
    },

    beforeUnmount() {
        this.$root.emitter.off("newImportantHeartbeat", this.onNewImportantHeartbeat);
    },

    methods: {
        getResBaseURL,
        testNotification() {
            this.$root.getSocket().emit("testNotification", this.monitor.id);
            this.$root.toastSuccess("Test notification is requested.");
        },
        pauseDialog() {
            this.$refs.confirmPause.show();
        },
        resumeMonitor() {
            this.$root.getSocket().emit("resumeMonitor", this.monitor.id, (res) => {
                this.$root.toastRes(res);
            });
        },
        pauseMonitor() {
            this.$root.getSocket().emit("pauseMonitor", this.monitor.id, (res) => {
                this.$root.toastRes(res);
            });
        },
        deleteDialog() {
            this.$refs.confirmDelete.show();
        },
        showScreenshotDialog() {
            this.$refs.screenshotDialog.show();
        },
        clearEventsDialog() {
            this.$refs.confirmClearEvents.show();
        },
        clearHeartbeatsDialog() {
            this.$refs.confirmClearHeartbeats.show();
        },
        deleteMonitor() {
            this.$root.deleteMonitor(this.monitor.id, (res) => {
                this.$root.toastRes(res);
                if (res.ok) {
                    this.$router.push("/dashboard");
                }
            });
        },
        clearEvents() {
            this.$root.clearEvents(this.monitor.id, (res) => {
                if (res.ok) {
                    this.getImportantHeartbeatListLength();
                } else {
                    toast.error(res.msg);
                }
            });
        },
        clearHeartbeats() {
            this.$root.clearHeartbeats(this.monitor.id, (res) => {
                if (! res.ok) {
                    toast.error(res.msg);
                }
            });
        },
        pingTitle(average = false) {
            let translationPrefix = "";
            if (average) {
                translationPrefix = "Avg. ";
            }

            if (this.monitor.type === "http" || this.monitor.type === "keyword" || this.monitor.type === "json-query") {
                return this.$t(translationPrefix + "Response");
            }

            return this.$t(translationPrefix + "Ping");
        },
        monitorURL(id) {
            return getMonitorRelativeURL(id);
        },
        filterPassword(urlString) {
            try {
                let parsedUrl = new URL(urlString);
                if (parsedUrl.password !== "") {
                    parsedUrl.password = "******";
                }
                return parsedUrl.toString();
            } catch (e) {
                // Handle SQL Server
                return urlString.replaceAll(/Password=(.+);/ig, "Password=******;");
            }
        },
        getImportantHeartbeatListLength() {
            if (this.monitor) {
                this.$root.getSocket().emit("monitorImportantHeartbeatListCount", this.monitor.id, (res) => {
                    if (res.ok) {
                        this.importantHeartBeatListLength = res.count;
                        this.getImportantHeartbeatListPaged();
                    }
                });
            }
        },
        getImportantHeartbeatListPaged() {
            if (this.monitor) {
                const offset = (this.page - 1) * this.perPage;
                this.$root.getSocket().emit("monitorImportantHeartbeatListPaged", this.monitor.id, offset, this.perPage, (res) => {
                    if (res.ok) {
                        this.displayedRecords = res.data;
                    }
                });
            }
        },
        onNewImportantHeartbeat(heartbeat) {
            if (heartbeat.monitorID === this.monitor?.id) {
                if (this.page === 1) {
                    this.displayedRecords.unshift(heartbeat);
                    if (this.displayedRecords.length > this.perPage) {
                        this.displayedRecords.pop();
                    }
                    this.importantHeartBeatListLength += 1;
                }
            }
        },
        pushExampleHighlighter(code) {
            return highlight(code, languages.js);
        },
        loadPushExample() {
            this.pushMonitor.code = "";
            this.$root.getSocket().emit("getPushExample", this.pushMonitor.currentExample, (res) => {
                let code = res.code
                    .replace("60", this.monitor.interval)
                    .replace("https://example.com/api/push/key?status=up&msg=OK&ping=", this.pushURL);
                this.pushMonitor.code = code;
            });
        },

        /**
         * Triggers the browser's print dialog to save the page as a PDF.
         * Relies on print-specific CSS to format the output.
         * @returns {void}
         */
        downloadPDF() {
            window.print();
        }
    },
};
</script>

<style lang="scss" scoped>
@import "../assets/vars.scss";

@media (max-width: 767px) {
    .badge {
        margin-top: 14px;
    }
}

@media (max-width: 550px) {
    .functions {
        text-align: center;
    }

    .ping-chart-wrapper {
        padding: 10px !important;
    }

    .dropdown-clear-data {
        margin-bottom: 10px;
    }
}

@media (max-width: 400px) {
    .btn {
        display: inline-flex;
        flex-direction: column;
        align-items: center;
        padding-top: 10px;
        font-size: 0.9em;
    }

    a.btn {
        padding-left: 25px;
        padding-right: 25px;
    }

    .dropdown-clear-data {
        button {
            display: block;
            padding-top: 4px;
        }
    }
}

.url {
    color: $primary;
    margin-bottom: 20px;
    font-weight: bold;

    a {
        color: $primary;
    }
}

.shadow-box {
    padding: 20px;
    margin-top: 25px;
}

.word {
    color: $secondary-text;
    font-size: 14px;
}

table {
    font-size: 14px;

    tr {
        transition: all ease-in-out 0.2ms;
    }
}

.stats p {
    font-size: 13px;
    color: $secondary-text;
}

.stats {
    padding: 10px;

    .col {
        margin: 20px 0;
    }
}

@media (max-width: 550px) {
    .stats {
        .col {
            margin: 10px 0 !important;
        }

        h4 {
            font-size: 1.1rem;
        }
    }
}

.keyword {
    color: black;
}

.dropdown-clear-data {
    float: right;

    ul {
        width: 100%;
        min-width: unset;
        padding-left: 0;
    }
}

.dark {
    .keyword {
        color: $dark-font-color;
    }

    .keyword-inverted {
        color: $dark-font-color;
    }

    .dropdown-clear-data {
        ul {
            background-color: $dark-bg;
            border-color: $dark-bg2;
            border-width: 2px;

            li button {
                color: $dark-font-color;
            }

            li button:hover {
                background-color: $dark-bg2;
            }
        }
    }
}

.tags {
    margin-bottom: 0.5rem;
}

.tags > div:first-child {
    margin-left: 0 !important;
}

.monitor-id {
    display: inline-flex;
    font-size: 0.7em;
    margin-left: 0.3em;
    color: $secondary-text;
    flex-direction: row;
    flex-wrap: nowrap;

    .hash {
        user-select: none;
    }

    .dark & {
        opacity: 0.7;
    }
}

.functions {
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
}
</style>

<style lang="scss">
@media print {
    @import "../assets/vars.scss";

    :root {
        -webkit-print-color-adjust: exact !important;
        print-color-adjust: exact !important;
    }

    // 1. Hide everything on the page by default.
    body * {
        visibility: hidden;
    }

    // 2. Make ONLY the printable area and its children visible.
    .printable-content, .printable-content * {
        visibility: visible;
    }

    // 3. Position the printable area to take up the entire page.
    .printable-content {
        position: absolute;
        left: 0;
        top: 0;
        width: 100%;
        margin: 0;
        padding: 1rem;
    }

    // 4. Hide unwanted interactive elements INSIDE the printable area.
    .printable-content .functions,
    .printable-content .dropdown-clear-data,
    .printable-content .kuma_pagination,
    .printable-content .btn,
    .printable-content .confirm {
        display: none !important;
    }

    // 5. Apply print-friendly styling within the printable area.
    .printable-content .shadow-box {
        box-shadow: none !important;
        border: 1px solid #dee2e6;
        page-break-inside: avoid;
    }

    .printable-content a[href]:after {
        content: "" !important;
    }

    * {
        color-adjust: exact !important;
    }

    // ===============================================================
    // NEW: Styles for Heartbeat Bar and Status Layout
    // ===============================================================
    .printable-content .shadow-box:first-of-type > .row {
        display: flex;
        justify-content: space-between;
        align-items: center;
        flex-wrap: nowrap;
    }

    .printable-content .shadow-box:first-of-type .col-md-4 {
        width: auto;
        flex: 0 0 auto;
    }

    // ===============================================================
    // NEW: Styles for Table Header
    // ===============================================================
    .printable-content .table thead th {
        background-color: #f2f2f2;
        border: 1px solid #dee2e6 !important;
    }

    // 6. Handle dark theme colors correctly within the printable area.
    .dark .printable-content {
        background-color: #161B22 !important;

        h1, h2, h3, h4, p, span, div, td, th, a, .keyword, .monitor-id, .word, .num {
            color: #c9d1d9 !important;
        }

        .shadow-box {
            background-color: #0D1117 !important;
            border: 1px solid #30363d !important;
        }

        // Dark mode override for new table header styles
        .table thead th {
            background-color: #212529;
            border-color: #495057 !important;
        }

        .table {
            --bs-table-color: #c9d1d9;
            --bs-table-bg: #0D1117;
            --bs-table-border-color: #30363d;
            --bs-table-hover-color: #c9d1d9;
            --bs-table-hover-bg: #161B22;
        }
    }

}
</style>
