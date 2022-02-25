<template>
    <div class="color-alpha" @mousedown.prevent.stop="selectAlpha">
        <canvas ref="canvasAlpha" />
        <div :style="slideAlphaStyle" class="slide" />
    </div>
</template>

<script>
import mixin from "./mixin";
export default {
    mixins: [mixin],
    props: {
        color: {
            type: String,
            default: "#000000"
        },
        rgba: {
            type: Object,
            default: null
        },
        width: {
            type: Number,
            default: 100
        },
        height: {
            type: Number,
            default: 10
        }
    },
    data() {
        return {
            slideAlphaStyle: {},
            alphaSize: 5
        };
    },
    watch: {
        color() {
            this.renderColor();
        },
        "rgba.a"() {
            this.renderSlide();
        }
    },
    mounted() {
        this.renderColor();
        this.renderSlide();
    },
    methods: {
        renderColor() {
            const canvas = this.$refs.canvasAlpha;
            const width = this.width;
            const height = this.height;
            const size = this.height / 2;
            const canvasSquare = this.createAlphaSquare(size);

            const ctx = canvas.getContext("2d");
            canvas.width = width;
            canvas.height = height;

            ctx.fillStyle = ctx.createPattern(canvasSquare, "repeat");
            ctx.fillRect(0, 0, width, height);

            this.createLinearGradient("l", ctx, width, height, "rgba(255,255,255,0)", this.color);
        },
        renderSlide() {
            this.slideAlphaStyle = {
                left: this.rgba.a * this.width - 2 + "px"
            };
        },
        selectAlpha(e) {
            const { left: hueLeft } = this.$el.getBoundingClientRect();

            const mousemove = e => {
                let x = e.clientX - hueLeft;

                if (x < 0) {
                    x = 0;
                }
                if (x > this.width) {
                    x = this.width;
                }

                let a = parseFloat((x / this.width).toFixed(2));
                this.$emit("selectAlpha", a);
            };

            mousemove(e);

            const mouseup = () => {
                document.removeEventListener("mousemove", mousemove);
                document.removeEventListener("mouseup", mouseup);
            };

            document.addEventListener("mousemove", mousemove);
            document.addEventListener("mouseup", mouseup);
        }
    }
};
</script>

<style lang="scss" scoped>
.color-alpha {
    display: flex;
    align-items: center;
    position: relative;
    // margin-left: 8px;
    cursor: pointer;
    .slide {
        position: absolute;
        left: 0;
        top: 0px;
        height: 100%;
        width: 4px;
        background: #fff;
        border: 1px solid #f0f0f0;
        border-radius: 1px;
        box-shadow: 0 0 2px rgb(0 0 0 / 60%);
        pointer-events: none;
        box-sizing: border-box;
    }
}
</style>
