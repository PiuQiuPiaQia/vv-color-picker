<template>
    <div class="hu-color-picker" :class="{ light: isLightTheme }" :style="{ width: saturationWidth + 'px' }">
        <Saturation
            ref="saturation"
            :color="rgbString"
            :hsv="hsv"
            :width="saturationWidth"
            @selectSaturation="selectSaturation"
        />
        <div class="color-set">
            <div class="preview">
                <Preview :color="rgbaString" :width="previewHeight" :height="previewHeight" />
            </div>
            <div class="set">
                <Hue ref="hue" :hsv="hsv" :width="hueWidth" :height="hueHeight" @selectHue="selectHue" />
                <Alpha
                    ref="alpha"
                    :color="rgbString"
                    :rgba="rgba"
                    :width="hueWidth"
                    :height="hueHeight"
                    @selectAlpha="selectAlpha"
                />
            </div>
        </div>
        <Colors
            :color="rgbaString"
            :colorsTheme="colorsTheme"
            :colorsGradient="colorsGradient"
            :colorsStandard="colorsStandard"
            :colors-history-key="colorsHistoryKey"
            @selectColor="selectColor"
        />
        <div class="color-show">
            <Sucker
                ref="sucker"
                v-if="!suckerHide"
                :sucker-canvas="suckerCanvas"
                :sucker-area="suckerArea"
                @openSucker="openSucker"
                @selectSucker="selectSucker"
            />
            <Box name="HEX" :color="modelHex" @inputColor="inputHex" />
            <!-- <Box name="RGBA" :color="modelRgba" @inputColor="inputRgba" /> -->
        </div>
    </div>
</template>

<script>
import mixin from "./mixin";
import Saturation from "./Saturation.vue";
import Hue from "./Hue.vue";
import Alpha from "./Alpha.vue";
import Preview from "./Preview.vue";
import Sucker from "./Sucker.vue";
import Box from "./Box.vue";
import Colors from "./Colors.vue";
export default {
    components: {
        Saturation,
        Hue,
        Alpha,
        Preview,
        Sucker,
        Box,
        Colors
    },
    mixins: [mixin],
    props: {
        color: {
            type: String | Object,
            default: "#000000"
        },
        theme: {
            type: String,
            default: "dark"
        },
        suckerHide: {
            type: Boolean,
            default: true
        },
        suckerCanvas: {
            type: null, // HTMLCanvasElement
            default: null
        },
        suckerArea: {
            type: Array,
            default: () => []
        },
        colorsTheme: {
            type: Array,
            default: () => []
        },
        colorsGradient: {
            type: Array,
            default: () => []
        },
        colorsStandard: {
            type: Array,
            default: () => []
        },
        colorsHistoryKey: {
            type: String,
            default: "vue-colorpicker-history"
        }
    },
    data() {
        return {
            saturationWidth: 230,
            previewHeight: 30,
            hueHeight: 12,
            modelRgba: "",
            modelHex: "",
            r: 0,
            g: 0,
            b: 0,
            a: 1,
            h: 0,
            s: 0,
            v: 0
        };
    },
    computed: {
        isLightTheme() {
            return this.theme === "light";
        },
        hueWidth() {
            return this.saturationWidth - this.previewHeight - 10;
        },
        previewWidth() {
            return this.totalWidth - (this.suckerHide ? 0 : this.previewHeight);
        },
        rgba() {
            return {
                r: this.r,
                g: this.g,
                b: this.b,
                a: this.a
            };
        },
        hsv() {
            return {
                h: this.h,
                s: this.s,
                v: this.v
            };
        },
        rgbString() {
            return `rgb(${this.r}, ${this.g}, ${this.b})`;
        },
        rgbaStringShort() {
            return `${this.r}, ${this.g}, ${this.b}, ${this.a}`;
        },
        rgbaString() {
            return `rgba(${this.rgbaStringShort})`;
        },
        hexString() {
            return this.rgb2hex(this.rgba, true);
        }
    },
    created() {
        Object.assign(this, this.setColorValue(this.color));
        this.setText();

        // 避免初始化时，也会触发changeColor事件
        this.$watch("rgba", () => {
            this.$emit("changeColor", {
                rgba: this.rgba,
                hsv: this.hsv,
                hex: this.modelHex
            });
        });
    },
    methods: {
        selectSaturation(color) {
            const { r, g, b, h, s, v } = this.setColorValue(color);
            Object.assign(this, { r, g, b, h, s, v });
            this.setText();
        },
        selectHue(color) {
            const { r, g, b, h, s, v } = this.setColorValue(color);
            Object.assign(this, { r, g, b, h, s, v });
            this.setText();
            this.$nextTick(() => {
                this.$refs.saturation.renderColor();
                this.$refs.saturation.renderSlide();
            });
        },
        selectAlpha(a) {
            this.a = a;
            this.setText();
        },
        inputHex(color) {
            const { r, g, b, a, h, s, v } = this.setColorValue(color);
            Object.assign(this, { r, g, b, a, h, s, v });
            this.modelHex = color;
            this.modelRgba = this.rgbaStringShort;
            this.$nextTick(() => {
                this.$refs.saturation.renderColor();
                this.$refs.saturation.renderSlide();
                this.$refs.hue.renderSlide();
            });
        },
        inputRgba(color) {
            const { r, g, b, a, h, s, v } = this.setColorValue(color);
            Object.assign(this, { r, g, b, a, h, s, v });
            this.modelHex = this.hexString;
            this.modelRgba = color;
            this.$nextTick(() => {
                this.$refs.saturation.renderColor();
                this.$refs.saturation.renderSlide();
                this.$refs.hue.renderSlide();
            });
        },
        setText() {
            this.modelHex = this.hexString;
            this.modelRgba = this.rgbaStringShort;
        },
        openSucker(isOpen) {
            this.$emit("openSucker", isOpen);
        },
        selectSucker(color) {
            const { r, g, b, a, h, s, v } = this.setColorValue(color);
            Object.assign(this, { r, g, b, a, h, s, v });
            this.setText();
            this.$nextTick(() => {
                this.$refs.saturation.renderColor();
                this.$refs.saturation.renderSlide();
                this.$refs.hue.renderSlide();
            });
        },
        selectColor(color) {
            const { r, g, b, a, h, s, v } = this.setColorValue(color);
            Object.assign(this, { r, g, b, a, h, s, v });
            this.setText();
            this.$nextTick(() => {
                this.$refs.saturation.renderColor();
                this.$refs.saturation.renderSlide();
                this.$refs.hue.renderSlide();
            });
        }
    }
};
</script>

<style lang="scss" scoped>
.hu-color-picker {
    display: flex;
    flex-direction: column;
    row-gap: 10px;
    padding: 10px;
    background: #1d2024;
    border-radius: 4px;
    // box-shadow: 0 0 16px 0 rgba(0, 0, 0, 0.16);
    z-index: 1;
    &.light {
        background: #f7f8f9;
    }
    .color-set {
        display: flex;
        justify-content: space-between;
        // margin-bottom: 4px;
        .preview {
            display: flex;
        }
        .set {
            display: flex;
            flex-direction: column;
            justify-content: space-between;
        }
    }
    .color-show {
        // margin-top: 8px;
        display: flex;
        align-items: center;
        column-gap: 10px;
    }
}
</style>
