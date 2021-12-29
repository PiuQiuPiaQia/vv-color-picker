<template>
    <div id="app">
        <img alt="Vue logo" src="./assets/logo.png" />
        <HelloWorld msg="Welcome to Your Vue.js App" />
        <div class="here">
            <b :style="{ color: color }">搁这儿呢！！！</b>
            <ColorPicker v-model="color" :selector="selector"></ColorPicker>
        </div>

        <div class="box" :style="{ backgroundColor: color }"></div>
    </div>
</template>

<script>
import HelloWorld from "./components/HelloWorld.vue";
import ColorPicker from "./components/ColorPicker/ColorPicker.vue";

export default {
    name: "App",
    components: {
        HelloWorld,
        ColorPicker,
    },
    data() {
        return {
            selector: "body",
            color: "red",
        };
    },
};
</script>

<style>
#app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
}
.here {
    display: flex;
}
.box {
    height: 200px;
}
</style>
