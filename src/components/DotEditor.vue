<template>
    <div class="container dotEditor">
        <div class="editorBlock">
            <div class="editorGrid">
                <div
                    v-for="(dotLine, dotLineIndex) in dotList"
                    :key="dotLineIndex"
                    class="row"
                >
                    <div
                        v-for="(dot, dotIndex) in dotLine"
                        :key="dotIndex"
                        @mousedown="onClickDot(dot)"
                        @mouseover="onOverDot(dot)"
                        class="col"
                        :style="{ backgroundColor: dot.color }"
                    ></div>
                </div>
            </div>
            <chrome-picker v-model="nowColor" />
        </div>
        <div class="colorHistoryBrock">
            <div
                v-for="(color, colorIndex) in colorHistory"
                :key="colorIndex"
                @click="onClickColorHistory(color)"
                :style="{ backgroundColor: color }"
                class="colorHistory"
                :class="{ active: !isEraserMode }"
            ></div>
            <div
                class="eraser"
                :class="{ active: isEraserMode }"
                @click="onClickEraser()"
            >
                <div class="left"></div>
                <div class="right">
                    <div class="top"><p>PLASTIC ERASER</p></div>
                    <div class="middle"><p>MONO</p></div>
                    <div class="bottom"><p>TOMBO</p></div>
                </div>
            </div>
        </div>
        <div class="btn download" @click="onClickDownLoad">download</div>
        <div class="btn load" @click="onClickLoadBtn">load json</div>
        <input
            ref="inputFile"
            style="display: none;"
            @change="onChangeFile"
            type="file"
            value=""
        />
    </div>
</template>

<script lang="ts">
import { Component, Watch, Vue } from "vue-property-decorator";
import { Chrome } from "vue-color";

type dotList = [{ color: string }[]?];

@Component({
    components: {
        chromePicker: Chrome,
    },
})
export default class DotEditor extends Vue {
    dotList: dotList = [];
    colorHistory: string[] = [];
    nowColor = { hex: "#000" };
    isEraserMode = false;
    inputFileElement?: HTMLInputElement;

    created() {
        for (let i = 1; i <= 16; i++) {
            const arr = [];
            for (let i = 1; i <= 16; i++) {
                arr.push({ color: "" });
            }
            this.dotList.push(arr);
        }
    }
    mounted() {
        this.inputFileElement = this.$refs.inputFile as HTMLInputElement;
    }
    changeColor(dot: { color: string }) {
        if (this.isEraserMode) {
            dot.color = "";
            return;
        }
        dot.color = this.nowColor.hex;
        if (this.colorHistory.indexOf(dot.color) < 0) {
            this.colorHistory.push(dot.color);
            if (this.colorHistory.length > 17) {
                this.colorHistory.shift();
            }
        }
    }
    onClickDot(dot: { color: string }): void {
        this.changeColor(dot);
    }
    onOverDot(dot: { color: string }): void {
        this.changeColor(dot);
    }
    onClickColorHistory(color: string): void {
        this.isEraserMode = false;
        this.nowColor = { hex: color };
    }
    onClickEraser() {
        this.isEraserMode = !this.isEraserMode;
    }
    onClickDownLoad(): void {
        const link: HTMLAnchorElement = document.createElement("a");
        const data: { dotList: dotList; colorHistory: string[] } = {
            dotList: this.dotList,
            colorHistory: this.colorHistory,
        };
        link.href =
            "data:text/plain," + encodeURIComponent(JSON.stringify(data));
        link.download = `${Math.round(new Date().getTime() / 1000)}.json`;
        link.click();
    }
    onChangeFile(event: any): void {
        if (event.currentTarget.value) {
            const reader: FileReader = new FileReader();
            reader.readAsText(event.currentTarget.files[0]);
            reader.addEventListener("load", (event: any) => {
                // value を消さないと同じファイルを連続で読み込む際に change event が発火しない
                if (this.inputFileElement) this.inputFileElement.value = "";
                const result = JSON.parse(event.currentTarget.result);
                this.dotList = result.dotList;
                this.colorHistory = result.colorHistory;
            });
        }
    }
    onClickLoadBtn() {
        if (this.inputFileElement) this.inputFileElement.click();
    }

    @Watch("nowColor.hex")
    onChangeNewColor() {
        this.isEraserMode = false;
    }
}
</script>
