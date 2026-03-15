<script setup lang="ts">
import { ref, onMounted, computed } from "vue";
import LucideIcon from "./components/LucideIcon.vue";

const visible = ref<boolean>(false);

const ready = ref<boolean>(false);
const playing = ref<boolean>(false);
const ended = ref<boolean>(false);
const color = ref<string | null>(null);
const rid = ref<string>("0");
const remaining = ref<string | null>(null);
const file = ref<string | null>(null);
const audio = ref<HTMLAudioElement | null>(null);
const startedDate = ref<Date | null>(null);

const canPlay = computed<boolean>(() => ready.value && !playing.value);
const canStop = computed<boolean>(() => ready.value && playing.value);

function onChangeFile(event: InputEvent): void {
    ready.value = false;
    file.value = URL.createObjectURL(
        (event.target as HTMLInputElement).files[0],
    );
    audio.value = new Audio(file.value);
    audio.value.addEventListener("loadeddata", onAudioLoadedData);
    audio.value.addEventListener("ended", onAudioEnded);
}

function onAudioLoadedData(): void {
    ready.value = true;
}

function onPlay(): void {
    ended.value = false;
    playing.value = true;
    startedDate.value = new Date();
    setTimeout(play, 5000);
}

async function play() {
    if (audio.value) {
        await audio.value.play();
    }
}

function onAudioEnded(): void {
    ended.value = true;
    onStop();
}

function onStop(): void {
    if (audio.value) {
        audio.value.pause();
        audio.value.currentTime = 0;
        playing.value = false;
    }
}

function getColor(): string | null {
    if (
        startedDate.value &&
        playing.value &&
        (new Date().getTime() - startedDate.value.getTime()) / 1000 <= 5
    ) {
        return "#f00";
    } else if (playing.value) {
        return "#0f0";
    } else if (ended.value) {
        return "#f50";
    }
    return null;
}

function getRemaining(): string {
    if (ended.value) {
        return "00:00";
    }
    if (!ready.value || !playing.value) {
        return "00:05";
    }
    const d1 = Math.floor(
        (new Date().getTime() - startedDate.value.getTime()) / 1000,
    );
    if (d1 < 5) {
        return `00:${String(Math.floor(5 - d1)).padStart(2, "0")}`;
    }
    const d2 = audio.value.duration - d1 + 5;
    return `${String(Math.floor(d2 / 60)).padStart(2, "0")}:${String(Math.floor(d2) % 60).padStart(2, "0")}`;
}

function refresh(): void {
    rid.value = Math.random().toString();
    const newColor = getColor();
    if (newColor != color.value) {
        color.value = newColor;
        document.body.setAttribute(
            "style",
            color.value ? "background: " + color.value : "!important",
        );
    }
    remaining.value = getRemaining();
}

onMounted(() => {
    setTimeout(() => {
        visible.value = true;
    });
    setInterval(refresh, 200);
});
</script>

<template>
    <main :style="{ display: visible ? 'inherit' : 'none' }">
        <h1>
            <LucideIcon name="cassette-tape" />
            Tape Record
        </h1>
        <br />
        <p>
            <input
                id="file"
                type="file"
                accept="audio/*"
                @change="onChangeFile"
            /><br />
        </p>
        <div>
            <button :disabled="!canPlay" @click="onPlay">
                <LucideIcon name="play" /> Play</button
            >&nbsp;
            <button :disabled="!canStop" @click="onStop">
                <LucideIcon name="square" /> Stop
            </button>
        </div>
        <h1 :id="rid" :title="rid" :style="{ color: color }">
            {{ remaining }}
        </h1>
        <hr />
        <small class="footer">
            <LucideIcon name="at-sign" />
            &nbsp;
            <a href="https://github.com/klemek" target="_blank">klemek</a>
            -
            <LucideIcon name="github" />
            &nbsp;
            <a href="https://github.com/klemek/tape-record" target="_blank">
                Repository
            </a>
            - 2025
        </small>
    </main>
</template>

<style scoped>
.button {
    display: block;
    width: 100%;
    text-decoration: none;
    padding: 1em;
    margin-bottom: 0.75em;
    border: 1px solid var(--color-primary);
    border-radius: 0.5em;
    background-color: var(--background);
    cursor: pointer;
    font-size: 1.333em;
}

.button:hover {
    background-color: var(--background-secondary);
}

.footer {
    opacity: 50%;
}
</style>
