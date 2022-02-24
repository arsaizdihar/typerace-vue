<script>
import { computed, onMounted, ref, watch } from "vue";

export default {
  setup() {
    const text = ref("");
    onMounted(async () => {
      const res = await fetch(
        "https://typesfaster.herokuapp.com/api/random-words"
      );
      text.value = await res.json();
    });
    const start = ref();
    const input = ref("");

    const inputRef = ref();
    const correctInput = computed(() => {
      let res = "";
      for (let i = 0; i < text.value.length; i++) {
        if (text.value[i] !== input.value[i]) {
          return res;
        }
        res += text.value[i];
      }
      if (res.length === text.value.length) {
        start.value = null;
      }
      return res;
    });
    const wrongInput = computed(() => {
      return text.value.slice(correctInput.value.length, input.value.length);
    });

    const textRest = computed(() => {
      return text.value.slice(
        correctInput.value.length + wrongInput.value.length
      );
    });
    watch(input, () => {
      if (!start.value && input.value.length > 0) {
        start.value = new Date();
      }
    });
    const wpm = ref(0);

    const interval = ref();

    watch(start, () => {
      if (start.value) {
        interval.value = setInterval(() => {
          const startTime = start.value;
          if (!startTime) return;
          const time = new Date().getTime() - startTime.getTime();
          if (time < 500) return;
          wpm.value = (
            correctInput.value.length /
            5 /
            (time / 1000 / 60)
          ).toFixed(2);
        }, 500);
      } else {
        if (interval.value) {
          clearInterval(interval.value);
        }
      }
    });

    return { text, input, inputRef, correctInput, wrongInput, textRest, wpm };
  },
};
</script>

<template>
  <h2 class="text-center text-3xl font-bold">{{ wpm }}</h2>
  <div
    class="max-w-screen-sm mx-auto font-mono text-xl bg-blue-200 rounded-md p-4"
    @click="inputRef.focus()"
  >
    <p>
      <span class="text-blue-500">{{ correctInput }}</span>
      <span class="bg-red-500 text-white">{{ wrongInput }}</span>
      <span>{{ textRest }}</span>
    </p>
    <input v-model="input" class="w-0 h-0 absolute" ref="inputRef" />
  </div>
</template>
