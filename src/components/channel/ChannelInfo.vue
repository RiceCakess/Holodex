<template>
    <v-list-item-content>
        <v-list-item-title>
            <router-link :to="`/channel/${channel.id}`" class="no-decoration text-truncate">
                {{ channelName }}
                <div class="text-body-2 text--secondary" v-if="!noGroup && channel.group">
                    {{ channel.group }}
                </div>
            </router-link>
        </v-list-item-title>
        <v-list-item-subtitle>
            <template v-if="!noSubscriberCount">
                {{ subscriberCount }}
                <span class="green--text" v-if="subscriberGains">
                    {{ subscriberGains }}
                </span>
            </template>
            <template v-if="includeVideoCount">
                <br />
                {{ $t("component.channelInfo.videoCount", [channel.video_count]) }}
                <router-link :to="`/channel/${channel.id}/clips`" class="no-decoration" v-if="channel.clip_count > 0">
                    •
                    <span class="primary--text">{{ $tc("component.channelInfo.clipCount", channel.clip_count) }}</span>
                </router-link>
            </template>
        </v-list-item-subtitle>
        <v-list-item-subtitle v-if="includeSocials">
            <ChannelSocials :channel="channel" />
        </v-list-item-subtitle>
        <slot></slot>
    </v-list-item-content>
</template>

<script>
import { formatCount } from "@/utils/functions";

export default {
    components: {
        ChannelSocials: () => import("./ChannelSocials"),
    },
    props: {
        channel: {
            type: Object,
            required: true,
        },
        includeSocials: {
            type: Boolean,
            default: false,
        },
        includeVideoCount: {
            type: Boolean,
            default: false,
        },
        noSubscriberCount: {
            type: Boolean,
            default: false,
        },
        noGroup: {
            type: Boolean,
            default: false,
        },
        // noLink: {
        //     type: Boolean,
        //     default: false,
        // },
    },
    methods: {
        formatCount,
    },
    computed: {
        subscriberCount() {
            if (this.channel.subscriber_count) {
                return this.$tc("component.channelInfo.subscriberCount", formatCount(this.channel.subscriber_count));
            }
            return this.$t("component.channelInfo.subscriberNA");
        },
        subscriberGains() {
            return this.channel.subscriber_gains ? `(+${formatCount(this.channel.subscriber_gains)})` : null;
        },
        channelName() {
            const prop = this.$store.state.nameProperty;
            if (this.channel[prop]) return this.channel[prop];
            return this.channel.name;
        },
    },
};
</script>

<style>
.no-decoration {
    text-decoration: initial;
    color: initial;
    font-weight: 400 !important;
    font-size: inherit;
}
</style>