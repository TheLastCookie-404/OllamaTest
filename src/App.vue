<template>
  <div class="fixed top-0 left-0 size-full p-3 grid place-items-center">
    <div class="size-full flex flex-col gap-3">
      <div class="bg-base-200 size-full rounded-box">
        <div
          v-html="llmResponse"
          class="size-full max-w-none prose p-5 overflow-auto"></div>
      </div>
      <form class="flex gap-3">
        <div class="bg-base-200 flex gap-3 items-center px-5 rounded-field">
          <label class="label">Thoughts: </label>
          <label class="swap text-sm">
            <input v-model="isThinkVisible" type="checkbox" />
            <span class="swap-on text-success">ON</span>
            <span class="swap-off text-error">OFF</span>
          </label>
        </div>
        <input
          v-model="usrMessage"
          type="text"
          placeholder="Type here"
          class="input w-full" />
        <button
          @click="sendMessage(usrMessage)"
          class="btn btn-success"
          :disabled="isBtnDisabled">
          Send
        </button>
      </form>
    </div>
  </div>
</template>

<script lang="ts" setup>
  import { ref, onMounted } from "vue";
  import axios from "axios";
  import { useStorage } from "@vueuse/core";
  import { marked } from "marked";
  import { Ollama } from "ollama";

  const usrMessage = ref<string | undefined>();
  const llmResponse = useStorage<string>("llmResponse", "");
  const isBtnDisabled = ref<boolean>(false);
  const isThinkVisible = ref<boolean>(false);

  // const ollama = new Ollama({ host: "http://127.0.0.1:11434/" });

  // onMounted(() => {
  //   lol();
  // });

  // async function lol() {
  //   console.log(
  //     await ollama.chat({
  //       model: "deepseek-r1:8b",
  //       messages: [{ role: "user", content: "Why is the sky blue?" }],
  //     })
  //   );
  // }

  function sendMessage(message: string | undefined) {
    isBtnDisabled.value = true;

    axios
      .post(
        "http://127.0.0.1:11434/api/generate",
        {
          model: "deepseek-r1:8b",
          prompt: message,
          stream: false,
        },
        {
          headers: {
            "Content-Type": "application/json",
          },
        }
      )
      .then((response) => {
        console.log(response);
        llmResponse.value = response.data.response;

        isBtnDisabled.value = false;
      })
      .catch((error) => {
        console.error(error);
      });
  }
</script>

<style scoped lang="css">
  :deep(think) {
    color: var(--color-info);
    margin-bottom: 10px;
    display: v-bind("`${isThinkVisible ? 'inline-block' : 'none'}`");
  }
</style>
