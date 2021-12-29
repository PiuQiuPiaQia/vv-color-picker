<template>
    <div class="color-type">
        <span class="name">
            {{ name }}
        </span>
        <input v-model="modelColor" class="value" />
    </div>
</template>

<script>
export default {
    props: {
        name: {
            type: String,
            default: ""
        },
        color: {
            type: String,
            default: ""
        }
    },
    computed: {
        modelColor: {
            get() {
                return this.color;
            },
            set(val) {
                this.$emit("inputColor", val);
            }
        }
    }
};
</script>

<style lang="scss" scoped>
.color-type {
    display: flex;
    justify-content: space-between;
    // margin-top: 8px;
    font-size: 12px;
    overflow: hidden;
    .name {
        width: 60px;
        height: 30px;
        float: left;
        display: flex;
        justify-content: center;
        align-items: center;
        color: #999;
        // background: #252930;
        background: #e7e8e9;
    }
    .value {
        flex: 1;
        height: 30px;
        min-width: 100px; // 可以让flex起作用
        padding: 0 12px;
        border: 0;
        box-sizing: border-box;
        // color: #fff;
        // background: #2e333a;
        color: #666;
        background: #eceef0;
        box-sizing: border-box;
    }
}
</style>
