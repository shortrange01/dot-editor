<template>
    <div class="container dotEditor">
        <StepAnimationModal v-show="showStepAnimationModal" @clickCloseBtn="closeStepAnimationModal" />
        <CanvasBlock ref="canvasBlock" :dot-list="dotList" />
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
                <DownloadBtn :dot-list="dotList" :color-history="colorHistory" />
                <div class="btn" @click="onClickLoadBtn">Load JSON</div>
                <div class="btn" @click="onClickPngDownLoadBtn">PNG Download</div>
                <div class="btn" @click="openStepAnimationModal">Step Anime</div>
            </div>
            <input ref="inputFile" style="display: none;" @change="onChangeFile" type="file" value />
        </div>
    </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import CanvasBlock from './CanvasBlock.vue';
import EditorBlock, { DotList } from './EditorBlock.vue';
import PickerBtnBlock from './PickerBtnBlock.vue';
import ColorHistoryBlock from './ColorHistoryBlock.vue';
import StepAnimationModal from './StepAnimationModal.vue';
import DownloadBtn from './DownloadBtn.vue';

const DotLineCount = 16; // １行分のドット数
const MaxHistoryCount = 17; // 色選択履歴の最大保持数

@Component({
    components: {
        CanvasBlock,
        EditorBlock,
        PickerBtnBlock,
        ColorHistoryBlock,
        StepAnimationModal,
        DownloadBtn,
    },
})
export default class DotEditor extends Vue {
    dotList: DotList = [];
    colorHistory: string[] = [];
    isEraserMode = false;
    inputFileElement?: HTMLInputElement;
    showStepAnimationModal: boolean = false;
    pickerComponent = 'ChromePicker';

    $refs!: {
        canvasBlock: CanvasBlock;
        editorBlock: EditorBlock;
        inputFile: HTMLInputElement;
    };

    created() {
        this.initDotList();
    }

    mounted() {
        this.inputFileElement = this.$refs.inputFile;
    }

    initDotList() {
        // 空の256ドット配列を作成
        for (let i = 1; i <= DotLineCount; i++) {
            const arr = [];
            for (let i = 1; i <= DotLineCount; i++) {
                arr.push({ color: '' });
            }
            this.dotList.push(arr);
        }
    }

    changeColor(dot: { color: string }) {
        if (this.isEraserMode) {
            dot.color = '';
            this.$refs.canvasBlock.drowCanvas();
            return;
        }
        dot.color = this.$refs.editorBlock.nowColor.hex;

        this.$refs.canvasBlock.drowCanvas();
        if (this.colorHistory.indexOf(dot.color) < 0) {
            this.colorHistory.push(dot.color);
            if (this.colorHistory.length > MaxHistoryCount) {
                this.colorHistory.shift();
            }
        }
    }

    // 色変え
    selectNewColor(color: string): void {
        this.$refs.editorBlock.nowColor = { hex: color };
    }

    // 消しゴム機能の on/off
    toggleEraserMode() {
        this.isEraserMode = !this.isEraserMode;
    }

    onClickPngDownLoadBtn(): void {
        let link: HTMLAnchorElement = document.createElement('a');
        link.href = this.$refs.canvasBlock.canvas ? this.$refs.canvasBlock.canvas.toDataURL('image/png') : '';
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

                this.$refs.canvasBlock.drowCanvas();
            });
        }
    }

    onClickLoadBtn() {
        if (this.inputFileElement) this.inputFileElement.click();
    }

    openStepAnimationModal() {
        this.showStepAnimationModal = true;
    }

    closeStepAnimationModal() {
        this.showStepAnimationModal = false;
    }

    setPickerComponent(picker: string) {
        this.pickerComponent = picker;
    }

    disableElaser() {
        this.isEraserMode = false;
    }
}
</script>
