<script setup lang="ts">
import { Button } from "@/components/ui/button";
import { Textarea } from "./components/ui/textarea";
import { onMounted, ref } from "vue";
import DarkModeSwitcher from "./components/DarkModeSwitcher.vue";
import { GoogleGenAI } from "@google/genai";
import { DownloadIcon, Volume2Icon } from "lucide-vue-next";
import DeleteChat from "./components/DeleteChat.vue";
// import VchatAI from "./components/VchatAI.vue";

interface Chat {
  text: string;
  user: "me" | "bot";
}

const inputText = ref<string>("");
const chatList = ref<Chat[]>([]);
const isLoading = ref<boolean>(false);
const isSpeaking = ref<boolean>(false);

const ai = new GoogleGenAI({ apiKey: import.meta.env.VITE_GENAI_API_KEY });

async function generateContent(inputText: string) {
  isLoading.value = true;
  const response = await ai.models.generateContent({
    model: "gemini-2.0-flash",
    contents: `${inputText}\n\nPlease respond only in Japanese using hiragana, katakana, or kanji. Do not use English or romaji.`,
    // contents: `${inputText}`,
  });
  // console.log(response.text);
  if (response.text) {
    addChat(response.text, "bot");
    if (isSpeaking.value === false) {
      speakText(response.text);
    }
    saveChat();
    isLoading.value = false;
  }
}

// Speak text using Voicevox
// Get speakerId from speakers.json and https://voicevox.hiroshiba.jp/
async function speakText(text: string, speakerId = 18) {
  isSpeaking.value = true;
  const baseUrl = "http://127.0.0.1:50021";

  const queryResponse = await fetch(
    `${baseUrl}/audio_query?text=${encodeURIComponent(
      text
    )}&speaker=${speakerId}`,
    { method: "POST" }
  );
  const queryData = await queryResponse.json();

  const synthResponse = await fetch(
    `${baseUrl}/synthesis?speaker=${speakerId}`,
    {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(queryData),
    }
  );

  const audioBlob = await synthResponse.blob();
  const audioUrl = URL.createObjectURL(audioBlob);

  const audio = new Audio(audioUrl);
  audio.play();
  audio.onended = () => {
    isSpeaking.value = false;
  };

  return { audioBlob, audioUrl };
}

async function downloadVoice(text: string, speakerId = 18) {
  const { audioBlob } = await speakText(text, speakerId);
  const blobUrl = URL.createObjectURL(audioBlob);

  const link = document.createElement("a");
  link.href = blobUrl;
  link.download = "voicevox_audio.wav";
  document.body.appendChild(link);
  link.click();
  document.body.removeChild(link);
}

const saveChat = () => {
  const chatData = JSON.stringify(chatList.value);
  localStorage.setItem("chatData", chatData);
};

const loadChat = () => {
  const chatData = localStorage.getItem("chatData");
  if (chatData) {
    chatList.value = JSON.parse(chatData);
  }
};

const addChat = (text: string, user: "me" | "bot") => {
  inputText.value = "";
  chatList.value.push({ text, user });
  if (user === "me") {
    generateContent(text);
    saveChat();
  }
};

const deleteChat = () => {
  chatList.value = [];
  saveChat();
};

onMounted(() => {
  loadChat();
});
</script>

<template>
  <div class="border-b-4">
    <div class="container mx-auto p-5 gap-5 text-center flex justify-between">
      <h1 class="text-2xl font-semibold">VChat Voicevox</h1>
      <div class="flex gap-2">
        <DarkModeSwitcher />
        <DeleteChat :delete-chat="deleteChat" />
      </div>
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
          class="px-3 py-2 bg-gray-200 rounded-md mb-2 relative"
          :class="
            chat.user === 'me'
              ? 'bg-sky-600 text-white'
              : 'bg-slate-200 text-slate-900'
          "
        >
          {{ chat.text }}
          <button
            v-if="chat.user == 'bot'"
            class="bg-sky-500 text-white p-2 absolute -top-6 -right-4 rounded-md disabled:opacity-70"
            @click="speakText(chat.text)"
            :disabled="isSpeaking"
          >
            <Volume2Icon class="size-5" />
          </button>
          <button
            v-if="chat.user == 'bot'"
            class="bg-emerald-500 text-white p-2 absolute -top-6 right-6 rounded-md disabled:opacity-70"
            @click="downloadVoice(chat.text)"
          >
            <DownloadIcon class="size-5" />
          </button>
        </div>
      </div>
      <!-- Empty Chat -->
      <div v-if="chatList.length === 0">
        <p class="text-center text-slate-400">No chat yet</p>
      </div>
      <!-- Loading Chat -->
      <div v-if="isLoading" class="flex justify-center items-center">
        <div
          class="animate-spin rounded-full h-10 w-10 border-t-2 border-b-2 border-sky-500"
        ></div>
      </div>
    </div>
    <!-- <VchatAI /> -->
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
