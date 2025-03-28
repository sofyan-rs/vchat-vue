<script setup lang="ts">
import { Button } from "@/components/ui/button";
import { Textarea } from "./components/ui/textarea";
import { ref } from "vue";
import DarkModeSwitcher from "./components/DarkModeSwitcher.vue";
import { GoogleGenAI } from "@google/genai";

interface Chat {
  text: string;
  user: "me" | "bot";
}

const inputText = ref<string>("");
const chatList = ref<Chat[]>([]);
const isLoading = ref<boolean>(false);

const ai = new GoogleGenAI({ apiKey: import.meta.env.VITE_GENAI_API_KEY });

async function generateContent(inputText: string) {
  isLoading.value = true;
  const response = await ai.models.generateContent({
    model: "gemini-2.0-flash",
    contents: `${inputText}\n\nPlease respond only in Japanese using hiragana, katakana, or kanji. Do not use English or romaji.`,
  });
  // console.log(response.text);
  if (response.text) {
    addChat(response.text, "bot");
    speakText(response.text);
    isLoading.value = false;
  }
}

async function speakText(text: string, speakerId = 20) {
  const baseUrl = "http://127.0.0.1:50021";

  // Create audio query
  const queryResponse = await fetch(
    `${baseUrl}/audio_query?text=${encodeURIComponent(
      text
    )}&speaker=${speakerId}`,
    {
      method: "POST",
    }
  );
  const queryData = await queryResponse.json();

  // Synthesize speech audio
  const synthResponse = await fetch(
    `${baseUrl}/synthesis?speaker=${speakerId}`,
    {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify(queryData),
    }
  );

  // Get the audio blob
  const audioBlob = await synthResponse.blob();
  const audioUrl = URL.createObjectURL(audioBlob);

  // Create an audio element and play the sound
  const audio = new Audio(audioUrl);
  audio.play();
}

const addChat = (text: string, user: "me" | "bot") => {
  inputText.value = "";
  chatList.value.push({ text, user });
  if (user === "me") {
    generateContent(text);
  }
};
</script>

<template>
  <div class="border-b-4">
    <div class="container mx-auto p-5 text-center flex justify-between">
      <h1 class="text-2xl font-semibold">VChat with Voicevox TTS</h1>
      <DarkModeSwitcher />
    </div>
  </div>
  <div class="container mx-auto p-5">
    <div
      class="min-h-[calc(100dvh-280px)] max-h-[calc(100dvh-280px)] mb-5 overflow-y-auto p-5"
    >
      <div
        v-for="chat in chatList"
        :key="chat.text"
        class="flex gap-5 space-y-5"
        :class="chat.user === 'me' ? 'justify-end' : 'justify-start'"
      >
        <div
          class="px-3 py-2 bg-gray-200 rounded-md mb-2"
          :class="
            chat.user === 'me'
              ? 'bg-sky-600 text-white'
              : 'bg-slate-200 text-slate-900'
          "
        >
          {{ chat.text }}
        </div>
      </div>
      <!-- Loading Chat -->
      <div v-if="isLoading" class="flex justify-center items-center">
        <div
          class="animate-spin rounded-full h-10 w-10 border-t-2 border-b-2 border-sky-500"
        ></div>
      </div>
    </div>
    <div class="flex flex-col gap-5">
      <Textarea v-model="inputText" placeholder="Write your chat here..." />
      <Button
        class="bg-sky-600 hover:bg-sky-700 text-white"
        size="lg"
        :disabled="!inputText.trim() || isLoading"
        @click="addChat(inputText, 'me')"
        >Send</Button
      >
    </div>
  </div>
</template>
