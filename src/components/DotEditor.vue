<template>
    <div class="container dotEditor">
        <demo v-show="showDemoModal" @onclick-close-btn="closeDemoModal"></demo>
        <canvas id="canvas" class="canvas" width="512" height="512"></canvas>
        <div>
            <EditorBlock
                ref="editorBlock"
                :picker-component="pickerComponent"
                :dot-list="dotList"
                @changeColor="changeColor"
                @selectNewColor="disableElaser"
            />
            <PickerBtnBlock :picker-component="pickerComponent" @selectPicker="setPickerComponent" />
            <ColorHistoryBlock
                :color-history="colorHistory"
                :is-eraser-mode="isEraserMode"
                @clickEraser="toggleEraserMode"
                @selectNewColor="selectNewColor"
            />

            <div class="fileBtnBlock">
                <div class="btn" @click="onClickDownLoadBtn">Download</div>
                <div class="btn" @click="onClickLoadBtn">Load JSON</div>
                <div class="btn" @click="onClickPngDownLoadBtn">PNG Download</div>
                <div class="btn" @click="onClickDemoBtn">Demo</div>
            </div>
            <input ref="inputFile" style="display: none;" @change="onChangeFile" type="file" value />
        </div>
    </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import EditorBlock, { DotList } from './EditorBlock.vue';
import PickerBtnBlock from './PickerBtnBlock.vue';
import ColorHistoryBlock from './ColorHistoryBlock.vue';
import Demo from './Demo.vue';

@Component({
    components: {
        EditorBlock,
        PickerBtnBlock,
        ColorHistoryBlock,
        Demo,
    },
})
export default class DotEditor extends Vue {
    dotList: DotList = [];
    colorHistory: string[] = [];
    isEraserMode = false;
    inputFileElement?: HTMLInputElement;
    canvas?: HTMLCanvasElement;
    showDemoModal: boolean = false;
    pickerComponent = 'ChromePicker';

    $refs!: {
        editorBlock: EditorBlock;
        inputFile: HTMLInputElement;
    };

    created() {
        // 空の256ドット配列を作成
        for (let i = 1; i <= 16; i++) {
            const arr = [];
            for (let i = 1; i <= 16; i++) {
                arr.push({ color: '' });
            }
            this.dotList.push(arr);
        }
    }
    mounted() {
        this.inputFileElement = this.$refs.inputFile;
        this.canvas = document.querySelector('#canvas') as HTMLCanvasElement;
    }
    changeColor(dot: { color: string }) {
        if (this.isEraserMode) {
            dot.color = '';
            this.drowCanvas();
            return;
        }
        dot.color = this.$refs.editorBlock.nowColor.hex;
        this.drowCanvas();
        if (this.colorHistory.indexOf(dot.color) < 0) {
            this.colorHistory.push(dot.color);
            if (this.colorHistory.length > 17) {
                this.colorHistory.shift();
            }
        }
    }
    // 色変え
    selectNewColor(color: string): void {
        this.$refs.editorBlock.nowColor = { hex: color };
    }
    // 消しゴム
    toggleEraserMode() {
        this.isEraserMode = !this.isEraserMode;
    }
    onClickDownLoadBtn(): void {
        const link: HTMLAnchorElement = document.createElement('a');
        const data: { dotList: DotList; colorHistory: string[] } = {
            dotList: this.dotList,
            colorHistory: this.colorHistory,
        };
        link.href = 'data:text/plain,' + encodeURIComponent(JSON.stringify(data));
        // ファイル名は取り合えずUNIXTIME
        link.download = `${Math.round(new Date().getTime() / 1000)}.json`;
        link.click();
    }
    onClickPngDownLoadBtn(): void {
        let link: HTMLAnchorElement = document.createElement('a');
        link.href = this.canvas ? this.canvas.toDataURL('image/png') : '';
        // ファイル名は取り合えずUNIXTIME
        link.download = `${Math.round(new Date().getTime() / 1000)}.png`;
        link.click();
    }
    onChangeFile(event: any): void {
        if (event.currentTarget.value) {
            const reader: FileReader = new FileReader();
            reader.readAsText(event.currentTarget.files[0]);
            reader.addEventListener('load', (event: any) => {
                // value を消さないと同じファイルを連続で読み込む際に change event が発火しない
                if (this.inputFileElement) this.inputFileElement.value = '';
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
    onClickDemoBtn() {
        this.showDemoModal = true;
    }
    closeDemoModal() {
        this.showDemoModal = false;
    }

    // canvas描画
    drowCanvas() {
        // const canvas:HTMLCanvasElement | null = document.querySelector('#canvas');
        let ctx: CanvasRenderingContext2D | null = this.canvas ? this.canvas.getContext('2d') : null;
        if (ctx !== null && this.canvas) ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
        this.dotList.forEach((list, listIndex) => {
            if (list) {
                list.forEach((val, valIndex) => {
                    if (val.color && ctx !== null) {
                        ctx.fillStyle = val.color;
                        ctx.fillRect(valIndex * 32, listIndex * 32, 32, 32);
                    }
                });
            }
        });
    }

    setPickerComponent(picker: string) {
        this.pickerComponent = picker;
    }

    disableElaser() {
        this.isEraserMode = false;
    }
}
</script>
