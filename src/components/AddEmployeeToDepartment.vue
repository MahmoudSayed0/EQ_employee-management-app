<template>
  <div class="grid gap-4 py-4">
    <div class="grid grid-cols-4 items-center gap-4">
      <Label for="employeeId" class="text-right"> Employee ID </Label>
      <Input
        id="employeeId"
        v-model="employeeId"
        class="col-span-3"
        type="number" />
    </div>
    <div class="grid grid-cols-4 items-center gap-4">
      <Label for="departmentId" class="text-right"> Department ID </Label>
      <Input
        id="departmentId"
        v-model="departmentId"
        class="col-span-3"
        type="number" />
    </div>
    <Button @click="addEmployeeToDepartment">
      Add Employee to Department
    </Button>
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";
import axios from "axios";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";

let employeeId = ref("");
let departmentId = ref("");

const addEmployeeToDepartment = async () => {
  try {
    await axios.post(
      `https://interview.frontend.equinesolutions.co/api/Departments/${departmentId.value}/employees/${employeeId.value}`
    );
    console.log("Employee added to department successfully");
  } catch (error) {
    console.error("Error:", error);
  }
};
</script>
