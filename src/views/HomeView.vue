<template>
  <div>
    <MainLayout>
      <!-- <DataTable></DataTable> -->
      <h2 class="my-5 text-3xl">Employees</h2>
      <EmploymentDataTabel :apiData="filteredEmployees" />
      <h2 class="my-5 text-3xl">Departments</h2>

      <DepartmentDataTable />
    </MainLayout>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, computed, h, watch } from "vue";
import axios from "axios";
import { Search } from "lucide-vue-next";
import { PlusCircle } from "lucide-vue-next";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import MainLayout from "@/layouts/MainLayout.vue";
import EmploymentDataTabel from "@/components/EmploymentDataTabel.vue";
import DepartmentDataTable from "@/components/DepartmentDataTable.vue";
import { useToast } from "@/components/ui/toast/use-toast";
import Toaster from "@/components/ui/toast/Toaster.vue";
const { toast } = useToast();
import {
  Dialog,
  DialogContent,
  DialogDescription,
  DialogFooter,
  DialogHeader,
  DialogTitle,
  DialogTrigger,
} from "@/components/ui/dialog";
import { Label } from "@/components/ui/label";

const apiData = ref([]);
const searchTerm = ref("");

const name = ref("");

const manager = ref("");
const depName = ref("");
// search
let employeeDepartments = ref([]);

// async function fetchEmployeeDepartments(employeeId) {
//   try {
//     const response = await axios.get(
//       `https://interview.frontend.equinesolutions.co/api/Employees/${employeeId}/departments`
//     );
//     employeeDepartments.value = response.data;
//     console.log(response.data);
//   } catch (error) {
//     console.error("Error:", error);
//   }
// }

// onMounted(() => {
//   fetchEmployeeDepartments(64); // replace 1 with the actual employee ID
// });

function addedDep(values: any) {
  toast({
    title: "You submitted a Dep/Name:",
    description: h(
      "pre",
      { class: "mt-2 w-[340px] rounded-md bg-slate-950 p-4" },
      h("code", { class: "text-white" }, JSON.stringify(values, null, 2))
    ),
  });
}

onMounted(async () => {
  try {
    const response = await axios.get(
      "https://interview.frontend.equinesolutions.co/api/Employees/"
    );
    apiData.value = response.data;
    console.error(response.data);
  } catch (error) {
    console.error("Error:", error);
  }

  try {
    const response = await axios.get(
      "https://interview.frontend.equinesolutions.co/api/Departments/"
    );
    apiData.value = response.data;
    console.error(response.data);
  } catch (error) {
    console.error("Error:", error);
  }
});
</script>
