<template>
    <div>
        <h1>Simon says</h1>
        <div class="main">
            <div class="playground">
                <div @click="handlePress(0)" class="blue" :class="isActive === 0 ? 'active' : ''"></div>
                <div @click="handlePress(1)" class="red" :class="isActive === 1 ? 'active' : ''"></div>
                <div @click="handlePress(2)" class="yellow" :class="isActive === 2 ? 'active' : ''"></div>
                <div @click="handlePress(3)" class="green" :class="isActive === 3 ? 'active' : ''"></div>
            </div>
            <div class="options">
                <h2>Раунд: {{ round }}</h2>
                <p>Вы сейчас на сложности: {{ difficulty }}</p>
                <button :disabled="pending" @click="handleStart">Старт</button>
                <ul>
                    <li>
                        <button :disabled="pending" @click="changeDifficulty('Легкий', 1.5)">Легкий</button>
                    </li>
                    <li>
                        <button :disabled="pending" @click="changeDifficulty('Нормальный', 1)">Нормальный</button>
                    </li>
                    <li>
                        <button :disabled="pending" @click="changeDifficulty('Сложный', 0.4)">Сложный</button>
                    </li>
                </ul>
            </div>
        </div>
        <audio ref="audioPlayer" :src="activeAudio"></audio>
    </div>
</template>

<script>
import first from './assets/sounds/1.mp3'
import second from './assets/sounds/2.mp3'
import third from './assets/sounds/3.mp3'
import fourth from './assets/sounds/4.mp3'

export default {
    data() {
        return {
            difficulty: 'Легкий',
            delay: 1.5,
            isActive: -1,
            round: 0,
            combination: [],
            accumulator: '',
            step: 0,
            index: 0,
            pending: false,
            activeAudio: first,
            audioList: [first, second, third, fourth],
        }
    },
    methods: {
        async animate() {
            this.pending = true
            this.step = 0

            function delay(ms) {
                return new Promise(resolve => setTimeout(resolve, ms))
            }

            for (let i = 0; i < this.combination.length; i++) {
                if (isNaN(this.combination[this.step])) {
                    this.pending = false
                    return
                } else {
                    await delay((this.delay / 2) * 1000)
                    this.isActive = this.combination[this.step]
                    this.activeAudio = this.audioList[this.isActive]
                    this.$refs.audioPlayer.load()
                    this.$refs.audioPlayer.addEventListener('loadeddata', () => {
                        this.$refs.audioPlayer.play()
                    })
                    await delay((this.delay / 2) * 1000)
                    this.step++
                }
                this.isActive = -1
            }

            this.pending = false;
        },
        changeDifficulty(str, num) {
            this.difficulty = str
            this.delay = num
        },
        async handleStart() {
            this.combination = []
            this.accumulator = ''
            this.isActive = -1
            this.round = 0
            this.combination.push(Math.floor(Math.random() * 4))
            await this.animate()
            this.round++
        },
        handlePress(num) {
            if (!this.pending) {
                this.accumulator += num

                if (this.combination.join('').includes(this.accumulator)) {
                    if (this.combination.length === this.accumulator.length) {
                        this.combination.push(Math.floor(Math.random() * 4))
                        this.round++
                        this.accumulator = ''
                        this.animate()
                    }
                } else {
                    this.combination = []
                    this.accumulator = ''
                    this.isActive = -1
                    this.round = 0
                }
            }

            this.activeAudio = this.audioList[num]
            this.$refs.audioPlayer.load()
            this.$refs.audioPlayer.addEventListener('loadeddata', () => {
                this.$refs.audioPlayer.play()
            })
        },
    },
}
</script>

<style>
body {
    font-family: sans-serif;
}
h1 {
    text-align: center;
}
.playground {
    display: flex;
    width: 200px;
    height: 200px;
    flex-wrap: wrap;
}

.options {
    max-width: 200px;
}

.main {
    display: flex;
    justify-content: center;
    flex-wrap: wrap;
    gap: 30px;
}

.playground div {
    height: 100px;
    width: 100px;
    opacity: 0.5;
}

.playground div:active {
    opacity: 1;
}

.red {
    background: red;
}
.blue {
    background: blue;
}
.yellow {
    background: yellow;
}
.green {
    background: green;
}

.red.active,
.blue.active,
.yellow.active,
.green.active {
    opacity: 1;
}
</style>
