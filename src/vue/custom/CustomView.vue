<template>
    <div class="custom-view">
        <div v-if="!showError" class="birthday-container" :class="{ 'content-ready': showContent }">
            <transition name="pop-in">
                <h1 v-if="showTitle" class="happy-birthday-title">
                    <span class="wave-text">
                        <span v-for="(char, i) in happyArr" :key="'h'+i" :style="{ '--i': i, animationDelay: (i * 0.15) + 's' }" class="wave-char rainbow-char">{{ char }}</span>
                    </span>
                    <span class="wave-space"> </span>
                    <span class="wave-text">
                        <span v-for="(char, i) in birthdayArr" :key="'b'+i" :style="{ '--i': i + happyArr.length, animationDelay: ((i + happyArr.length) * 0.15) + 's' }" class="wave-char rainbow-char">{{ char }}</span>
                    </span>
                    <span class="birthday-name wave-text">
                        <span v-for="(char, i) in nameTextArr" :key="'n'+i" :style="{ '--i': i + happyArr.length + birthdayArr.length }" class="wave-char">{{ char }}</span>
                    </span>
                </h1>
            </transition>
            <transition name="pop-in">
                <div v-if="showSurprise" class="surprise-message">
                    <span class="sender-name">{{ senderName }}</span> tarafından sana bir sürpriz var,
                </div>
            </transition>
            <transition name="pop-in">
                <div v-if="showCake" class="cake-container">
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
            </transition>
            <transition name="pop-in">
                <div v-if="showBirthdayMsg" class="birthday-message">
                    <p>{{ birthdayMessage }}</p>
                </div>
            </transition>
            <transition name="pop-in">
                <div v-if="showWishHint && !blowCandle" class="congrats-message">
                    Bir dilek tut ve pastaya dokun!
                </div>
            </transition>
            <transition name="congrats-fade" @after-enter="onCongratsAfterEnter">
                <div v-if="blowCandle" class="congrats-message">
                    Tebrikler!
                </div>
            </transition>
            <div class="fireworks-container" ref="fireworksContainer"></div>
        </div>
        
        <div v-else class="error-container">
            <img src="/sad.gif" alt="Üzgün surat" class="sad-gif big-sad-gif" />
            <h2 class="error-title">Ooops! Görünüşe bakılırsa burada kimsenin doğum günü yok! olsun</h2>
        </div>
        <div v-if="isLoading" class="loading-overlay">
            <div class="curtains" :class="{ 'curtains-open': showCurtains }">
                <div class="curtain curtain-left"></div>
                <div class="curtain curtain-right"></div>
            </div>
            <div class="loading-content" :class="{ 'fade-out': hideLoadingText }">
                <h2 class="loading-text">Neredeyse Hazır!</h2>
                <div class="loading-dots">
                    <span></span>
                    <span></span>
                    <span></span>
                </div>
            </div>
        </div>
        <transition name="pop-in">
            <footer class="footer" v-if="showFooter && blowCandle">
                made by <a :href="baseUrl" target="_blank" rel="noopener">BehknQux</a>
            </footer>
        </transition>
    </div>
</template>

<script setup>
import { onMounted, ref, computed, watch } from 'vue';
import Fireworks from 'fireworks-js';

const fireworksContainer = ref(null);
const cakeHover = ref(false);
const blowCandle = ref(false);
const isLoading = ref(true);
const hideLoadingText = ref(false);
const showCurtains = ref(false);
const showContent = ref(false);
const showTitle = ref(false);
const showCake = ref(false);
const showSurprise = ref(false);
const showBirthdayMsg = ref(false);
const showWishHint = ref(false);
const showFooter = ref(false);
const showError = ref(false);

// API'den gelecek veriler için state'ler
const senderName = ref('');
const birthdayMessage = ref('');
const nameText = ref('');
const errorMessage = ref('');

const happyArr = 'HAPPY'.split('');
const birthdayArr = 'BIRTHDAY'.split('');
const nameTextArr = computed(() => nameText.value.split(''));

const rainbowColors = [
  '#FF5252', // red
  '#FFB300', // orange
  '#FFD600', // yellow
  '#69F0AE', // green
  '#40C4FF', // blue
  '#7C4DFF', // purple
  '#FF4081', // pink
  '#8D6E63', // brown
  '#00B8D4', // cyan
  '#C6FF00', // lime
  '#FF6D00', // deep orange
  '#00E676', // light green
  '#2979FF', // deep blue
  '#D500F9', // magenta
];

const getCandleStyle = (index, totalCandles) => {
    // Tek mum için ortada
    return {
        left: '50%',
        top: '-20px',
        marginLeft: '-8px',
    };
};

// URL'den ID'yi al
const getCardId = () => {
    const path = window.location.pathname;
    const match = path.match(/^\/sdc\/([^\/]+)$/);
    return match ? match[1] : null;
};

