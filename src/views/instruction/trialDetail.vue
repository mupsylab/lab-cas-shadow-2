<script setup lang="ts">
const props = defineProps({
    imgUrl1: {
        type: String,
        default: ""
    },
    imgUrl2: {
        type: String,
        default: ""
    },
    isHor: {
        type: Boolean,
        default: true
    }
})
</script>

<template>
    <div class="trial-detail-box">
        <div class="word-box">
            <p v-if="props.isHor">您接下来将会看到如左图所示的A, B, C三张卡片, 它们分别放置在背景D上方。</p>
            <p v-else>您接下来将会看到如左图所示的A, B, C三张卡片, 它们分别放置在背景D上方, A和D之间, 有卡片C遮挡。</p>
            <p>您需要<strong>比较A、B卡片相对于背景D的高度</strong> (如右边的侧视图所示)</p>
        </div>
        <div class="img-box">
            <div :style="`--url: url(${props.imgUrl1})`" :class="{img1: true, hor: props.isHor, ver: !props.isHor}">
                <div class="a">A</div>
                <div class="b">B</div>
                <div class="c">C</div>
                <div class="d">D</div>
            </div>
            <div :style="`--url: url(${props.imgUrl2})`"></div>
        </div>
        <div class="word-box button">按空格键进行实验</div>
    </div>
</template>

<style scoped>
.trial-detail-box {
    width: 1280px;
}
.word-box {
    text-align: center;
}
.button {
    font-size: 24px;
    line-height: 32px;
}
.img-box>div {
    display: inline-block;
    width: 640px;
    height: 360px;
    background-image: var(--url);
    background-size: cover;
}
.img1 {
    position: relative;
}
.img1>div {
    position: absolute;
    color: var(--dark-idegrey-1);
}
.img1.hor .a {
    top: 170px;
    left: 65px;
}
.img1.hor .b {
    top: 170px;
    left: 565px;
}
.img1.hor .c {
    top: 170px;
    left: 320px;
}
.img1.hor .d {
    top: 300px;
    left: 320px;
}

.img1.ver .a {
    top: 170px;
    left: 320px;
}
.img1.ver .b {
    top: 170px;
    left: 565px;
}
.img1.ver .c {
    top: 250px;
    left: 400px;
}
.img1.ver .d {
    top: 300px;
    left: 400px;
}
</style>