<template>
    <div class="btn" @click="downloadJonFile()">Download</div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from 'vue-property-decorator';
import { DotList } from './EditorBlock.vue';

@Component
export default class DownloadJsonFileBtn extends Vue {
    @Prop({ type: Array }) dotList?: DotList;
    @Prop({ type: Array }) colorHistory?: string[];

    downloadJonFile(): void {
        const link: HTMLAnchorElement = document.createElement('a');
        const data: { dotList?: DotList; colorHistory?: string[] } = {
            dotList: this.dotList,
            colorHistory: this.colorHistory,
        };
        link.href = 'data:text/plain,' + encodeURIComponent(JSON.stringify(data));
        // ファイル名は[UNIXTIME].json
        link.download = `${Math.round(new Date().getTime() / 1000)}.json`;
        link.click();
    }
}
</script>
