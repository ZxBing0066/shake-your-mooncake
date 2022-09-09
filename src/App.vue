<script setup lang="ts">
import { reactive, ref } from 'vue';

import moonCakes from './moonCakes';

const isBind = ref(false);
const isShaking = ref(false);
const showMoonCake = ref<null | number>(null);

const l = moonCakes.length;

const randomMoonCakeIndex = () => {
    const t = (l * Math.random()) | 0;
    return t;
};
const targetRef = ref(randomMoonCakeIndex());

const logs = reactive<any[]>([]);

const closeDialog = () => {
    showMoonCake.value = null;
    targetRef.value = randomMoonCakeIndex();
};

const bindShake = () => {
    if (isBind.value) return;
    isBind.value = true;
    // let t: ReturnType<typeof setTimeout> | null = null;
    let latestDevicemotion: any = null;
    // 摇动的事件/次数
    let shakeCount = 0;

    const bindEvent = () => {
        window.addEventListener('devicemotion', e => {
            console.log(e);
            if (showMoonCake.value !== null) return;
            if (!e.accelerationIncludingGravity || !e.acceleration) return;

            const target = targetRef.value;

            const devicemotion = {
                accelerationIncludingGravityX: e.accelerationIncludingGravity.x!,
                accelerationIncludingGravityY: e.accelerationIncludingGravity.y!,
                accelerationIncludingGravityZ: e.accelerationIncludingGravity.z!,
                accelerationX: e.acceleration.x,
                accelerationY: e.acceleration.y,
                accelerationZ: e.acceleration.z
            };

            if (latestDevicemotion) {
                // 摇一摇的幅度
                const thresholdCount = Math.max(
                    Math.abs(
                        latestDevicemotion.accelerationIncludingGravityX - devicemotion.accelerationIncludingGravityX
                    ),
                    Math.abs(
                        latestDevicemotion.accelerationIncludingGravityY - devicemotion.accelerationIncludingGravityY
                    ),
                    Math.abs(
                        latestDevicemotion.accelerationIncludingGravityZ - devicemotion.accelerationIncludingGravityZ
                    )
                );

                // 大于一定幅度才计算
                if (thresholdCount > 20) {
                    shakeCount += 8;
                }

                logs.push({
                    thresholdCount,
                    target,
                    l,
                    shakeCount
                });

                if (thresholdCount > 30 + (target / l) * 60 && shakeCount >= 100 + (target / l) * 150) {
                    window.navigator.vibrate?.(200);
                    showMoonCake.value = target;
                    shakeCount = 0;
                    const audio = document.createElement('audio');
                    audio.src = '/success.mp3';
                    document.body.appendChild(audio);
                    audio.onended = () => document.body.removeChild(audio);
                    audio.play();
                }
                shakeCount = Math.max(0, shakeCount - 2);
            }

            latestDevicemotion = devicemotion;
        });
    };

    if (typeof (DeviceMotionEvent as any).requestPermission === 'function') {
        (DeviceMotionEvent as any)
            .requestPermission()
            .then((permissionState: string) => {
                if (permissionState === 'granted') {
                    bindEvent();
                }
            })
            .catch(() => {
                alert('获取权限失败');
            });
    } else {
        bindEvent();
    }
};
</script>

<template>
    <div class="wrap">
        <div class="content" @click="bindShake" v-if="!isBind">
            <h2>🥮 中秋快乐 🥮</h2>
            <p>中秋节到了，快点击页面后摇一摇手机看看你的本命月饼是什么吧。</p>
        </div>
        <template v-else>
            <div class="content">
                <p>🥮 🥮 🥮 🥮 🥮 🥮 🥮 🥮 🥮 🥮 🥮 🥮</p>
                <p>摇动手机，摇出你的本命月饼吧。</p>
                <p>如果一直摇不出，恭喜你也许抽到了稀有月饼。</p>
                <p>加大摇一摇幅度，多坚持一会试试吧。</p>
                <p>🥮 🥮 🥮 🥮 🥮 🥮 🥮 🥮 🥮 🥮 🥮 🥮</p>
            </div>
            <div class="dialog" v-if="showMoonCake">
                <div>
                    <h2>{{ moonCakes[showMoonCake].name }}</h2>
                    <img :src="'/' + moonCakes[showMoonCake].name + '.webp'" />
                    <p>{{ moonCakes[showMoonCake].feature }}</p>
                    <p>{{ moonCakes[showMoonCake].desc }}</p>
                </div>
                <button @click="closeDialog">关闭</button>
            </div>
            <!-- <div>
                <p v-for="log in logs">{{ JSON.stringify(log) }}</p>
            </div> -->
        </template>
    </div>
</template>

<style scoped>
.wrap {
    height: 100%;
    will-change: auto;
    transition: all 1s;
    background-image: url('/bg.jpeg');
    background-position: center;
    background-size: cover;
}
@keyframes dialogAni {
    0% {
        transform: scale(0.3);
    }
    end {
        transform: scale(1);
    }
}
.content {
    overflow: auto;
    display: flex;
    height: 100%;
    align-items: center;
    justify-content: center;
    flex-direction: column;
}
.content p {
    font-size: larger;
    font-weight: bolder;
    padding: 0 2em;
}
.dialog {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    will-change: auto;
    animation: dialogAni 1s ease-in-out;
    -webkit-backdrop-filter: saturate(50%) blur(8px);
    backdrop-filter: saturate(50%) blur(8px);
    background: rgba(255, 255, 255, 0.4);
    padding: 5em 3em;
    overflow: auto;
}
.dialog img {
    width: 100%;
    object-fit: cover;
    border-radius: 1em;
}
</style>
