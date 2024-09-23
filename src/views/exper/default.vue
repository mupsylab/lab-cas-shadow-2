<template>
    <div id="exp"></div>
</template>

<script setup lang="ts">
import { initJsPsych } from 'jspsych';
import { onMounted, render, h } from 'vue';

const sort = [1, 2, 3, 4];

import jsPsychHtmlKeyboardResponse from "@jspsych/plugin-html-keyboard-response";

import { useCheckBrowserInfo } from "../../store/browserCheck";
import { useLoaderAssets } from '../../store/loadAssetsToBlob';
import Session from '../../utils/session';
import { getUuid } from '../../utils/random';
const loader = useLoaderAssets();
const cbi = useCheckBrowserInfo();
const session = new Session();

const jsPsych = initJsPsych({
    display_element: "exp"
});

const timeline: object[] = [{
    type: jsPsychHtmlKeyboardResponse,
    choices: ["NO_KEYS"],
    stimulus: "<span id='a1'>0</span>/<span id='a2'>1</span>",
    on_load() {
        // 初始化代码放这里
        if (!loader.isInit) {
            loader.addAssets("./assets/imgs/inst_img_h.png");
            loader.addAssets("./assets/imgs/inst_img_v.png");
            // 进行加载，同时避免vue的热更新
            ["不平铺", "平铺"].forEach((v1) => {
                ["考虑遮挡", "不考虑遮挡"].forEach((v2) => {
                    const high = (function () {
                        const res = [];
                        for (let i = 0; i < 31; i++) {
                            if(v2 == "不考虑遮挡") {
                                res.push((0.5 + i * 0.1).toFixed(1));
                            } else if (v2 == "考虑遮挡") {
                                res.push((1.0 + i * 0.2).toFixed(1));
                            }
                        }
                        return res;
                    })();
                    high.forEach((v3) => {
                        loader.addAssets(`./assets/imgs/C面光/${v1}-${v2}/${v3}.png`);
                    });
                });
            });
        }
        loader.startLoad();

        const totalNumDom = document.querySelector("#a2") as HTMLDivElement;
        const countNumDom = document.querySelector("#a1") as HTMLDivElement;

        const i = setInterval(() => {
            if (cbi.isInit && loader.isFinish) {
                clearInterval(i);
                jsPsych.finishTrial({
                    type: "init-function"
                });
            } else {
                const { len, left, loading } = loader.progress;
                totalNumDom.innerText = len.toString();
                countNumDom.innerText = (len - left + loading).toFixed(2);
            }
        }, 100);
    }
}];

