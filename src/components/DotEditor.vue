<template>
    <div class="container dotEditor">
        <StepAnimationModal v-show="showStepAnimationModal" @clickCloseBtn="closeStepAnimationModal" />
        <CanvasBlock ref="canvasBlock" :dot-list="dotList" />
        <div>
            <EditorBlock
                ref="editorBlock"
                :picker-component="pickerComponent"
                :dot-list="dotList"
                @setDotColor="setDotColor"
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
                <DownloadJsonFileBtn :dot-list="dotList" :color-history="colorHistory" />
                <div class="btn" @click="openFileSelector">Load JSON</div>
                <div class="btn" @click="$refs.canvasBlock.downloadPngFile()">PNG Download</div>
                <div class="btn" @click="openStepAnimationModal">Step Anime</div>
            </div>
            <input ref="inputFile" style="display: none;" @change="loadJsonFile($event)" type="file" value />
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
import DownloadJsonFileBtn from './DownloadJsonFileBtn.vue';

const DotLineCount = 16; // １行分のドット数
const MaxHistoryCount = 17; // 色選択履歴の最大保持数

interface FileAPIEventTarget extends EventTarget {
    result: string;
}

@Component({
    components: {
        CanvasBlock,
        EditorBlock,
        PickerBtnBlock,
        ColorHistoryBlock,
        StepAnimationModal,
        DownloadJsonFileBtn,
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

    setDotColor(dot: { color: string }) {
        dot.color = this.isEraserMode ? '' : this.$refs.editorBlock.nowColor.hex;
        this.$refs.canvasBlock.drowCanvas();
        if (!this.isEraserMode) this.addHistoryColor(dot);
    }

    addHistoryColor(dot: { color: string }) {
        if (this.colorHistory.indexOf(dot.color) < 0) {
            this.colorHistory.push(dot.color);
            if (this.colorHistory.length > MaxHistoryCount) {
                this.colorHistory.shift();
            }
        }
    }

    selectNewColor(color: string): void {
        this.$refs.editorBlock.nowColor = { hex: color };
    }

    toggleEraserMode() {
        this.isEraserMode = !this.isEraserMode;
    }

    loadJsonFile(event: Event): void {
        const target = event.target as HTMLInputElement;
        if (target.files) {
            const reader: FileReader = new FileReader();
            reader.readAsText(target.files[0]);
            reader.addEventListener('load', (event: ProgressEvent) => {
                // value を消さないと同じファイルを連続で読み込む際に change event が発火しない
                if (this.inputFileElement) this.inputFileElement.value = '';
                const target = event.target as FileAPIEventTarget;
                const result = JSON.parse(target.result);
                this.dotList = result.dotList;
                this.colorHistory = result.colorHistory;
                this.$nextTick().then(() => {
                    this.$refs.canvasBlock.drowCanvas();
                });
            });
        }
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

    openFileSelector() {
        if (this.inputFileElement) this.inputFileElement.click();
    }
}
</script>