// API çağrısı
const fetchBirthdayData = async () => {
    try {
        const cardId = getCardId();
        if (!cardId) {
            throw new Error('Card ID not found in URL');
        }

        const response = await fetch(`https://ne41k5dtad.execute-api.eu-north-1.amazonaws.com/cards/${cardId}?type=birthday`);
        if (!response.ok) {
            throw new Error('Card not found');
        }
        const data = await response.json();
        
        return {
            senderName: data.sender,
            birthdayMessage: data.message,
            nameText: data.recipient
        };
    } catch (error) {
        console.error('API çağrısı sırasında hata oluştu:', error);
        if (error.message === 'Card not found') {
            errorMessage.value = 'Bu doğum günü kartı bulunamadı... 😢';
        } else {
            errorMessage.value = 'Bugün kimsenin doğum günü değil gibi görünüyor... 😢';
        }
        showError.value = true;
        isLoading.value = false;
        return null;
    }
};

function onCongratsAfterEnter() {
    setTimeout(() => {
        showFooter.value = true;
    }, 3000);
}

const handleContentVisible = () => {
    // Sıralı animasyonlar
    setTimeout(() => {
        showTitle.value = true;
        setTimeout(() => {
            showSurprise.value = true;
            setTimeout(() => {
                showCake.value = true;
                setTimeout(() => {
                    showBirthdayMsg.value = true;
                    setTimeout(() => {
                        showWishHint.value = true;
                    }, 3000);
                }, 3000);
            }, 3000);
        }, 3000);
    }, 100); // içerik geldikten hemen sonra başlat
};

const startLoadingSequence = async () => {
    try {
        const data = await fetchBirthdayData();
        if (!data) return; // Eğer fetchBirthdayData null dönerse (hata durumu) işlemi sonlandır

        senderName.value = data.senderName;
        birthdayMessage.value = data.birthdayMessage;
        nameText.value = data.nameText;

        // 1. Loading yazısını kaybet
        hideLoadingText.value = true;
        
        // 2. Perdeleri aç
        setTimeout(() => {
            showCurtains.value = true;
        }, 1000);

        // 3. İçeriği göster
        setTimeout(() => {
            showContent.value = true;
            isLoading.value = false;
            handleContentVisible(); // içerik görününce zamanlayıcı başlat
        }, 2000);

    } catch (error) {
        console.error('Veri yüklenirken hata oluştu:', error);
        errorMessage.value = 'Bir şeyler ters gitti... 😢';
        showError.value = true;
        isLoading.value = false;
    }
};

const baseUrl = computed(() => {
    const url = window.location.origin;
    return url;
});

