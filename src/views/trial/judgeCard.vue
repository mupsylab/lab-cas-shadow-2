<script setup lang="ts">
import { onMounted } from 'vue';

const props = defineProps({
    imgUrl: {
        type: String,
        default: ""
    },
    isHor: {
        type: Boolean,
        default: true
    },
    isQues: {
        type: Boolean,
        default: false
    },
    showQuick: {
        type: Boolean,
        default: false
    }
});
const emits = defineEmits(["endTrial"]);

onMounted(() => {
    const canvas = document.querySelector("canvas") as HTMLCanvasElement;
    const ctx = canvas.getContext("2d") as CanvasRenderingContext2D;

    const img = new Image();
    img.onload = function() {
        ctx.clearRect(0, 0, 1282, 718);
        // 图片实际大小: 1923, 1077
        ctx.drawImage(img, 0, 0, 1923, 1077, 0, 0, 1282, 718);
    }
    img.src = props.imgUrl;
});
</script>

<template>
    <div class="judge-card-box">
        <canvas ref="canvas" width="1282" height="718"></canvas>
        <div v-if="!props.isQues" style="text-align: center; font-size: 24px; line-height: 32px;">
            <p>认为A比较高, 请按“F”键; 认为B比较高, 请选择“J” 键。</p>
            <p v-if="props.showQuick" style="font-weight: 700;">即使你认为A和B一样高，仍然需要从A、B中尽快做出选择！</p>
        </div>
        <div :class="{ tag: true, hor: props.isHor, ver: !props.isHor }">
            <div class="a">A</div>
            <div class="b">B</div>
            <div class="c">C</div>
            <div class="d">D</div>
        </div>
    </div>
</template>

<style scoped>
.judge-card-box {
    position: relative;
}

.tag {
    display: block;
    position: absolute;
    top: 0;
    left: 0;
    color: var(--dark-idegrey-1);
}
.tag>div {
    position: absolute;
    font-size: 24px;
    line-height: 24px;
}
.tag.hor .a {
    top: 345px;
    left: 135px;
}
.tag.hor .b {
    top: 345px;
    left: 1135px;
}
.tag.hor .c {
    top: 345px;
    left: 640px;
}
.tag.hor .d {
    top: 610px;
    left: 640px;
}

.tag.ver .a {
    top: 345px;
    left: 640px;
}
.tag.ver .b {
    top: 345px;
    left: 1135px;
}
.tag.ver .c {
    top: 505px;
    left: 790px;
}
.tag.ver .d {
    top: 610px;
    left: 790px;
}
</style>