import trialDetail from '../instruction/trialDetail.vue';
import judgeCard from '../trial/judgeCard.vue';
import feedbackView from '../trial/feedbackView.vue';
import questionnaire from "../../components/questionnaire.vue";
const trial_instr = {
    type: jsPsychHtmlKeyboardResponse,
    choices: [" "],
    stimulus: "<div id='box'></div>",
    on_load() {
        const { ac, mc } = jsPsych.getAllTimelineVariables();
        render(h(trialDetail, {
            imgUrl2: loader.getAssets(ac == "平铺" ? "./assets/imgs/inst_img_v.png" : "./assets/imgs/inst_img_h.png"),
            imgUrl1: loader.getAssets(`./assets/imgs/C面光/${ac}-${mc}/${mc=="考虑遮挡"?"4.0":"2.0"}.png`),
            isHor: ac == "平铺"
        }), document.querySelector("#box") as Element);
    }
};
const trial = {
    type: jsPsychHtmlKeyboardResponse,
    choices: ["f", "j"],
    stimulus: "<div id='box'></div>",
    on_load() {
        const { ac, mc, highIndex, showQuick = false } = jsPsych.getAllTimelineVariables();
        const high = mc == "不考虑遮挡" ? (0.5 + highIndex * 0.1).toFixed(1) : (1.0 + highIndex * 0.2).toFixed(1);
        render(h(judgeCard, {
            imgUrl: loader.getAssets(`./assets/imgs/C面光/${ac}-${mc}/${high}.png`),
            isHor: ac == "平铺",
            showQuick: showQuick
        }), document.querySelector("#box") as Element);
    },
    on_finish(data: { response: "j" | "f" }) {
        const { ac, mc, highIndex } = jsPsych.getAllTimelineVariables();
        const high = mc == "不考虑遮挡" ? (0.5 + highIndex * 0.1).toFixed(1) : (1.0 + highIndex * 0.2).toFixed(1);
        const corrResp = highIndex > 15 ? "j" : "f";
        const corr = corrResp == data.response ? 1 : 0;
        jsPsych.data.write(Object.assign({}, {
            ac, mc, high, corrResp, corr
        }, data, {
            save: true,
            type: "trial"
        }));
    }
};
const fixation = {
    type: jsPsychHtmlKeyboardResponse,
    choices: ["NO_KEYS"],
    stimulus: "<div style='font-size: 64px; line-height: 64px; overflow: hidden;'>+</div>",
    trial_duration: 500
};
const feedback = {
    type: jsPsychHtmlKeyboardResponse,
    choices: ["NO_KEYS"],
    stimulus: "<div id='box'></div>",
    on_load() {
        const resp = jsPsych.data.get().last().values()[0];
        const { highIndex } = jsPsych.getAllTimelineVariables();

        const corrResp = highIndex > 15 ? "j" : "f";

        render(h(feedbackView, {
            respCorr: resp.response == corrResp,
            AhigherB: highIndex < 15
        }), document.querySelector("#box") as Element);
    },
    trial_duration: 1500
};
const ques = {
    type: jsPsychHtmlKeyboardResponse,
    choices: ["NO_KEYS"],
    stimulus: "<div style='display: flex;justify-content: center;align-items: center;' id='box'><div id='canvas'></div><div id='ques'></div></div>",
    on_load() {
        const { ac, mc, highIndex } = jsPsych.getAllTimelineVariables();
        const high = mc == "不考虑遮挡" ? (0.5 + highIndex * 0.1).toFixed(1) : (1.0 + highIndex * 0.2).toFixed(1);
        render(h(judgeCard, {
            imgUrl: loader.getAssets(`./assets/imgs/C面光/${ac}-${mc}/${high}.png`),
            isHor: ac == "平铺",
            isQues: true
        }), document.querySelector("#canvas") as Element);
        render(h(questionnaire, {
            title: "美观和自然度评估",
            desc: "设计师希望用B卡的阴影效果来模拟当前A卡的高度，请评价B卡片阴影的美观和自然程度。",
            questionList: [
                { type: "radio", name: "natural", title: "我认为 B 的阴影", desc: "", choices: ["很不自然", "较不自然", "略不自然", "略自然", "较自然", "很自然"] },
                { type: "radio", name: "beaut", title: "我认为 B 的阴影", desc: "", choices: ["很不美观", "较不美观", "略不美观", "略美观", "较美观", "很美观"] }
            ],
            onEndTrial(data) {
                jsPsych.finishTrial(Object.assign({}, {
                    mc, ac, high
                }, data, {
                    save: true,
                    type: "ques"
                }));
            }
        }), document.querySelector("#ques") as Element);
    }
};
// 实验流程开始
import InstTotal from '../instruction/instTotal.vue';
import InstPart1 from '../instruction/instPart1.vue';
import InstPart1Prac from '../instruction/instPart1Prac.vue';
import InstPart1Form from '../instruction/instPart1Form.vue';
import InstPart2 from '../instruction/instPart2.vue';
timeline.push({
    type: jsPsychHtmlKeyboardResponse,
    choices: [" "],
    stimulus: "<div id='box'></div>",
    on_load() {
        render(h(InstTotal), document.querySelector("#box") as Element);
    }
});
timeline.push({
    type: jsPsychHtmlKeyboardResponse,
    choices: [" "],
    stimulus: "<div id='box'></div>",
    on_load() {
        render(h(InstPart1), document.querySelector("#box") as Element);
    }
});
// 练习阶段
timeline.push({
    type: jsPsychHtmlKeyboardResponse,
    choices: [" "],
    stimulus: "<div id='box'></div>",
    on_load() {
        render(h(InstPart1Prac), document.querySelector("#box") as Element);
    }
});
timeline.push({
    timeline: [{
        timeline: [trial_instr],
        timeline_variables: [{ mc: "不考虑遮挡" }]
    }, {
        timeline: [fixation, trial, feedback],
        timeline_variables: [
            { mc: "不考虑遮挡", highIndex: 0, showQuick: false },
            { mc: "考虑遮挡", highIndex: 30, showQuick: false },
            { mc: "考虑遮挡", highIndex: 16, showQuick: true },
        ]
    }],
    timeline_variables: [
        { ac: "平铺" },
        { ac: "不平铺" }
    ],
    loop_function() {
        const trials = jsPsych.data.get().filter({save: true}).last(6).values();
        let corrVal = 0;
        trials.forEach(v => {
            corrVal += v.corr;
        });
        if (corrVal / 6 < 0.8) {
            alert("错误率过高, 请重新作答");
            return true;
        }
        else return false;
    }
});
// 正式阶段
timeline.push({
    type: jsPsychHtmlKeyboardResponse,
    choices: [" "],
    stimulus: "<div id='box'></div>",
    on_load() {
        render(h(InstPart1Form), document.querySelector("#box") as Element);
    }
});
timeline.push({
    timeline: [trial_instr, {
        timeline: [fixation, trial],
        timeline_variables: (function () {
            const res = [];
            for (let i = 0; i < 31; i++) {
                res.push({ highIndex: i });
            }
            return jsPsych.randomization.shuffle(res);
        })(),
        repetitions: 2,
        radomization: true
    }],
    timeline_variables: (function () {
        const rr = [
            { mc: "不考虑遮挡", ac: "平铺" },
            { mc: "考虑遮挡", ac: "平铺" },
            { mc: "不考虑遮挡", ac: "不平铺" },
            { mc: "考虑遮挡", ac: "不平铺" }
        ];
        const r: { mc: string; ac: string; }[] = [];
        sort.forEach(i => { r.push(rr[i - 1]) });
        return r;
    })()
});
// 问卷部分
timeline.push({
    type: jsPsychHtmlKeyboardResponse,
    choices: [" "],
    stimulus: "<div id='box'></div>",
    on_load() {
        render(h(InstPart2), document.querySelector("#box") as Element);
    }
});
timeline.push({
    timeline: [ques],
    timeline_variables: [
        { mc: "不考虑遮挡", ac: "不平铺", highIndex: 0 },
        { mc: "不考虑遮挡", ac: "不平铺", highIndex: 15 },
        { mc: "不考虑遮挡", ac: "不平铺", highIndex: 30 },
        { mc: "不考虑遮挡", ac: "平铺", highIndex: 0 },
        { mc: "不考虑遮挡", ac: "平铺", highIndex: 15 },
        { mc: "不考虑遮挡", ac: "平铺", highIndex: 30 },
        { mc: "考虑遮挡", ac: "不平铺", highIndex: 0 },
        { mc: "考虑遮挡", ac: "不平铺", highIndex: 15 },
        { mc: "考虑遮挡", ac: "不平铺", highIndex: 30 },
        { mc: "考虑遮挡", ac: "平铺", highIndex: 0 },
        { mc: "考虑遮挡", ac: "平铺", highIndex: 15 },
        { mc: "考虑遮挡", ac: "平铺", highIndex: 30 },
    ],
    radomization: true
});
import endExp from "./endExp.vue";
timeline.push({
    type: jsPsychHtmlKeyboardResponse,
    choices: ["NO_KEYS"],
    stimulus: "<div id='box'></div>",
    on_load() {
        jsPsych.data.write(cbi.browser);
        session.offlineSave(jsPsych.data.get().filter({save:true}).csv(), getUuid());
        render(h(endExp), document.querySelector("#box") as Element);
    }
});

onMounted(() => {
    jsPsych.run(timeline);
});
</script>