onMounted(() => {
    startLoadingSequence();
    
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

watch(() => blowCandle.value, (newVal) => {
    if (newVal) {
        setTimeout(() => {
            showFooter.value = true;
        }, 3000);
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
    opacity: 0;
    transition: opacity 1s ease-in-out;
    
    &.content-ready {
        opacity: 1;
    }
}

.happy-birthday-title {
    font-size: 3.5rem;
    margin-bottom: 1.5rem;
    color: #d84315;
    font-family: 'Pacifico', cursive, sans-serif;
    letter-spacing: 2px;
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

.rainbow-char {
    animation: rainbow-hue 2.5s steps(1) infinite;
}

@keyframes rainbow-hue {
    0% {
        color: #FF5252;
    }
    14% {
        color: #FFB300;
    }
    28% {
        color: #FFD600;
    }
    42% {
        color: #69F0AE;
    }
    56% {
        color: #40C4FF;
    }
    70% {
        color: #7C4DFF;
    }
    84% {
        color: #FF4081;
    }
    100% {
        color: #FF5252;
    }
}

.birthday-name .wave-char {
    color: #ff4da6;
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

.loading-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    z-index: 1000;
    pointer-events: none;
}

.curtains {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    z-index: 1;
    
    .curtain {
        transition: transform 1s cubic-bezier(0.4, 0, 0.2, 1);
    }
    
    &.curtains-open {
        .curtain-left {
            transform: scaleX(0);
        }
        .curtain-right {
            transform: scaleX(0);
        }
    }
}

.curtain {
    position: absolute;
    top: 0;
    width: 50%;
    height: 100%;
    background: linear-gradient(45deg, #d84315, #ff4da6);
    box-shadow: 0 0 20px rgba(0, 0, 0, 0.5);
}

.curtain-left {
    left: 0;
    transform-origin: left;
}

.curtain-right {
    right: 0;
    transform-origin: right;
}

.loading-content {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    z-index: 2;
    text-align: center;
    color: white;
    text-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
    transition: opacity 0.5s ease-in-out;
    
    &.fade-out {
        opacity: 0;
    }
}

.loading-text {
    font-size: 3rem;
    font-family: 'Pacifico', cursive;
    margin-bottom: 2rem;
    animation: pulse 1.5s infinite;
}

.loading-dots {
    display: flex;
    justify-content: center;
    gap: 0.5rem;
    
    span {
        width: 15px;
        height: 15px;
        background: white;
        border-radius: 50%;
        animation: bounce 0.5s infinite alternate;
        
        &:nth-child(2) {
            animation-delay: 0.2s;
        }
        
        &:nth-child(3) {
            animation-delay: 0.4s;
        }
    }
}

@keyframes pulse {
    0% {
        transform: scale(1);
        opacity: 1;
    }
    50% {
        transform: scale(1.05);
        opacity: 0.8;
    }
    100% {
        transform: scale(1);
        opacity: 1;
    }
}

@keyframes bounce {
    from {
        transform: translateY(0);
    }
    to {
        transform: translateY(-20px);
    }
}

@media (max-width: 600px) {
  .happy-birthday-title {
    font-size: 3rem;
  }
  .surprise-message {
    font-size: 1.1rem;
  }
  .birthday-message {
    font-size: 1rem;
  }
}

.congrats-message {
    text-align: center;
    margin-top: 2rem;
    font-size: 1.5rem;
    color: #d84315;
    font-family: 'Pacifico', cursive;
    font-weight: bold;
    text-shadow: 0 0 10px #fff, 0 0 20px #ff4da6;
    animation: pop-in 0.7s cubic-bezier(0.4, 2, 0.6, 1);
}

@keyframes pop-in {
    0% {
        transform: scale(0.5);
        opacity: 0;
    }
    80% {
        transform: scale(1.1);
        opacity: 1;
    }
    100% {
        transform: scale(1);
        opacity: 1;
    }
}

.pop-in-enter-active {
    animation: pop-in 0.7s cubic-bezier(0.4, 2, 0.6, 1);
}
.pop-in-leave-active {
    transition: opacity 0.7s;
}
.pop-in-enter-from, .pop-in-leave-to {
    opacity: 0;
    transform: scale(0.5);
}
.pop-in-enter-to, .pop-in-leave-from {
    opacity: 1;
    transform: scale(1);
}

.congrats-fade-enter-active, .congrats-fade-leave-active {
    transition: opacity 0.7s;
}
.congrats-fade-enter-from, .congrats-fade-leave-to {
    opacity: 0;
}
.congrats-fade-enter-to, .congrats-fade-leave-from {
    opacity: 1;
}

.fade-enter-active, .fade-leave-active {
    transition: opacity 0.7s;
}
.fade-enter-from, .fade-leave-to {
    opacity: 0;
}
.fade-enter-to, .fade-leave-from {
    opacity: 1;
}

.footer {
    min-height: 2.2em;
    transition: min-height 0.3s;
}

@keyframes pop-in-fast {
    0% {
        transform: scale(0.5);
        opacity: 0;
    }
    80% {
        transform: scale(1.1);
        opacity: 1;
    }
    100% {
        transform: scale(1);
        opacity: 1;
    }
}

.pop-in-fast-enter-active {
    animation: pop-in-fast 0.5s cubic-bezier(0.4, 2, 0.6, 1);
}
.pop-in-fast-leave-active {
    transition: opacity 0.2s, transform 0.2s;
}
.pop-in-fast-enter-from, .pop-in-fast-leave-to {
    opacity: 0;
    transform: scale(0.5);
}
.pop-in-fast-enter-to, .pop-in-fast-leave-from {
    opacity: 1;
    transform: scale(1);
}

.wish-hint {
    text-align: center;
    margin-top: 1.2rem;
    font-size: 1.5rem;
    color: #d84315;
    font-family: 'Pacifico', cursive;
    font-weight: bold;
    letter-spacing: 1px;
    text-shadow: 0 0 10px #fff, 0 0 20px #ff4da6;
    animation: pulse 1.5s infinite;
    min-height: 1.7em;
    transition: min-height 0.3s;
}

.error-container {
    text-align: center;
    padding: 2rem;
    animation: fadeIn 0.5s ease-in;
}

.sad-gif {
    width: 120px;
    height: 120px;
    margin-bottom: 1rem;
    display: inline-block;
    animation: bounce 2s infinite;
}

.big-sad-gif {
    width: 200px;
    height: 200px;
}

.error-title {
    font-size: 2.5rem;
    color: #d84315;
    font-family: 'Pacifico', cursive;
    margin-bottom: 1rem;
}

.error-message {
    font-size: 1.5rem;
    color: #2a2a2a;
    margin-bottom: 1rem;
}

.error-submessage {
    font-size: 1.2rem;
    color: #666;
    font-style: italic;
}

@keyframes fadeIn {
    from {
        opacity: 0;
        transform: translateY(20px);
    }
    to {
        opacity: 1;
        transform: translateY(0);
    }
}

@keyframes bounce {
    0%, 100% {
        transform: translateY(0);
    }
    50% {
        transform: translateY(-20px);
    }
}
</style> 