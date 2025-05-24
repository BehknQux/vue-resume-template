<template>
    <div class="custom-view">
        <div class="birthday-container">
            <h1 class="happy-birthday-title">
                <span class="wave-text">
                    <span v-for="(char, i) in happyArr" :key="'h'+i" :style="{ '--i': i }" class="wave-char">{{ char }}</span>
                </span>
                <span class="wave-space"> </span>
                <span class="wave-text">
                    <span v-for="(char, i) in birthdayArr" :key="'b'+i" :style="{ '--i': i + happyArr.length }" class="wave-char">{{ char }}</span>
                </span>
                <span class="birthday-name wave-text">
                    <span v-for="(char, i) in nameTextArr" :key="'n'+i" :style="{ '--i': i + happyArr.length + birthdayArr.length }" class="wave-char">{{ char }}</span>
                </span>
            </h1>
            <div class="surprise-message">
                <span class="sender-name">Ahmet</span> tarafından sana bir sürpriz var,
            </div>
            <div class="birthday-message">
                <p>Bugün senin günün! Nice mutlu, sağlıklı, neşeli yaşlara! 🎈</p>
            </div>
            <div class="cake-container">
                <div class="cake" @mouseover="cakeHover = true" @mouseleave="cakeHover = false" @click="blowCandle = true" :class="{ 'cake-hover': cakeHover }">
                    <div class="plate"></div>
                    <div class="layer layer-bottom"></div>
                    <div class="layer layer-middle"></div>
                    <div class="layer layer-top"></div>
                    <div class="icing"></div>
                    <div class="drip drip1"></div>
                    <div class="drip drip2"></div>
                    <div class="drip drip3"></div>
                    <div class="candle" :style="getCandleStyle(1, 1)">
                        <div v-if="!blowCandle" class="flame"></div>
                    </div>
                </div>
            </div>
            <div class="fireworks-container" ref="fireworksContainer"></div>
        </div>
        <footer class="footer">
            made by <a href="https://github.com/BehknQux" target="_blank" rel="noopener">BehknQux</a>
        </footer>
    </div>
</template>

<script setup>
import { onMounted, ref } from 'vue';
import Fireworks from 'fireworks-js';

const fireworksContainer = ref(null);
const cakeHover = ref(false);
const blowCandle = ref(false);

const happyArr = 'HAPPY'.split('');
const birthdayArr = 'BIRTHDAY'.split('');
const nameText = 'Hamza';
const nameTextArr = nameText.split('');

const getCandleStyle = (index, totalCandles) => {
    // Tek mum için ortada
    return {
        left: '50%',
        top: '-20px',
        marginLeft: '-8px',
    };
};

onMounted(() => {
    if (fireworksContainer.value) {
        const width = window.innerWidth;
        const height = window.innerHeight;
        const fireworks = new Fireworks(fireworksContainer.value, {
            speed: 4,
            acceleration: 1.05,
            friction: 0.97,
            gravity: 1.5,
            particles: 140,
            explosion: 8,
            autoresize: true,
            delay: { min: 100, max: 300 },
            boundaries: {
                x: 0,
                y: height * 0.7,
                width: width,
                height: height * 0.3
            },
            hue: { min: 0, max: 360 },
            brightness: {
                min: 50,
                max: 80
            },
            decay: {
                min: 0.015,
                max: 0.03
            }
        });
        fireworks.start();
    }
});
</script>

