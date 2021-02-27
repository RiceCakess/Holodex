<template>
    <v-container fluid v-if="!isLoading && !hasError" class="video-editor">
        <v-row>
            <v-col :md="3" :lg="4" cols="12" class="px-0 pt-0 px-md-3">
                <WatchFrame :video="video">
                    <template v-slot:youtube>
                        <youtube
                            v-if="video.id"
                            :video-id="video.id"
                            @ready="ready"
                            :playerVars="{
                                ...(timeOffset && { start: timeOffset }),
                                autoplay: $store.state.settings.autoplayVideo ? 1 : 0,
                                playsinline: 1,
                            }"
                        >
                        </youtube>
                    </template>
                </WatchFrame>
                <WatchInfo :video="video" key="info" />
                <WatchComments
                    :comments="comments"
                    :video="video"
                    :limit="$store.state.isMobile ? 5 : 0"
                    @timeJump="seekTo"
                    key="comments"
                    v-if="comments && comments.length"
                />
            </v-col>
            <v-col class="related-videos pt-0" :md="9" :lg="8">
                <v-row fluid>
                    <v-tabs v-model="currentTab">
                        <v-tab>Topic</v-tab>
                        <v-tab>Songs</v-tab>
                        <v-tab disabled>Channel Mentions</v-tab>
                        <v-tab disabled>Sources/Clips</v-tab>
                    </v-tabs>
                    <v-col cols="12" class="pa-4">
                        <div v-show="currentTab === TABS.TOPIC">
                            <v-card-title>
                                <v-icon left>{{ icons.mdiAnimationPlay }}</v-icon>
                                <h5>Change stream topic</h5>
                            </v-card-title>
                            <v-card-text>
                                <p>
                                    All users can assign topic to videos, but only editors and admins can remove and
                                    change the topic once set.
                                </p>
                                <v-select :items="topics" label="Topic (leave empty to unset)" v-model="newTopic" />
                            </v-card-text>
                            <v-card-actions>
                                <v-btn color="blue darken-1" text @click="saveTopic"> Save </v-btn>
                            </v-card-actions>
                        </div>
                        <div v-show="currentTab === TABS.SONGS">
                            <VideoEditSongs
                                :video="video"
                                :currentTime="currentTime"
                                @timeJump="seekTo"
                            ></VideoEditSongs>
                        </div>
                    </v-col>
                </v-row>
            </v-col>
        </v-row>
    </v-container>
    <LoadingOverlay :isLoading="isLoading" :showError="hasError" v-else />
</template>

<script>
import VueYouTubeEmbed from "vue-youtube-embed";
import Vue from "vue";
import LoadingOverlay from "@/components/common/LoadingOverlay";
import WatchInfo from "@/components/watch/WatchInfo";
import WatchFrame from "@/components/watch/WatchFrame";
import WatchComments from "@/components/watch/WatchComments";
import WatchRelatedVideos from "@/components/watch/WatchRelatedVideos";
import WatchLiveChat from "@/components/watch/WatchLiveChat";
import VideoEditSongs from "@/components/media/VideoEditSongs";
import { decodeHTMLEntities } from "@/utils/functions";
// import { dayjs } from "@/utils/time";
import * as icons from "@/utils/icons";
import api from "@/utils/backend-api";

export default {
    name: "Watch",
    metaInfo() {
        return {
            title: this.title,
        };
    },
    components: {
        LoadingOverlay,
        WatchInfo,
        WatchFrame,
        WatchLiveChat,
        WatchRelatedVideos,
        VideoEditSongs,
        WatchComments,
        WatchMugen: () => import("@/components/watch/WatchMugen"),
    },
    data() {
        return {
            isLoading: true,
            hasError: false,
            id: 0,
            video: null,
            comments: null,
            startTime: 0,
            icons,
            currentTab: 0,
            TABS: Object.freeze({
                TOPIC: 0,
                SONGS: 1,
                MENTIONS: 2,
                SOURCES_CLIPS: 3,
            }),

            newTopic: null,
            topics: [],

            timer: null,
            currentTime: 0,
        };
    },
    created() {
        Vue.use(VueYouTubeEmbed);
    },
    mounted() {
        this.init();
    },
    beforeDestroy() {
        clearInterval(this.timer);
    },
    methods: {
        init() {
            this.id = this.videoId;
            this.fetchVideo();
            this.initTab();
        },
        initTab() {
            if (this.currentTab === this.TABS.TOPIC) {
                this.populateTopics();
            }
        },
        ready(event) {
            this.player = event.target;
            this.setTimer();
        },
        seekTo(time) {
            if (!this.player) return;
            this.player.seekTo(time);
        },
        fetchVideo() {
            if (!this.id) throw new Error("Invalid id");
            this.isLoading = true;
            return api
                .video(this.id)
                .then(({ data }) => {
                    this.video = data;
                    api.comments(this.id).then((res) => {
                        this.comments = res.data;
                    });
                    this.isLoading = false;
                })
                .catch((e) => {
                    this.hasError = true;
                    console.error(e);
                });
        },
        async populateTopics() {
            this.topics = (await api.topics()).data.map((topic) => ({
                value: topic.id,
                text: `${topic.id} (${topic.count ?? 0})`,
            }));
        },
        saveTopic() {
            this.dialog = false;
            api.topicSet(this.newTopic, this.videoId, this.$store.state.userdata.jwt);
            this.topic = this.newTopic;
        },
        setTimer() {
            if (this.timer) clearInterval(this.timer);
            if (this.player) {
                this.timer = setInterval(() => {
                    this.currentTime = this.player.getCurrentTime();
                }, 1000);
            }
        },
    },
    computed: {
        related() {
            return {
                simulcasts: this.video.simulcasts || [],
                clips:
                    (this.video.clips &&
                        this.video.clips.filter((x) => this.$store.state.settings.clipLangs.includes(x.lang))) ||
                    [],
                sources: this.video.sources || [],
                refers: this.video.refers || [],
            };
        },
        videoId() {
            return this.$route.params.id || this.$route.query.v;
        },
        timeOffset() {
            return +this.$route.query.t || this.startTime;
        },
        title() {
            return (this.video && this.video.title && decodeHTMLEntities(this.video.title)) || "";
        },
        role() {
            return this.$store.state.userdata?.user?.role;
        },
    },
    watch: {
        // eslint-disable-next-line func-names
        "$route.params.id": function () {
            this.init();
        },
        // eslint-disable-next-line func-names
        "$route.query.v": function () {
            this.init();
        },
        currentTab() {
            this.initTab();
        },
    },
};
</script>

<style>
.video-editor .embedded-chat {
    position: relative;
    min-height: 600px;
}

.video-editor .embedded-chat > iframe {
    position: absolute;
    width: 100%;
    min-height: 600px;
}

.video-editor .watch-card {
    border: none !important;
    box-shadow: none !important;
}

.video-editor .thumbnail-overlay {
    background-color: rgba(0, 0, 0, 0.5);
    width: 100%;
    height: 100%;
    position: absolute;
    top: 0;
}

.video-editor .thumbnail {
    position: relative;
}
.video-editor .watch-btn-group > .v-btn {
    margin-right: 5px;
}
</style>