<template>
    <div id="app">
        <DotEdiotor />
    </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import DotEdiotor from "./components/DotEditor.vue";
import "destyle.css"; // reset css読み込み

@Component({
    components: {
        DotEdiotor,
    },
})
export default class App extends Vue {}
</script>
