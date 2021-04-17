<template>
    <canvas id="canvas" class="canvas" width="512" height="512"></canvas>
</template>

<script lang="ts">
import { Component, Prop, Vue } from 'vue-property-decorator';
import { DotList } from './EditorBlock.vue';

@Component
export default class CanvasBlock extends Vue {
    @Prop({ type: Array }) dotList?: DotList;

    canvas?: HTMLCanvasElement;

    mounted() {
        this.canvas = document.querySelector('#canvas') as HTMLCanvasElement;
    }

    // canvas描画
    drowCanvas() {
        // const canvas:HTMLCanvasElement | null = document.querySelector('#canvas');
        let ctx: CanvasRenderingContext2D | null = this.canvas ? this.canvas.getContext('2d') : null;
        if (ctx !== null && this.canvas) ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
        if (!this.dotList) return;
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
}
</script>
