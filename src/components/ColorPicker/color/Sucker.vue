<template>
    <div class="container">
        <svg
            v-if="!isSucking"
            :class="{ active: isOpenSucker }"
            class="sucker"
            xmlns="http://www.w3.org/2000/svg"
            viewBox="-12 -12 48 48"
            @click="openSucker"
        >
            <path
                d="M13.1,8.2l5.6,5.6c0.4,0.4,0.5,1.1,0.1,1.5s-1.1,0.5-1.5,0.1c0,0-0.1,0-0.1-0.1l-1.4-1.4l-7.7,7.7C7.9,21.9,7.6,22,7.3,22H3.1C2.5,22,2,21.5,2,20.9l0,0v-4.2c0-0.3,0.1-0.6,0.3-0.8l5.8-5.8C8.5,9.7,9.2,9.6,9.7,10s0.5,1.1,0.1,1.5c0,0,0,0.1-0.1,0.1l-5.5,5.5v2.7h2.7l7.4-7.4L8.7,6.8c-0.5-0.4-0.5-1-0.1-1.5s1.1-0.5,1.5-0.1c0,0,0.1,0,0.1,0.1l1.4,1.4l3.5-3.5c1.6-1.6,4.1-1.6,5.8-0.1c1.6,1.6,1.6,4.1,0.1,5.8L20.9,9l-3.6,3.6c-0.4,0.4-1.1,0.5-1.5,0.1"
            />
        </svg>
        <svg
            v-if="isSucking"
            class="sucker"
            viewBox="-16 -16 68 68"
            xmlns="http://www.w3.org/2000/svg"
            stroke="#9099a4"
        >
            <g fill="none" fill-rule="evenodd">
                <g transform="translate(1 1)" stroke-width="4">
                    <circle stroke-opacity=".5" cx="18" cy="18" r="18" />
                    <path d="M36 18c0-9.94-8.06-18-18-18">
                        <animateTransform
                            attributeName="transform"
                            type="rotate"
                            from="0 18 18"
                            to="360 18 18"
                            dur="1s"
                            repeatCount="indefinite"
                        />
                    </path>
                </g>
            </g>
        </svg>
    </div>
</template>

