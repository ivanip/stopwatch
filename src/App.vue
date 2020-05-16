<template>
    <div id="app">
        <timer :time="displayTime"/>
        <div id="stopwatch-controls">
            <ResetAndLap
                    @pressed="resetAndLapHandler"
                    :started="isStarted"
                    :stoped="isStopped"
            />
            <StartAndStopButton
                    @pressed="startAndStopHandler"
                    :started="isStarted"
                    :stoped="isStopped"
            />
            <Bland>My Awesome Timer</Bland>
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
    import Timer from './Timer.vue';
    import Bland from "@/Brand";   //@/ 代表Root directory
    import StartAndStopButton from "@/StartAndStopButton";
    import ResetAndLap from "@/ResetAndLap";

    export default {
        name: 'app',
        components: {ResetAndLap, StartAndStopButton, Bland, Timer,},
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
</style>