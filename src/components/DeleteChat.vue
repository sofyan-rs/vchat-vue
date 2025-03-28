<script setup lang="ts">
import { TrashIcon } from "lucide-vue-next";
import { Button } from "./ui/button";
import {
  AlertDialog,
  AlertDialogAction,
  AlertDialogCancel,
  AlertDialogContent,
  AlertDialogDescription,
  AlertDialogFooter,
  AlertDialogHeader,
  AlertDialogTitle,
  AlertDialogTrigger,
} from "@/components/ui/alert-dialog";

const props = defineProps<{
  deleteChat: () => void;
}>();
</script>

<template>
  <AlertDialog>
    <AlertDialogTrigger as-child
      ><Button variant="outline" class="text-red-600 p-3 h-auto">
        <TrashIcon /> </Button
    ></AlertDialogTrigger>
    <AlertDialogContent>
      <AlertDialogHeader>
        <AlertDialogTitle>Are you absolutely sure?</AlertDialogTitle>
        <AlertDialogDescription>
          This action cannot be undone. This will permanently delete your chat
          data.
        </AlertDialogDescription>
      </AlertDialogHeader>
      <AlertDialogFooter>
        <AlertDialogCancel>No, Cancel</AlertDialogCancel>
        <AlertDialogAction
          class="bg-red-600 text-white hover:bg-red-700"
          @click="props.deleteChat"
          >Yes, Continue</AlertDialogAction
        >
      </AlertDialogFooter>
    </AlertDialogContent>
  </AlertDialog>
</template>
