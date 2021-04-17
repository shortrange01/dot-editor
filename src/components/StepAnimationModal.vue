<template>
    <div class="demo">
        <div class="leftBox"></div>
        <div class="rightBox">
            <div class="btn close" @click="$emit('clickCloseBtn')">×</div>
            <div class="btn" @click="openPngFileSelector()">Set Png File</div>
            <div class="btn" @click="startAnimation()">Start</div>
            <div class="btn" @click="resetPngFiles()">Reset</div>
            <input ref="inputPngFile" style="display: none;" @change="addPngFileToList" type="file" value />
        </div>
    </div>
</template>
<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';

@Component
export default class StepAnimationModal extends Vue {
    inputPngFileElement?: HTMLInputElement;
    pngFileList: string[] = [];
    stepIntervalId: number = 0;

    mounted() {
        this.inputPngFileElement = this.$refs.inputPngFile as HTMLInputElement;
    }

    openPngFileSelector() {
        if (this.inputPngFileElement) this.inputPngFileElement.click();
    }

    addPngFileToList(event: any) {
        if (event.currentTarget.value) {
            const blobUrl: string | null = window.URL.createObjectURL(event.currentTarget.files[0]);
            if (blobUrl) this.pngFileList.push(blobUrl);
            this.$forceUpdate();
        }
    }

    startAnimation() {
        const leftBox = document.querySelector('.leftBox');
        if (leftBox && this.pngFileList.length > 0) {
            const maxCount = this.pngFileList.length - 1;
            let i: number = 0;
            this.stepIntervalId = window.setInterval(() => {
                leftBox.innerHTML = '<img width="300" src="' + this.pngFileList[i] + '">';
                i = i >= maxCount ? 0 : i + 1;
            }, 300);
        }
    }

    resetPngFiles() {
        this.pngFileList = [];
        window.clearInterval(this.stepIntervalId);
    }
}
</script>
