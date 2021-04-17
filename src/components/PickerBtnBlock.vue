<template>
    <div class="pickerBtnBlock">
        <div
            v-for="picker in pickerList"
            :key="picker"
            class="btn"
            :class="{ active: picker === pickerComponent }"
            @click="$emit('selectPicker', picker)"
        >
            {{ createPickerName(picker) }}
        </div>
    </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from 'vue-property-decorator';

@Component
export default class PickerBtnBlock extends Vue {
    @Prop({ type: String }) pickerComponent?: string;
    pickerList = [
        'ChromePicker',
        'PhotoshopPicker',
        'MaterialPicker',
        'CompactPicker',
        'SwatchesPicker',
        'SliderPicker',
        'SketchPicker',
    ];

    createPickerName(name: string): string {
        return name.split('Picker')[0];
    }
}
</script>
