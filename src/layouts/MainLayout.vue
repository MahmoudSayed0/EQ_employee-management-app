<template>
  <div class="flex flex-col min-h-screen px-10 pt-5">
    <header class="text-white py-4">
      <DropdownMenu>
        <DropdownMenuTrigger as-child>
          <Button variant="outline">
            <Icon
              icon="radix-icons:moon"
              class="h-[1.2rem] w-[1.2rem] rotate-0 text-black scale-100 transition-all dark:-rotate-90 dark:scale-0" />
            <Icon
              icon="radix-icons:sun"
              class="absolute h-[1.2rem] w-[1.2rem] rotate-90 scale-0 transition-all dark:rotate-0 dark:scale-100" />
            <span class="sr-only">Toggle theme</span>
          </Button>
        </DropdownMenuTrigger>
        <DropdownMenuContent align="end">
          <DropdownMenuItem @click="mode = 'light'"> Light </DropdownMenuItem>
          <DropdownMenuItem @click="mode = 'dark'"> Dark </DropdownMenuItem>
          <DropdownMenuItem @click="mode = 'auto'"> System </DropdownMenuItem>
        </DropdownMenuContent>
      </DropdownMenu>
    </header>
    <div class="flex flex-col items-center justify-center">
      <img src="../assets/logo.png" alt="" width="100" />
      <div class="flex flex-col justify-center content-center items-center">
        <h1 class="text-xl mt-5 text-center">
          Vue Frontend Developer Interview Assignment for Vue Frontend Developer
          position at Equine Solutions
        </h1>
        <p class="text-sm mt-5 w-1/2 text-center">
          Build a Vue 3 application that interacts with an API to manage
          employees and departments. The goal is to demonstrate your ability to
          work with RESTful APIs, handle CRUD operations, manage relationships
          between data entities, and handle errors and exceptions gracefully.
        </p>
      </div>
    </div>

    <main class="">
      <slot></slot>
    </main>

    <footer class="flex items-center justify-center text-white py-4">
      thank you for your patience❤️
    </footer>
  </div>
</template>

<script setup lang="ts">
import { useColorMode } from "@vueuse/core";
import { Icon } from "@iconify/vue";
import { Button } from "@/components/ui/button";
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuTrigger,
} from "@/components/ui/dropdown-menu";

const mode = useColorMode();
</script>
