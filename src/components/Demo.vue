<template>
    <div class="demo">
        <div class="leftBox"></div>
        <div class="rightBox">
            <div class="btn close" @click="$emit('onClickDemoClose')">
                ×
            </div>
            <div class="btn" @click="onClickSetPngFile()">Set Png File</div>
            <div class="btn" @click="onClickStart()">Start</div>
            <div class="btn" @click="onClickReset()">Reset</div>
            <input
                ref="inputPngFile"
                style="display: none;"
                @change="onChangePngFile"
                type="file"
                value=""
            />
        </div>
    </div>
</template>
<script lang="ts">
import { Component, Vue } from "vue-property-decorator";

@Component
export default class Demo extends Vue {
    inputPngFileElement?: HTMLInputElement;
    pngFileList: string[] = [];
    pngIntervalId: number = 0;

    mounted() {
        this.inputPngFileElement = this.$refs.inputPngFile as HTMLInputElement;
    }
    onClickSetPngFile() {
        if (this.inputPngFileElement) this.inputPngFileElement.click();
    }
    onChangePngFile(event: any) {
        if (event.currentTarget.value) {
            const blobUrl: string | null = window.URL.createObjectURL(
                event.currentTarget.files[0]
            );
            if (blobUrl) this.pngFileList.push(blobUrl);
            this.$forceUpdate();
        }
    }
    onClickStart() {
        const leftBox = document.querySelector(".leftBox");
        if (leftBox && this.pngFileList.length > 0) {
            const maxCount = this.pngFileList.length;
            let i: number = 0;
            this.pngIntervalId = window.setInterval(() => {
                if (i >= maxCount) i = 0;
                leftBox.innerHTML =
                    '<img width="300" src="' + this.pngFileList[i] + '">';
                i++;
            }, 300);
        }
    }
    onClickReset(){
        this.pngFileList = [];
        window.clearInterval(this.pngIntervalId);
    }
}
</script>
