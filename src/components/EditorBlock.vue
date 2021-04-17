<template>
    <div class="editorBlock">
        <div class="editorGrid">
            <div v-for="(dotLine, dotLineIndex) in dotList" :key="dotLineIndex" class="row">
                <div
                    v-for="(dot, dotIndex) in dotLine"
                    :key="dotIndex"
                    @mousedown="$emit('changeColor', dot)"
                    @mouseover="checkMouseBtnAndChangeColor($event, dot)"
                    class="col"
                    :style="{ backgroundColor: dot.color }"
                ></div>
            </div>
        </div>
        <div class="pickerWrap">
            <component :is="pickerComponent" v-model="nowColor" />
        </div>
    </div>
</template>

<script lang="ts">
import { Watch, Component, Prop, Vue } from 'vue-property-decorator';
import { Chrome, Photoshop, Material, Compact, Swatches, Slider, Sketch } from 'vue-color';

export type DotList = [{ color: string }[]?];

@Component({
    components: {
        ChromePicker: Chrome,
        PhotoshopPicker: Photoshop,
        MaterialPicker: Material,
        CompactPicker: Compact,
        SwatchesPicker: Swatches,
        SliderPicker: Slider,
        SketchPicker: Sketch,
    },
})
export default class EditorBlock extends Vue {
    nowColor = { hex: '#000' };

    @Prop({ type: Array }) dotList?: DotList;
    @Prop({ type: String }) pickerComponent?: string;

    checkMouseBtnAndChangeColor(event: MouseEventInit, dot: { color: string }): void {
        if (event.buttons === 0) {
            return;
        }
        this.$emit('changeColor', dot);
    }

    @Watch('nowColor.hex')
    onSelectNewColor() {
        // どの状態から色変えしても[消しゴム→描画モード]に切り替える
        this.$emit('selectNewColor');
    }
}
</script>
