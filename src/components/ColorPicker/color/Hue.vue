<template>
    <div
        class="hue"
        @mousedown.prevent.stop="selectHue"
    >
        <canvas ref="canvasHue" />
        <div
            :style="slideHueStyle"
            class="slide"
        />
    </div>
</template>

<script>
export default {
    props: {
        hsv: {
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
            slideHueStyle: {}
        }
    },
    // 不能监听，否则操作saturation时，这里的slide会抖动
    // watch: {
    //     'hsv.h'() {
    //         this.renderSlide()
    //     }
    // },
    mounted() {
        this.renderColor()
        this.renderSlide()
    },
    methods: {
        renderColor() {
            const canvas = this.$refs.canvasHue
            const width = this.width
            const height = this.height
            const ctx = canvas.getContext('2d')
            canvas.width = width
            canvas.height = height

            const gradient = ctx.createLinearGradient(0, height, width, height)
            gradient.addColorStop(0, '#FF0000') // 红
            gradient.addColorStop(0.17 * 1, '#FFFF00') // 紫
            gradient.addColorStop(0.17 * 2, '#00FF00') // 蓝
            gradient.addColorStop(0.17 * 3, '#00FFFF') // 青
            gradient.addColorStop(0.17 * 4, '#0000FF') // 绿
            gradient.addColorStop(0.17 * 5, '#FF00FF') // 黄
            gradient.addColorStop(1, '#FF0000') // 红
            ctx.fillStyle = gradient
            ctx.fillRect(0, 0, width, height)
        },
        renderSlide() {
            this.slideHueStyle = {
                left: (1 - this.hsv.h / 360) * this.width - 2 + 'px'
            }
        },
        selectHue(e) {
            const { left: hueLeft } = this.$el.getBoundingClientRect()
            const ctx = e.target.getContext('2d')

            const mousemove = e => {
                let x = e.clientX - hueLeft

                if (x < 0) {
                    x = 0
                }
                if (x > this.width) {
                    x = this.width
                }

                this.slideHueStyle = {
                    left: x - 2 + 'px'
                }
                // 如果用最大值，选择的像素会是空的，空的默认是黑色
                const imgData = ctx.getImageData(Math.min(x, this.width - 1), 0, 1, 1)
                const [r, g, b] = imgData.data
                this.$emit('selectHue', { r, g, b })
            }

            mousemove(e)

            const mouseup = () => {
                document.removeEventListener('mousemove', mousemove)
                document.removeEventListener('mouseup', mouseup)
            }

            document.addEventListener('mousemove', mousemove)
            document.addEventListener('mouseup', mouseup)
        }
    }
}
</script>

<style lang="scss" scoped>
.hue {
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
