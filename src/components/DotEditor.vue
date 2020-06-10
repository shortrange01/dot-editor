<template>
    <div>
        <div class="editorBlock">
            <div class="editorGrid">
                <div v-for="(dotLine, dotLineIndex) in dotList" :key="dotLineIndex" class="row">
                    <div v-for="(dot, dotIndex) in dotLine" :key="dotIndex" @mousedown="onClickDot(dot)" @mouseover="onOverDot(dot)" class="col" :style="{backgroundColor: dot.color}"></div>
                </div>
            </div>
            <chrome-picker v-model="nowColor" />
        </div>
        <div class="colorHistoryBrock">
            <div v-for="(color, colorIndex) in colorHistory" :key="colorIndex" @click="onClickColorHistory(color)" :style="{backgroundColor: color}" class="colorHistory"></div>
        </div>
        <div class="btn download" @click="onClickDownLoad">download</div>
        <input @change="onChangeFile" type="file" />
    </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from "vue-property-decorator";
import {Chrome} from 'vue-color';

type dotList = [{color: string}[]?];

@Component({
    components: {
     chromePicker:Chrome
    }
})
export default class DotEditor extends Vue {
//   @Prop() private msg!: string;
    dotList:dotList = [];
    colorHistory:string[] = [];
    nowColor = {hex:'#000'};
    
    created() {
        for(let i = 1; i <= 16; i++ ){
            const arr = [];
            for(let i = 1; i <= 16; i++ ){
                arr.push({color: ''});
            }
            this.dotList.push(arr);
        }
    }
    changeColor(dot: {color: string}){
        dot.color = this.nowColor.hex;
        if(this.colorHistory.indexOf(dot.color) < 0) {
            this.colorHistory.push(dot.color);
            if(this.colorHistory.length > 20) {
                this.colorHistory.shift();
            }
        }
    }
    onClickDot(dot: {color: string}):void {
        this.changeColor(dot);
    }
    onOverDot(dot: {color: string}):void {
        this.changeColor(dot);
    }
    onClickColorHistory(color: string):void {
        this.nowColor = {hex: color};
    }
    onClickDownLoad():void {
        const link:HTMLAnchorElement = document.createElement("a");
        const data:{dotList: dotList, colorHistory: string[]} = {dotList: this.dotList, colorHistory: this.colorHistory};
        link.href = "data:text/plain," + encodeURIComponent(JSON.stringify(data));
        link.download = `${Math.round((new Date()).getTime() / 1000)}.json`;
        link.click();
    }
    onChangeFile(event):void {
        console.info(event);
        const reader = new FileReader();
        reader.readAsText(event.currentTarget.files[0]);
        reader.addEventListener("load", (event:any) => {
            const result = JSON.parse(event.currentTarget.result);
            this.dotList = result.dotList;
            this.colorHistory = result.colorHistory;
        });
    }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
div {
    box-sizing: border-box;
}
.editorBlock {
    display: flex;
}
.editorGrid {
    box-sizing:content-box;
    width: 256px;
    height: 256px;
    border: solid 2px #999;
    margin-right: 10px;
}
.row {
    width: 256px;
    height: 16px;
    display: flex;
    border-bottom: solid 1px #ccc;
}
.col {
    width: 16px;
    border-left: solid 1px #ccc;
}
.colorHistoryBrock {
    display: flex;
    flex-wrap: wrap;
    margin-top: 10px;
    width: 500px;
}
.colorHistory {
    width: 46px;
    height: 46px;
    border-radius: 23px;
    margin-bottom: 4px;
    box-shadow: 4px 4px 4px -1px rgba(0,0,0,0.4) inset;
    margin-left: 4px;
}
.btn {
    padding: 10px;
    border: solid 1px #c3c;
    color: #c3c;
    border-radius: 10px;
}
.btn.download {
    width: 500px;
    margin: 10px auto;
}
</style>