<script>
export default {
    props: {
        suckerCanvas: {
            type: null, // HTMLCanvasElement
            default: null,
        },
        suckerArea: {
            type: Array,
            default: () => [],
        },
    },
    data() {
        return {
            isOpenSucker: false, // 是否处于吸管状态
            suckerPreview: null, // 吸管旁边的预览颜色dom
            suckerSize: 130, // 吸管大小
            isSucking: false, // 是否处于吸管等待状态
            gridSize: 10,
        };
    },
    computed: {
        gridR() {
            return this.suckerSize / this.gridSize;
        },
        gridC() {
            return this.suckerSize / this.gridSize;
        },
    },
    watch: {
        suckerCanvas(newVal) {
            this.isSucking = false;
            this.suckColor(newVal);
            // newVal.style.cursor = `url(${imgSucker}) 0 32, default`;
        },
    },
    methods: {
        openSucker() {
            if (!this.isOpenSucker) {
                this.isOpenSucker = true;
                this.isSucking = true;
                this.$emit("openSucker", true);
                document.addEventListener("keydown", this.keydownHandler);
            } else {
                // 和按下esc键的处理逻辑一样
                this.keydownHandler({ keyCode: 27 });
            }
        },
        keydownHandler(e) {
            console.log("down");
            // esc
            if (e.keyCode === 27) {
                this.isOpenSucker = false;
                this.isSucking = false;
                this.$emit("openSucker", false);
                document.removeEventListener("keydown", this.keydownHandler);
                document.removeEventListener("mousemove", this.mousemoveHandler);
                document.removeEventListener("mouseup", this.mousemoveHandler);
                if (this.suckerPreview) {
                    this.suckerPreview.remove();
                    this.suckerPreview = null;
                }
            }
        },
        async mousemoveHandler(e) {
            const { clientX, clientY } = e;
            const { top: domTop, left: domLeft} = this.suckerCanvas.getBoundingClientRect();
            // 添加像素预览
            this.addSelection(e);

            const style = this.suckerPreview.style;
            const suckerSize = this.suckerSize;
            Object.assign(style, {
                position: "absolute",
                left: clientX - domLeft + "px",
                top: clientY - domTop + "px",
                transform: "translate(-50%, -50%)",
                width: suckerSize + "px",
                height: suckerSize + "px",
                borderRadius: "50%",
                // border: "1px solid #fff",
                boxShadow: "0 0 8px 0 rgba(0, 0, 0, 0.16)",
                cursor: "none",
                // background: `rgba(${r}, ${g}, ${b}, ${a})`,
                zIndex: 999, // 吸管的小圆圈预览色的层级不能超过颜色选择器
            });
            if (
                clientX >= this.suckerArea[0] &&
                clientY >= this.suckerArea[1] &&
                clientX <= this.suckerArea[2] &&
                clientY <= this.suckerArea[3]
            ) {
                style.display = "";
            } else {
                style.display = "none";
            }
        },
        async suckColor(dom) {
            if (dom && dom.tagName !== "CANVAS") {
                return;
            }
            const dpi = window.devicePixelRatio;

            // 创建吸盘
            this.suckerPreview = document.createElement("canvas");
            // 设置预览canvas的像素比为系统像素比
            this.suckerPreview.width = this.suckerSize * dpi;
            this.suckerPreview.height = this.suckerSize * dpi;
            dom.parentElement.append(this.suckerPreview);

            document.addEventListener("mousemove", this.mousemoveHandler);
            document.addEventListener("mouseup", this.mousemoveHandler);

            this.suckerPreview.addEventListener("click", (e) => {
                e.stopPropagation();
                e.preventDefault();
                const gridSize = this.gridSize;
                const col = this.gridC,
                    row = this.gridC;
                const ctx = this.suckerPreview.getContext("2d");
                // 获取preview的中心点颜色
                const imgData = ctx.getImageData(
                    ((col - 1) / 2 + 0.5) * gridSize * dpi,
                    ((row - 1) / 2 + 0.5) * gridSize * dpi,
                    1,
                    1
                );
                let [r, g, b, a] = imgData.data;
                a = parseFloat((a / 255).toFixed(2));
                this.$emit("selectSucker", { r, g, b, a });
            });
        },
        addSelection(e) {
            const dpi = window.devicePixelRatio;
            // 网格大小
            var girdSize = this.gridSize * dpi;
            const ctx = this.suckerPreview.getContext("2d");
            var canvasWidth = ctx.canvas.width;
            var canvasHeight = ctx.canvas.height;
            // 绘制中心点
            var gridR = this.gridR; // x轴条数
            var gridC = this.gridC; // y轴的条数
            // 像素颜色
            const canvasCtx = this.suckerCanvas.getContext("2d");

            const { top: domTop, left: domLeft, width, height } = this.suckerCanvas.getBoundingClientRect();
            const { clientX, clientY } = e;
            const x = clientX - domLeft;
            const y = clientY - domTop;
            for (var i = 0; i < gridR; i++) {
                for (var j = 0; j < gridC; j++) {
                    const originX = Math.min(x + i - (gridC - 1) / 2, width - 1) * dpi,
                        originY = Math.min(y + j - (gridR - 1) / 2, height - 1) * dpi;
                    // 鼠标坐标 + 网格坐标 - 负向移动一半的网格
                    // 乘以dpi是初始化时候使用了系统像素比
                    const currentData = canvasCtx.getImageData(originX, originY, 1, 1);
                    let [r, g, b, a] = currentData.data;
                    ctx.fillStyle = `rgba(${r},${g},${b},${a / 255})`;
                    ctx.fillRect(i * girdSize, j * girdSize, (i + 1) * girdSize, (j + 1) * girdSize);
                }
            }
            // 网格线
            for (let i = 0; i < gridR; i++) {
                ctx.beginPath();
                ctx.moveTo(0, girdSize * i - 0.5); // -0.5是为了解决像素模糊问题
                ctx.lineTo(canvasWidth, girdSize * i - 0.5);
                ctx.strokeStyle = "#ccc";
                ctx.stroke();
            }
            for (let j = 0; j < gridC; j++) {
                ctx.beginPath();
                ctx.moveTo(girdSize * j, 0);
                ctx.lineTo(girdSize * j, canvasHeight);
                ctx.strokeStyle = "#ccc";
                ctx.stroke();
            }
            // 中心方块
            ctx.beginPath();
            ctx.moveTo(((gridC - 1) / 2) * girdSize, ((gridR - 1) / 2) * girdSize);
            ctx.lineTo(((gridC + 1) / 2) * girdSize, ((gridR - 1) / 2) * girdSize);
            ctx.strokeStyle = "red";
            ctx.stroke();
            ctx.beginPath();
            ctx.moveTo(((gridC - 1) / 2) * girdSize, ((gridR - 1) / 2 + 1) * girdSize);
            ctx.lineTo(((gridC + 1) / 2) * girdSize, ((gridR - 1) / 2 + 1) * girdSize);
            ctx.strokeStyle = "red";
            ctx.stroke();

            ctx.beginPath();
            ctx.moveTo(((gridC - 1) / 2) * girdSize, ((gridR - 1) / 2) * girdSize);
            ctx.lineTo(((gridC - 1) / 2) * girdSize, ((gridR - 1) / 2 + 1) * girdSize);
            ctx.strokeStyle = "red";
            ctx.stroke();
            ctx.beginPath();
            ctx.moveTo(((gridC - 1) / 2 + 1) * girdSize, ((gridR - 1) / 2) * girdSize);
            ctx.lineTo(((gridC - 1) / 2 + 1) * girdSize, ((gridR - 1) / 2 + 1) * girdSize);
            ctx.strokeStyle = "red";
            ctx.stroke();
        },
    },
    beforeDestroy() {
        this.suckerPreview?.remove();
    },
};
</script>

<style lang="scss" scoped>
.container {
    width: 30px;
    height: 30px;
    display: flex;
    .sucker {
        width: 30px;
        fill: #9099a4;
        // background: #2e333a;
        background: #eceef0;
        cursor: pointer;
        transition: all 0.3s;
        &:hover,
        &.active {
            fill: #1593ff;
        }
    }
}
</style>
