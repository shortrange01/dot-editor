<template>
    <div class="container dotEditor">
        <canvas id="canvas" class="canvas" width="512" height="512"></canvas>
        <div>
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
                <div class="pickerWrap">
                    <component :is="pickerComponent" v-model="nowColor" />
                </div>
            </div>
            <div class="pickerBtnBrock">
                <div class="btn" @click="pickerComponent = 'ChromePicker'">Chrome</div>
                <div class="btn" @click="pickerComponent = 'PhotoshopPicker'">Photoshop</div>
                <div class="btn" @click="pickerComponent = 'MaterialPicker'">Material</div>
                <div class="btn" @click="pickerComponent = 'CompactPicker'">Compact</div>
                <div class="btn" @click="pickerComponent = 'SwatchesPicker'">Swatches</div>
                <div class="btn" @click="pickerComponent = 'SliderPicker'">Slider</div>
                <div class="btn" @click="pickerComponent = 'SketchPicker'">Sketch</div>
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
            <div class="fileBtnBrock">
                <div class="btn" @click="onClickDownLoad">download</div>
                <div class="btn" @click="onClickLoadBtn">load json</div>
                <div class="btn" @click="onClickPingDownLoad">png download</div>
            </div>
            <input
                ref="inputFile"
                style="display: none;"
                @change="onChangeFile"
                type="file"
                value=""
            />
        </div>
    </div>
</template>

<script lang="ts">
import { Component, Watch, Vue } from "vue-property-decorator";
import { Chrome, Photoshop, Material, Compact, Swatches, Slider, Sketch } from "vue-color";

type dotList = [{ color: string }[]?];

@Component({
    components: {
        ChromePicker: Chrome,
        PhotoshopPicker: Photoshop,
        MaterialPicker: Material,
        CompactPicker: Compact,
        SwatchesPicker: Swatches,
        SliderPicker: Slider,
        SketchPicker: Sketch
    },
})
export default class DotEditor extends Vue {
    dotList: dotList = [];
    colorHistory: string[] = [];
    nowColor = { hex: "#000" };
    isEraserMode = false;
    inputFileElement?: HTMLInputElement;
    pickerComponent = "ChromePicker";
    canvas?:HTMLCanvasElement;

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
        this.canvas = document.querySelector('#canvas') as HTMLCanvasElement;
    }
    changeColor(dot: { color: string }) {
        if (this.isEraserMode) {
            dot.color = "";
            this.drowCanvas();
            return;
        }
        dot.color = this.nowColor.hex;
        this.drowCanvas();
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
    onClickPingDownLoad(): void {
        let link:HTMLAnchorElement = document.createElement("a");
        link.href = this.canvas ? this.canvas.toDataURL("image/png") : "";
        link.download = `${Math.round(new Date().getTime() / 1000)}.png`;
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
                this.drowCanvas();
            });
        }
    }
    onClickLoadBtn() {
        if (this.inputFileElement) this.inputFileElement.click();
    }
    drowCanvas() {
        // const canvas:HTMLCanvasElement | null = document.querySelector('#canvas');
        let ctx:CanvasRenderingContext2D | null = this.canvas ? this.canvas.getContext('2d') : null;
        if(ctx !== null && this.canvas) ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
        this.dotList.forEach((list, listIndex) => {
            if(list){
                list.forEach((val, valIndex) => {
                    if(val.color && ctx !== null) {
                        ctx.fillStyle = val.color;
                        ctx.fillRect(valIndex*32, listIndex*32, 32, 32);
                    }
                });
            }
        });
    }

    @Watch("nowColor.hex")
    onChangeNewColor() {
        this.isEraserMode = false;
    }
}
</script>
