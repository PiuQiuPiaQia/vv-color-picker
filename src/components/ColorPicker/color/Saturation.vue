<template>
    <div class="saturation" @mousedown.prevent.stop="selectSaturation">
        <canvas ref="canvasSaturation" />
        <div :style="slideSaturationStyle" class="slide" />
    </div>
</template>

<script>
import mixin from "./mixin";
export default {
    mixins: [mixin],
    props: {
        color: {
            type: String,
            default: "#000000",
        },
        hsv: {
            type: Object,
            default: null,
        },
        width: {
            type: Number,
            default: 152,
        },
    },
    data() {
        return {
            slideSaturationStyle: {},
        };
    },
    computed: {
        height() {
            return this.width * 0.6
        }
    },
    // 不能监听，否则自己改变自己时，颜色也会发生变化
    // watch: {
    //     color() {
    //         this.renderColor()
    //     }
    // },
    mounted() {
        this.renderColor();
        this.renderSlide();
    },
    methods: {
        renderColor() {
            const canvas = this.$refs.canvasSaturation;
            const width = this.width;
            const height = this.height;
            const ctx = canvas.getContext("2d");
            canvas.width = width;
            canvas.height = height;

            // 纯色底色
            ctx.fillStyle = this.color;
            ctx.fillRect(0, 0, width, height);

            // 横向纯白到透明白
            this.createLinearGradient("l", ctx, width, height, "rgba(255,255,255,1)", "rgba(255,255,255,0)");
            // 纵向透明黑到纯黑
            this.createLinearGradient("p", ctx, width, height, "rgba(0,0,0,0)", "rgba(0,0,0,1)");
        },
        renderSlide() {
            this.slideSaturationStyle = {
                left: this.hsv.s * this.width - 5 + "px",
                top: (1 - this.hsv.v) * this.height - 5 + "px",
            };
        },
        selectSaturation(e) {
            const { top: saturationTop, left: saturationLeft } = this.$el.getBoundingClientRect();
            const ctx = e.target.getContext("2d");

            const mousemove = (e) => {
                let x = e.clientX - saturationLeft;
                let y = e.clientY - saturationTop;

                if (x < 0) {
                    x = 0;
                }
                if (y < 0) {
                    y = 0;
                }
                if (x > this.width) {
                    x = this.width;
                }
                if (y > this.height) {
                    y = this.height;
                }

                // 不通过监听数据变化来修改dom，否则当颜色为#ffffff时，slide会跑到左下角
                this.slideSaturationStyle = {
                    left: x - 5 + "px",
                    top: y - 5 + "px",
                };
                // 如果用最大值，选择的像素会是空的，空的默认是黑色
                const imgData = ctx.getImageData(Math.min(x, this.width - 1), Math.min(y, this.height - 1), 1, 1);
                const [r, g, b] = imgData.data;
                this.$emit("selectSaturation", { r, g, b });
            };

            mousemove(e);

            const mouseup = () => {
                document.removeEventListener("mousemove", mousemove);
                document.removeEventListener("mouseup", mouseup);
            };

            document.addEventListener("mousemove", mousemove);
            document.addEventListener("mouseup", mouseup);
        },
    },
};
</script>

<style lang="scss" scoped>
.saturation {
    display: flex;
    position: relative;
    cursor: pointer;
    .slide {
        position: absolute;
        left: 100px;
        top: 0;
        width: 10px;
        height: 10px;
        border-radius: 50%;
        border: 1px solid #fff;
        box-shadow: 0 0 1px 1px rgba(0, 0, 0, 0.3);
        pointer-events: none;
    }
}
</style>
