<script setup lang="ts">
const props = defineProps({
    respCorr: {
        type: Boolean,
        default: true
    },
    AhigherB: {
        type: Boolean,
        default: true
    }
});
</script>

<template>
    <div class="feedback-view">
        <div>{{ props.respCorr ? "正确答案" : "错误答案" }}</div>
        <div>{{ props.AhigherB ? "B 低于 A" : "B 高于 A" }}</div>
    </div>
</template>