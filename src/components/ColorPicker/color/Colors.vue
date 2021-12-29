<template>
    <div class="color-list">
        <ul class="colors theme">
            <li
                v-for="item in colorsTheme"
                :key="item"
                class="item"
                @click="selectColor(item)"
            >
                <div
                    :style="{ background: `url(${imgAlphaBase64})` }"
                    class="alpha"
                />
                <div :style="{ background: item }" class="color" />
            </li>
        </ul>
        <ul class="colors gradient">
            <li
                v-for="item in colorsGradient"
                :key="item"
                class="item"
                @click="selectColor(item)"
            >
                <div
                    :style="{ background: `url(${imgAlphaBase64})` }"
                    class="alpha"
                />
                <div :style="{ background: item }" class="color" />
            </li>
        </ul>
        <ul class="colors standard">
            <li
                v-for="item in colorsStandard"
                :key="item"
                class="item"
                @click="selectColor(item)"
            >
                <div
                    :style="{ background: `url(${imgAlphaBase64})` }"
                    class="alpha"
                />
                <div :style="{ background: item }" class="color" />
            </li>
        </ul>
        <div>最近使用：</div>
        <ul v-if="colorsHistory.length" class="colors history">
            <li
                v-for="item in colorsHistory"
                :key="item"
                class="item"
                @click="selectColor(item)"
            >
                <div
                    :style="{ background: `url(${imgAlphaBase64})` }"
                    class="alpha"
                />
                <div :style="{ background: item }" class="color" />
            </li>
        </ul>
    </div>
</template>

<script>
import mixin from "./mixin";
export default {
    mixins: [mixin],
    props: {
        color: {
            type: [String, Object],
            default: () => {},
        },
        colorsTheme: {
            type: Array,
            default: () => [],
        },
        colorsGradient: {
            type: Array,
            default: () => [],
        },
        colorsStandard: {
            type: Array,
            default: () => [],
        },
        colorsHistoryKey: {
            type: String,
            default: "",
        },
    },
    data() {
        return {
            imgAlphaBase64: "",
            colorsHistory:
                JSON.parse(localStorage.getItem(this.colorsHistoryKey)) || [],
        };
    },
    created() {
        this.imgAlphaBase64 = this.createAlphaSquare(4).toDataURL();
    },
    destroyed() {
        this.setColorsHistory(this.color);
    },
    methods: {
        selectColor(color) {
            this.$emit("selectColor", color);
        },
        setColorsHistory(color) {
            if (!color) {
                return;
            }
            const colors = this.colorsHistory;
            const index = colors.indexOf(color);
            if (index >= 0) {
                colors.splice(index, 1);
            }
            if (colors.length >= 8) {
                colors.length = 7;
            }
            colors.unshift(color);
            this.colorsHistory = colors;
            localStorage.setItem(this.colorsHistoryKey, JSON.stringify(colors));
        },
    },
};
</script>

<style lang="scss" scoped>
.color-list {
    display: flex;
    flex-direction: column;
    row-gap: 6px;
    .colors {
        padding: 0;
        margin: 0;
        display: flex;
        justify-content: space-between;
        flex-wrap: wrap;
        column-gap: 6px;
        &.theme {
        }
        &.history {
            justify-content: flex-start;
        }
        .item {
            position: relative;
            width: 16px;
            height: 16px;
            // margin: 10px 0 0 6px;
            // border-radius: 3px;
            box-sizing: border-box;
            vertical-align: top;
            display: inline-block;
            transition: all 0.1s;
            cursor: pointer;
            &:hover {
                transform: scale(1.4);
                z-index: 2;
            }
            .alpha {
                height: 100%;
                // border-radius: 4px; // 大一像素，否则四个角会看到白点
            }
            .color {
                position: absolute;
                left: 0;
                top: 0;
                width: 100%;
                height: 100%;
                // border-radius: 3px;
            }
        }
    }
}
</style>
