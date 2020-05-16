<template>
    <div id="stopwatch">
        {{ time }}
    </div>

</template>
<script>
    export default {
        name: 'Timer',
        props:['time'],
    }
</script>
<style>
    #stopwatch {
        flex: 0px 1 1;
        display: flex;
        justify-content: center;
        align-items: center;
        font-size: 5rem;
    }
</style>