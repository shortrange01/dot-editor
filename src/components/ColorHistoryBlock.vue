<template>
    <div class="colorHistoryBlock">
        <div
            v-for="(color, colorIndex) in colorHistory"
            :key="colorIndex"
            @click="$emit('selectNewColor', color)"
            :style="{ backgroundColor: color }"
            class="colorHistory"
            :class="{ active: !isEraserMode }"
        ></div>
        <div class="eraser" :class="{ active: isEraserMode }" @click="$emit('clickEraser')">
            <div class="left"></div>
            <div class="right">
                <div class="top">
                    <p>PLASTIC ERASER</p>
                </div>
                <div class="middle">
                    <p>MONO</p>
                </div>
                <div class="bottom">
                    <p>TOMBO</p>
                </div>
            </div>
        </div>
    </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from 'vue-property-decorator';

@Component
export default class ColorHistoryBlock extends Vue {
    @Prop({ type: Array }) colorHistory?: string[];
    @Prop({ type: Boolean }) isEraserMode?: boolean;
}
</script>
