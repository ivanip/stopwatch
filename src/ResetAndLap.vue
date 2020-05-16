<template>
    <button id="reset-and-lap" @click="()=>{this.$emit('pressed')}">{{ (started && !stopped) ? 'Lap' : 'Reset' }}
    </button> <!-- [3] -->
</template>
<script>
    export default {
        name: 'ResetAndLap',
        props: ['started', 'stoped'],
    }
</script>
<style scoped>
    button#reset-and-lap {
        width: 80px;
        height: 80px;
        outline: none;
        background-color: #333;
        border: 2px solid #000;
        border-radius: 100%;
        color: #fff;
        font-size: 1rem;
        box-shadow: 0px 0px 0px 2px #333; /*X軸偏移，Y軸偏移，陰影模糊半徑，陰影擴散半徑、陰影顏色*/
        float: left;
    }
</style>