<style lang="scss" scoped>
.custom-view {
    padding: 2rem;
    color: #2a2a2a;
    background: linear-gradient(120deg, #ffe082, #ffd6e0, #b2f7ef, #ffe082);
    background-size: 400% 400%;
    animation: bg-gradient-move 18s ease-in-out infinite;
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    position: relative;
}

.birthday-container {
    text-align: center;
    z-index: 1;
}

.happy-birthday-title {
    font-size: 3.5rem;
    margin-bottom: 1.5rem;
    color: #d84315;
    text-shadow: 0 0 20px #fff3e0, 0 0 40px #fff3e0, 0 0 60px #fff;
    font-family: 'Pacifico', cursive, sans-serif;
    letter-spacing: 2px;
    animation: glow 2s ease-in-out infinite alternate;
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 1rem;
    flex-wrap: wrap;
    word-break: keep-all;
    white-space: pre-line;
}

.wave-text {
    display: inline-block;
}

.wave-char {
    display: inline-block;
    animation: wave 1.6s infinite ease-in-out;
    animation-delay: calc(var(--i) * 0.08s);
}

.birthday-name .wave-char {
    color: #ff4da6;
    text-shadow: 0 0 10px #fff, 0 0 20px #ff4da6;
}

@keyframes wave {
    0%, 100% { transform: translateY(0); }
    20% { transform: translateY(-10px); }
    40%, 80% { transform: translateY(0); }
}

.surprise-message {
    font-size: 1.5rem;
    margin-bottom: 1.2rem;
    color: #2a2a2a;
    font-family: 'Lato', sans-serif;
    font-style: italic;
    letter-spacing: 1px;
    background: none;
    border-radius: 0;
    padding: 0;
    display: inline;
    box-shadow: none;
}

.sender-name {
    color: #ff4da6;
    font-weight: bold;
    font-size: 1.1em;
    text-shadow: 0 0 10px #fff59d;
}

.birthday-message {
    font-size: 1.3rem;
    line-height: 1.6;
    margin-bottom: 2rem;
    color: #2a2a2a;
}

.footer {
    position: fixed;
    bottom: 10px;
    left: 0;
    width: 100vw;
    text-align: center;
    color: #2a2a2a;
    font-size: 1rem;
    z-index: 2;
    a {
        color: #d84315;
        text-decoration: underline;
        font-weight: bold;
    }
}

$vanilla: #f0e4d0;
$chocolate: #553c13;

@mixin foodColoring($color) {
    background-color: $color;
    box-shadow:
        0 2px 0px lighten($color, 5%),
        0 4px 0px darken($color, 8.2%),
        0 6px 0px darken($color, 8.4%),
        0 8px 0px darken($color, 8.6%),
        0 10px 0px darken($color, 8.8%),
        0 12px 0px darken($color, 9%),
        0 14px 0px darken($color, 9.2%),
        0 16px 0px darken($color, 9.4%),
        0 18px 0px darken($color, 9.6%),
        0 20px 0px darken($color, 9.8%),
        0 22px 0px darken($color, 10%),
        0 24px 0px darken($color, 10.2%),
        0 26px 0px darken($color, 10.4%),
        0 28px 0px darken($color, 10.6%),
        0 30px 0px darken($color, 10.8%);
}

.cake-container {
    position: relative;
    width: 300px;
    height: 300px;
    margin: 2rem auto;
}

.cake {
    position: absolute;
    width: 250px;
    height: 200px;
    top: 50%;
    left: 50%;
    margin-top: -70px;
    margin-left: -125px;
    transition: transform 0.3s cubic-bezier(0.4, 2, 0.6, 1);
}
.cake-hover {
    transform: scale(1.08);
}

.plate {
    width: 270px;
    height: 110px;
    position: absolute;
    bottom: -10px;
    left: -10px;
    background-color: #ccc;
    border-radius: 50%;
    box-shadow:
        0 2px 0 darken(#ccc, 10%),
        0 4px 0 darken(#ccc, 10%),
        0 5px 40px rgba(black, 0.5);
}

.cake > * {
    position: absolute;
}

.layer {
    position: absolute;
    display: block;
    width: 250px;
    height: 100px;
    border-radius: 50%;
    @include foodColoring($chocolate);
}

.layer-top { top: 0px; }
.layer-middle { top: 33px; }
.layer-bottom { top: 66px; }

.icing {
    top: 2px;
    left: 5px;
    background-color: $vanilla;
    width: 240px;
    height: 90px;
    border-radius: 50%;
    &:before {
        content: "";
        position: absolute;
        top: 4px;
        right: 5px;
        bottom: 6px;
        left: 5px;
        background-color: lighten($vanilla, 3%);
        box-shadow:
            0 0 4px lighten($vanilla, 5%),
            0 0 4px lighten($vanilla, 5%),
            0 0 4px lighten($vanilla, 5%);
        border-radius: 50%;
        z-index: 1;
    }
}

.drip {
    display: block;
    width: 50px;
    height: 60px;
    border-bottom-left-radius: 25px;
    border-bottom-right-radius: 25px;
    background-color: $vanilla;
}

.drip1 {
    top: 53px;
    left: 5px;
    transform: skewY(15deg);
    height: 48px;
    width: 40px;
}

.drip2 {
    top: 69px;
    left: 181px;
    transform: skewY(-15deg);
}

.drip3 {
    top: 54px;
    left: 90px;
    width: 80px;
    border-bottom-left-radius: 40px;
    border-bottom-right-radius: 40px;
}

.candle {
    background-color: #7B020B;
    width: 16px;
    height: 50px;
    border-radius: 8px / 4px;
    top: -20px;
    left: 50%;
    margin-left: -8px;
    z-index: 10;
    &:before {
        content: "";
        position: absolute;
        top: 0;
        left: 0;
        width: 16px;
        height: 8px;
        border-radius: 50%;
        background-color: lighten(#7B020B, 10%);
    }
}

.flame {
    position: absolute;
    background-color: orange;
    width: 15px;
    height: 35px;
    border-radius: 10px 10px 10px 10px / 25px 25px 10px 10px;
    top: -34px;
    left: 50%;
    margin-left: -7.5px;
    z-index: 10;
    box-shadow:
        0 0 10px rgba(orange, 0.5),
        0 0 20px rgba(orange, 0.5),
        0 0 60px rgba(orange, 0.5),
        0 0 80px rgba(orange, 0.5);
    transform-origin: 50% 90%;
    animation: flicker 1s ease-in-out alternate infinite;
}

@keyframes flicker {
    0% {
        transform: skewX(5deg);
        box-shadow: 
            0 0 10px rgba(orange, 0.2),
            0 0 20px rgba(orange, 0.2),
            0 0 60px rgba(orange, 0.2),
            0 0 80px rgba(orange, 0.2) }
    25% {
        transform: skewX(-5deg);
        box-shadow:
            0 0 10px rgba(orange, 0.5),
            0 0 20px rgba(orange, 0.5),
            0 0 60px rgba(orange, 0.5),
            0 0 80px rgba(orange, 0.5) }
    50% {
        transform: skewX(10deg);
        box-shadow:
            0 0 10px rgba(orange, 0.3),
            0 0 20px rgba(orange, 0.3),
            0 0 60px rgba(orange, 0.3),
            0 0 80px rgba(orange, 0.3) }
    75% {
        transform: skewX(-10deg);
        box-shadow:
            0 0 10px rgba(orange, 0.4),
            0 0 20px rgba(orange, 0.4),
            0 0 60px rgba(orange, 0.4),
            0 0 80px rgba(orange, 0.4) }
    100% {
        transform: skewX(5deg);
        box-shadow:
            0 0 10px rgba(orange, 0.5),
            0 0 20px rgba(orange, 0.5),
            0 0 60px rgba(orange, 0.5),
            0 0 80px rgba(orange, 0.5) }
}

.fireworks-container {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    pointer-events: none;
    z-index: -1;
    filter: blur(2px);
}

@import url('https://fonts.googleapis.com/css2?family=Pacifico&display=swap');
@import url('https://fonts.googleapis.com/css?family=Lato:300italic');

@keyframes glow {
    from {
        text-shadow: 0 0 10px #fff, 0 0 20px #fff, 0 0 30px #e60073;
    }
    to {
        text-shadow: 0 0 20px #fff, 0 0 30px #ff4da6, 0 0 40px #ff4da6;
    }
}

@keyframes bg-gradient-move {
    0% {
        background-position: 0% 50%;
    }
    50% {
        background-position: 100% 50%;
    }
    100% {
        background-position: 0% 50%;
    }
}

.wave-space {
    display: inline-block;
    min-width: 0.15em;
}
</style> 