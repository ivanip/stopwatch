<template><span id="brand"><slot/></span></template>

<script>
    export default {
        name: "Bland"
    }
</script>
<style scoped>
    /*
   Tip X:
   這個是常用的一個置中方法，使用絕對位置後，元素便脫離正常文檔流，
   left和top分別是元素左外邊距邊界和上外邊跟邊界
   transform是元素中心在X和Y軸傍移，它以元素自身的空間為座標
   */
    #brand {
        position: absolute;
        left: 50%;
        top: 50%;
        transform: translateX(-50%) translateY(-50%);
    }
</style>
