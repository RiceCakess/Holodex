<template>
    <!-- Confirmation for deleting layout -->
    <v-dialog v-bind:value="value" v-on:input="$emit('input', $event)" width="400">
        <v-card>
            <v-card-title> {{ $t("views.multiview.confirmOverwrite") }} </v-card-title>
            <v-card-text class="d-flex flex-column justify-center align-center">
                <LayoutPreview :layout="layoutPreview.layout" :content="layoutPreview.content" />
                <v-checkbox
                    v-model="overwriteMerge"
                    :label="`Fill empty cells with current videos`"
                    hide-details
                ></v-checkbox>
            </v-card-text>
            <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="primary" text @click="confirmFn(overwriteMerge)">
                    {{ $t("views.multiview.confirmOverwriteYes") }}
                </v-btn>
                <v-btn color="primary" text @click="cancelFn(overwriteMerge)">
                    {{ $t("views.library.deleteConfirmationCancel") }}
                </v-btn>
            </v-card-actions>
        </v-card>
    </v-dialog>
</template>

<script>
import LayoutPreview from "@/components/multiview/LayoutPreview.vue";

export default {
    name: "LayoutChangePrompt",
    components: {
        LayoutPreview,
    },
    data() {
        return {
            overwriteMerge: this.defaultOverwrite,
        };
    },
    props: {
        value: {
            type: Boolean,
            required: true,
        },
        confirmFn: {
            type: Function,
        },
        cancelFn: {
            type: Function,
        },
        defaultOverwrite: {
            type: Boolean,
        },
        layoutPreview: {
            type: Object,
        },
    },
};
</script>

<style></style>
