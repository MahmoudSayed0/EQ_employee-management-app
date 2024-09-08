<script setup lang="ts">
import { Button } from "@/components/ui/button";
import { useToast } from "@/components/ui/toast/use-toast";

const { toast } = useToast();
</script>

<template>
  <Button
    variant="outline"
    @click="
      () => {
        toast({
          description: 'you added a new employee ✅',
        });
      }
    ">
    Show Toast
  </Button>
</template>
