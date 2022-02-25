<template>
    <div class="color-picker">
        <el-popover
            placement="bottom"
            width="250"
            trigger="click"
            content=""
            popper-class="color-picker-popover"
            v-model="visible"
        >
            <div
                id="canvas"
                slot="reference"
                class="trigger"
                :style="{
                    backgroundColor: value,
                }"
            ></div>
            <div v-if="visible" class="panel" ref="popover">
                <Panel
                    ref="panel"
                    theme="light"
                    :color="color.rgba"
                    :sucker-hide="!hasSucker"
                    :sucker-canvas="suckerCanvas"
                    :sucker-area="suckerArea"
                    :colors-theme="tColor"
                    :colors-gradient="colorsGradient"
                    :colors-standard="bColor"
                    @changeColor="changeColor"
                    @openSucker="openSucker"
                ></Panel>
                <div class="btns">
                    <el-button size="mini" @click="visible = false"
                        >取消</el-button
                    >
                    <el-button size="mini" @click="confirm">确认</el-button>
                </div>
            </div>
        </el-popover>
    </div>
</template>

<script>
import Panel from "./color/Index.vue";
import html2canvas from "html2canvas";
import Mixin from "./color/mixin";
export default {
    props: {
        value: {
            default: () => {},
        },
        selector: {
            default: "",
        },
    },
    mixins: [Mixin],
    data() {
        return {
            color: {},
            visible: false,
            suckerCanvas: null,
            suckerArea: [],
            isOpenSucker: false,
            tColor: [
                "#000000",
                "#ffffff",
                "#eeece1",
                "#1e497b",
                "#4e81bb",
                "#e2534d",
                "#9aba60",
                "#8165a0",
                "#47acc5",
                "#f9974c",
            ],
            colorConfig: [
                ["#7f7f7f", "#f2f2f2"],
                ["#0d0d0d", "#808080"],
                ["#1c1a10", "#ddd8c3"],
                ["#0e243d", "#c6d9f0"],
                ["#233f5e", "#dae5f0"],
                ["#632623", "#f2dbdb"],
                ["#4d602c", "#eaf1de"],
                ["#3f3150", "#e6e0ec"],
                ["#1e5867", "#d9eef3"],
                ["#99490f", "#fee9da"],
            ],
            bColor: [
                "#c21401",
                "#ff1e02",
                "#ffc12a",
                "#ffff3a",
                "#90cf5b",
                "#00af57",
                "#00afee",
                "#0071be",
                "#00215f",
                "#72349d",
            ],
        };
    },
    computed: {
        colorPanel() {
            let colorArr = [];
            for (let color of this.colorConfig) {
                colorArr.push(this.gradient(color[1], color[0], 5));
            }
            return colorArr;
        },
        colorsGradient() {
            let colors = [];
            for (let index = 0; index < 5; index++) {
                this.colorPanel.forEach((list) => {
                    colors.push(list[index]);
                });
            }
            return colors;
        },
        hasSucker() {
            return this.selector;
        },
    },
    watch: {
        value() {
            const rgba = (/rgba?\((.*?)\)/.exec(this.value) || [
                "",
                "0,0,0,1",
            ])[1].split(",");
            const [r, g, b, a] = rgba;
            this.color.rgba = {
                r,
                g,
                b,
                a,
            };
        },
        visible(nval) {
            if (!nval) {
                this.suckerCanvas?.remove();
            }
        },
    },
    methods: {
        changeColor(color) {
            this.color.rgba = color.rgba;
            // console.log(color);
        },
        openSucker(isOpen) {
            if (isOpen) {
                const dom = document.querySelector(this.selector);
                if (!dom) {
                    return;
                }
                const {
                    width: domW,
                    height: domH,
                    left,
                    top,
                } = dom.getBoundingClientRect();
                // 防止click阻塞动效
                setTimeout(() => {
                    html2canvas(dom, {
                        useCORS: true,
                    }).then((canvas) => {
                        // 弹窗提前关闭
                        if (!this.visible) return;
                        const ctx = canvas.getContext("2d");
                        if (!ctx) {
                            return;
                        }
                        dom.append(canvas);
                        Object.assign(canvas.style, {
                            position: "absolute",
                            left: 0 + "px",
                            top: 0 + "px",
                            opacity: 0,
                            zIndex: 998,
                            cursor: "none",
                            userSelect: "none",
                        });

                        this.suckerCanvas = canvas;
                        this.suckerArea = [left, top, domW + left, domH + top];
                    });
                }, 0);
            } else {
                this.suckerCanvas?.remove();
            }
        },
        confirm() {
            this.visible = false;
            const { r, g, b, a } = this.color.rgba;
            this.$emit("input", `rgba(${r},${g},${b},${a})`);
            this.$emit("change", `rgba(${r},${g},${b},${a})`);
            if (this.hasSucker) {
                // @ts-ignore
                this.$refs["panel"].$refs["sucker"].isOpenSucker = false;
            }
        },
        // 计算渐变过渡颜色
        gradient(startColor, endColor, step) {
            // 讲 hex 转换为 rgb
            let sColor = this.hexToRgb(startColor);
            let eColor = this.hexToRgb(endColor);
            // 计算R\G\B每一步的差值
            let rStep = (eColor[0] - sColor[0]) / step;
            let gStep = (eColor[1] - sColor[1]) / step;
            let bStep = (eColor[2] - sColor[2]) / step;
            let gradientColorArr = [];
            // 计算每一步的hex值
            for (let i = 0; i < step; i++) {
                gradientColorArr.push(
                    this.rgbToHex(
                        rStep * i + sColor[0],
                        gStep * i + sColor[1],
                        bStep * i + sColor[2],
                    ),
                );
            }
            return gradientColorArr;
        },
        // HEX 转 RGB 颜色
        hexToRgb(hex) {
            let color = this.parseColor(hex);
            let rgb = [];
            for (let i = 1; i < 7; i += 2) {
                rgb.push(parseInt("0x" + color.slice(i, i + 2)));
            }
            return rgb;
        },
        // RGB 颜色 转 HEX 颜色
        rgbToHex(r, g, b) {
            let hex = ((r << 16) | (g << 8) | b).toString(16);
            return "#" + new Array(Math.abs(hex.length - 7)).join("0") + hex;
        },
        // 格式化 hex 颜色值
        parseColor(hexStr) {
            if (hexStr.length === 4) {
                hexStr =
                    "#" +
                    hexStr[1] +
                    hexStr[1] +
                    hexStr[2] +
                    hexStr[2] +
                    hexStr[3] +
                    hexStr[3];
            }
            return hexStr;
        },
    },
    beforeDestroy() {
        this.suckerCanvas?.remove();
    },
    components: {
        Panel,
    },
};
</script>

<style lang="scss" scoped>
.color-picker {
    display: flex;
    align-items: center;
    height: 24px;
    cursor: pointer;
    span {
        background-repeat: round;
        background-image: url("data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAwAAAAMCAIAAADZF8uwAAAAGUlEQVQYV2M4gwH+YwCGIasIUwhT25BVBADtzYNYrHvv4gAAAABJRU5ErkJggg==");
    }
    .trigger {
        display: flex;
        width: 14px;
        height: 14px;
        border-radius: 0;
        border: 1px solid #c9ced9;
    }
    .test-img {
        position: fixed;
        top: 0;
        left: 0;
    }
}
</style>

<style lang="scss">
.color-picker-popover {
    padding: 0;
    background-color: #f7f8f9;
    .panel {
        .btns {
            display: flex;
            justify-content: flex-end;
            padding: 0 5px 5px;
        }
    }
}
</style>
