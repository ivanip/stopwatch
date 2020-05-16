<template>
    <div id="app">
        <div id="stopwatch">
            {{ displayTime }}
        </div>
        <div id="stopwatch-controls">
            <button id="reset-and-lap" @click="resetAndLapHandler">{{ (isStarted && !isStopped) ? 'Lap' : 'Reset' }}
            </button> <!-- [3] -->
            <button id="start-and-stop" @click="startAndStopHandler" :class="{ red: isStarted && !isStopped }">{{
                (isStarted && !isStopped) ? 'Stop' : 'Start' }}
            </button>
            <span id="brand">Stopwatch</span>
        </div>
        <ul id="stopwatch-records">
            <li :class="{ red: lap.isSlowest, green: lap.isFastest }" :key="index" v-for="(lap, index) in lapsRecords">
                <!-- [2] -->
                <span>Lap {{ lapsRecords.length - index }}</span>
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
                stoppedTimeOffsetForLap: 0,  // [1]
                currentTime: null,
                isStarted: false,
                isStopped: false,
                lastLapTime: null,
                laps: [],
            }
        },
        computed: {
            lapsRecords() {  // [1]
                let lapsClone = [...this.laps]

                if (lapsClone.length > 2) {
                    lapsClone = lapsClone.map((oneLap) => {
                        return {
                            ...oneLap,
                            isFastest: false,
                            isSlowest: false,
                        }
                    })

                    let fastestIndex = 0
                    let slowestIndex = 0

                    for (let i = 0; i < lapsClone.length - 1; i++) {
                        if (lapsClone[i].time < lapsClone[fastestIndex].time) {
                            fastestIndex = i;
                        }
                        if (lapsClone[i].time > lapsClone[slowestIndex].time) {
                            slowestIndex = i;
                        }
                    }

                    lapsClone[fastestIndex].isFastest = true
                    lapsClone[slowestIndex].isSlowest = true
                }

                return lapsClone.reverse()
            }
        },
        watch: {  // [2]
            displayTime() {
                let lapsClone = [...this.laps];

                let elapsedTime = this.currentTime - this.lastLapTime - this.stoppedTimeOffsetForLap
                let lapTime = (elapsedTime / 1000).toFixed(2)

                lapsClone[this.laps.length - 1] = {
                    time: parseFloat(lapTime),
                    display: this.formatTime(lapTime),
                    isFastest: false,
                    isSlowest: false,
                };

                this.laps = lapsClone;
            }
        },
        methods: {
            startAndStopHandler() {  // [4]
                if (!this.isStarted) {
                    this.startTime = Date.now()
                    this.lastLapTime = Date.now()

                    this.timer = setInterval(() => {
                        this.currentTime = Date.now();
                        let elapsedTime = this.currentTime - this.startTime
                        this.displayTime = this.formatTime((elapsedTime / 1000).toFixed(2))
                    }, 10);
                    this.laps = [{   // [1]
                        time: 0,
                        display: this.displayTime,
                        isFastest: false,
                        isSlowest: false,
                    }];
                    this.isStarted = true
                    this.isStopped = false
                } else if (this.isStarted && !this.isStopped) {
                    clearInterval(this.timer)
                    this.stopTime = Date.now()
                    this.isStopped = true
                } else if (this.isStarted && this.isStopped) {
                    this.stoppedTimeOffset += Date.now() - this.stopTime
                    this.stoppedTimeOffsetForLap += Date.now() - this.stopTime
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
                    this.laps = []
                } else if (this.isStarted && !this.isStopped) {
                    // Lap
                    let elapsedTime = this.currentTime - this.lastLapTime - this.stoppedTimeOffsetForLap
                    let lapTime = (elapsedTime / 1000).toFixed(2)

                    this.laps.push({  // [4]
                        time: parseFloat(lapTime),
                        display: this.formatTime(lapTime),
                    })

                    this.lastLapTime = this.currentTime  // [5]
                    this.stoppedTimeOffsetForLap = 0
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

    /*todo: 嘗試將stopwatch-controls改為Flex佈局*/
    #stopwatch-controls {
        flex: 80px 0 0;
        padding: .2rem 1.2rem;
        overflow: hidden;
        position: relative; /*遵循正常文檔流，依top,right,bottom,left等屬性進行絕對定位*/
        margin-bottom: 1.5rem;
    }

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

    button#reset-and-lap, button#start-and-stop {
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