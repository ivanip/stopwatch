<template>
    <div id="app">
        <div id="stopwatch">
            {{ displayTime }}
        </div>
        <div id="stopwatch-controls">
            <button id="reset-and-lap" @click="resetAndLapHandler">{{ (isStarted && !isStopped) ? 'Lap' : 'Reset' }}</button> <!-- [3] -->
            <button id="start-and-stop" @click="startAndStopHandler" :class="{ red: isStarted && !isStopped }">{{ (isStarted && !isStopped) ? 'Stop' : 'Start' }}</button>
            <span id="brand">Stopwatch</span>
        </div>
        <ul id="stopwatch-records">
            <li :key="index" v-for="(lap, index) in laps"> <!-- [4] -->
                <span>Lap {{ index + 1 }}</span>
                <span>{{ lap.display }}</span>
            </li>
        </ul>
    </div>
</template>

<script>
    export default {
        name: 'app',
        components: {},
        data() {
            return {
                timer: null,
                displayTime: '00:00.0',
                startTime: null,
                stopTime: null,
                stoppedTimeOffset: 0,
                currentTime: null,
                isStarted: false,
                isStopped: false,
                lastLapTime: null,  // [1]
                laps: [],  // [2]
            }
        },
        methods: {
            startAndStopHandler() {  // [4]

                if (!this.isStarted) {
                    this.startTime = Date.now()
                    this.lastLapTime=Date.now()

                    this.timer = setInterval(() => {
                        this.currentTime = Date.now();
                        let elapsedTime = this.currentTime - this.startTime
                        this.displayTime = this.formatTime((elapsedTime / 1000).toFixed(2))
                    }, 10);

                    this.isStarted = true
                    this.isStopped = false
                } else if (this.isStarted && !this.isStopped) {
                    clearInterval(this.timer)
                    this.stopTime = Date.now()
                    this.isStopped = true
                } else if (this.isStarted && this.isStopped) {
                    this.stoppedTimeOffset += Date.now() - this.stopTime

                    this.timer = setInterval(() => {
                        this.currentTime = Date.now();
                        let elapsedTime = this.currentTime - this.startTime - this.stoppedTimeOffset  // [6]
                        this.displayTime = this.formatTime((elapsedTime / 1000).toFixed(2))
                    }, 10);

                    this.isStopped = false
                }
            },
            resetAndLapHandler() {
                if (this.isStarted && this.isStopped) {
                    // Reset
                    this.startTime = null
                    this.stopTime = null
                    this.stoppedTimeOffset = null
                    this.displayTime = '00:00.00'
                    clearInterval(this.timer)
                    this.isStarted = false
                    this.isStopped = false
                } else if (this.isStarted && !this.isStopped) {
                    // Lap
                    let elapsedTime = this.currentTime - this.lastLapTime
                    let lapTime = (elapsedTime / 1000).toFixed(2)

                    this.laps.push({  // [4]
                        time: parseFloat(lapTime),
                        display: this.formatTime(lapTime),
                    })

                    this.lastLapTime = this.currentTime  // [5]
                }
            },
            formatTime(seconds) {
                let date = new Date(null);
                date.setSeconds(seconds);
                let result = date.toISOString().substr(14, 5);

                return `${result}.${(seconds + '').split('.')[1]}`
            },
        },
    }
</script>

<style>
    #app {
        font-family: 'Avenir', Helvetica, Arial, sans-serif;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        background-color: #000;
        width: 100%;
        height: 100%;
        display: flex;
        flex-direction: column;
        color: #fff;
    }

    #stopwatch {
        flex: 0px 1 1;
        display: flex;
        justify-content: center;
        align-items: center;
        font-size: 5rem;
    }

    #stopwatch-records {
        flex: 0px 1 1;
    }

    #stopwatch-records {
        list-style: none;
        padding: 0;
        margin: 0 1.2rem;
        overflow: auto;
    }

    #stopwatch-records li {
        border-bottom: 1px solid #323234;
        display: flex;
        padding: .8rem 0;
    }

    #stopwatch-records li.green {
        color: #2ED158;
    }

    #stopwatch-records li.red {
        color: #FF453A;
    }

    #stopwatch-records li:first-child {
        border-top: 1px solid #323234;
    }

    #stopwatch-records span {
        display: block;
        flex: 0px 1 1;
    }

    #stopwatch-records span:last-child {
        text-align: right;
    }

    #stopwatch-controls {
        flex: 80px 0 0;
        padding: .2rem 1.2rem;
        overflow: hidden;
        position: relative;
        margin-bottom: 1.5rem;
    }

    #brand {
        position: absolute;
        left: 50%;
        top: 50%;
        transform: translateX(-50%) translateY(-50%);
    }

    button#reset-and-lap, button#start-and-stop {
        width: 80px;
        height: 80px;
        outline: none;
        background-color: #333;
        border: 2px solid #000;
        border-radius: 100%;
        color: #fff;
        font-size: 1rem;
        box-shadow: 0px 0px 0px 2px #333;
        float: left;
    }

    button#start-and-stop {
        background-color: #082A12;
        box-shadow: 0px 0px 0px 2px #082A12;
        color: #2ED158;
        float: right;
    }

    button#start-and-stop.red {
        background-color: #320E0B;
        box-shadow: 0px 0px 0px 2px #320E0B;
        color: #FF453A;
    }
</style>