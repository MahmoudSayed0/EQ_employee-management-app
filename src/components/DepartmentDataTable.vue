<template>
  <div class="border mt-3 p-2 rounded-2xl mb-5">
    <div class="flex justify-between items-center">
      <div class="relative w-full max-w-sm items-center">
        <Input
          v-model="searchTerm"
          placeholder="Search by Department"
          id="search"
          type="text"
          class="pl-10 rounded-xl" />
        <span
          class="absolute start-0 inset-y-0 flex items-center justify-center px-2">
          <Search class="size-6 text-muted-foreground" />
        </span>
      </div>
      <div>
        <!-- Create new Employee -->
        <Dialog>
          <DialogTrigger as-child>
            <Button variant="outline" class="rounded-xl">
              <PlusCircle class="size-5 mr-2 text-muted-foreground" />
              Add new employee
            </Button>
          </DialogTrigger>
          <DialogContent class="sm:max-w-[425px] mr-4">
            <DialogTitle>Add new Employee</DialogTitle>
            <DialogDescription class="text-red-500">
              Make sure to fill all inputs
            </DialogDescription>
            <!-- <div class="grid grid-cols-4 items-center gap-4">
              <Label for="department" class="text-right"> ID </Label>
              <Input id="id" type="number" v-model="id" class="col-span-3" />
            </div> -->
            <div class="grid gap-4 py-4">
              <div class="grid grid-cols-4 items-center gap-4">
                <Label for="first_name" class="text-right"> Manager </Label>
                <Input id="first_name" v-model="manager" class="col-span-3" />
              </div>
              <div class="grid grid-cols-4 items-center gap-4">
                <Label for="department" class="text-right">
                  department name
                </Label>
                <Input id="department" v-model="name" class="col-span-3" />
              </div>
            </div>
            <DialogFooter>
              <Button
                @click="
                  () => {
                    addDep(),
                      addedDep({
                        name: name,
                        manager: manager,
                      });
                  }
                ">
                save
              </Button>
            </DialogFooter>
          </DialogContent>
        </Dialog>
      </div>
    </div>
    <Table>
      <TableHeader>
        <TableRow>
          <TableHead>Department Name</TableHead>
          <TableHead>Manager</TableHead>
          <TableHead>id</TableHead>
          <TableHead class="w-[112px]">options</TableHead>
        </TableRow>
      </TableHeader>
      <TableBody>
        <TableRow v-for="(item, index) in filteredEmployees" :key="index">
          <TableCell class="font-medium dark:text-orange-300 text-orange-500">
            {{ item.name }}
          </TableCell>
          <TableCell>{{ item.manager }}</TableCell>
          <TableCell>{{ item.id }}</TableCell>

          <!-- Update exist Employee -->
          <div class="flex float-end justify-between">
            <DropdownMenu>
              <DropdownMenuTrigger as-child>
                <Button
                  variant="outline"
                  @click="
                    () => {
                      editItem(item);
                    }
                  ">
                  <Edit class="size-5" />
                </Button>
              </DropdownMenuTrigger>
              <DropdownMenuContent>
                <div class="p-3 flex" v-if="editingItem">
                  <form @submit.prevent="updateItem">
                    <div class="flex items-center content-center">
                      <Label class="text-xs w-24"> Department name </Label>
                      <Input
                        v-model="editingItem.name"
                        type="text"
                        class="mb-2"
                        placeholder="Name" />
                    </div>
                    <div class="flex items-center content-center">
                      <Label class="text-xs w-24"> Manager </Label>

                      <Input
                        v-model="editingItem.manager"
                        type="text"
                        placeholder="ID" />
                    </div>
                    <Button
                      type="submit"
                      class="w-full p-0 mt-5 text-xs"
                      @click="updetedEmplyee"
                      >Save</Button
                    >
                  </form>
                </div>
              </DropdownMenuContent>
            </DropdownMenu>
            <div class="ml-2">
              <Button variant="destructive" @click="deleteDep(item)">
                <Trash class="size-5" />
              </Button>
            </div>
          </div>
        </TableRow>
      </TableBody>
    </Table>
  </div>
  <Pagination
    v-model="currentPage"
    :total="Math.ceil(apiData.length / itemsPerPage)"
    :sibling-count="1"
    show-edges
    :default-page="1">
    <PaginationList v-slot="{ items }" class="flex items-center gap-1">
      <PaginationFirst />
      <PaginationPrev />

      <template v-for="(item, index) in items">
        <PaginationListItem
          v-if="item.type === 'page'"
          :key="index"
          :value="item.value"
          as-child>
          <Button
            class="w-9 h-9 p-0"
            :variant="item.value === page ? 'default' : 'outline'">
            {{ item.value }}
          </Button>
        </PaginationListItem>
        <PaginationEllipsis v-else :key="item.type" :index="index" />
      </template>

      <PaginationNext />
      <PaginationLast />
    </PaginationList>
  </Pagination>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, watch, h } from "vue";
import axios from "axios";
import Button from "./ui/button/Button.vue";
import { Trash } from "lucide-vue-next";
import { Edit } from "lucide-vue-next";
import { Search } from "lucide-vue-next";
import { PlusCircle } from "lucide-vue-next";

import {
  DropdownMenu,
  DropdownMenuCheckboxItem,
  DropdownMenuContent,
  DropdownMenuLabel,
  DropdownMenuSeparator,
  DropdownMenuTrigger,
} from "@/components/ui/dropdown-menu";

import {
  Dialog,
  DialogContent,
  DialogDescription,
  DialogFooter,
  DialogHeader,
  DialogTitle,
  DialogTrigger,
} from "@/components/ui/dialog";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import {
  Pagination,
  PaginationEllipsis,
  PaginationFirst,
  PaginationLast,
  PaginationList,
  PaginationListItem,
  PaginationNext,
  PaginationPrev,
} from "@/components/ui/pagination";

import {
  Table,
  TableBody,
  TableCell,
  TableCaption,
  TableHead,
  TableHeader,
  TableRow,
} from "@/components/ui/table";
import { useToast } from "@/components/ui/toast/use-toast";
import Toaster from "@/components/ui/toast/Toaster.vue";
const { toast } = useToast();

const props = defineProps({
  apiData: Array,
  page: Number, // Add this line
});

const apiData = ref([]);
const name = ref("");
const manager = ref("");

const itemsPerPage = 10;
const currentPage = ref(1);
const searchTerm = ref("");

const paginatedItems = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage;
  const end = start + itemsPerPage;
  return apiData.value.slice(start, end);
});

const filteredEmployees = computed(() => {
  if (!searchTerm.value) {
    return apiData.value;
  }
  return apiData.value.filter((employee) =>
    employee.name.toLowerCase().includes(searchTerm.value.toLowerCase())
  );
});
// watch(filteredEmployees, (newPage, oldPage) => {
//   console.log(newPage, oldPage);
// });
function addedEmployee(values: any) {
  toast({
    title: "You updated the following values:",
    description: h(
      "pre",
      { class: "mt-2 w-[340px] rounded-md bg-gray-950 p-4" },
      h("code", { class: "text-white" }, JSON.stringify(values, null, 2))
    ),
  });
}
function updetedEmplyee() {
  toast({
    title: "Edited!",
  });
}
watch(currentPage, (newPage, oldPage) => {
  console.log(newPage, oldPage);
});
onMounted(async () => {
  try {
    const response = await axios.get(
      "https://interview.frontend.equinesolutions.co/api/Departments/"
    );
    apiData.value = response.data;
    console.log(apiData.value);
  } catch (error) {
    console.error("Error:", error);
  }
});

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

const addDep = async () => {
  const newDep = {
    // id: Date.now(),
    manager: manager.value,
    name: name.value,
  };
  if (!name.value) {
    return;
  }
  const existingEmployee = apiData.value.find(
    (dep) => dep.name === name.value && dep.manager == manager.value
  );
  if (existingEmployee) {
    return;
  }
  try {
    const response = await axios.post(
      `https://interview.frontend.equinesolutions.co/api/Departments/`,
      newDep
    );
    apiData.value.push(response.data);
  } catch (error) {
    console.error("Error:", error);
  }
};

// edit values

let editingItem = ref(null);

function editItem(item) {
  editingItem.value = { ...item };
}

async function updateItem() {
  try {
    const response = await axios.patch(
      `https://interview.frontend.equinesolutions.co/api/Departments/${editingItem.value.id}`,
      editingItem.value
    );
    if (response.status === 200) {
      console.log("Item updated successfully");
    } else {
      console.log("Failed to update item");
    }
  } catch (error) {
    console.error("Error:", error);
  }
  editingItem.value = null;
}

function error422(val: any) {
  toast({
    title:
      "You can not remmove this department There are employees in this department. You must remove them first.",
    description: h(
      "div",
      { class: "mt-2 w-[340px] rounded-md  p-4" },
      h("div", { class: "text-red-500 font-bold" }, `Employment: ${val}`)
    ),
  });
}
let employeeDepartments = ref([]);

async function fetchEmployeeDepartments(departmentId) {
  try {
    const response = await axios.get(
      `https://interview.frontend.equinesolutions.co/api/Departments/${departmentId}/employees`
    );
    employeeDepartments.value = response.data;
    return response.data.map((e) => `id: ${e.id}, name: ${e.name}`);
  } catch (error) {
    console.error("Error:", error);
  }
}
const deleteDep = async (depDelete) => {
  try {
    const employees = await fetchEmployeeDepartments(depDelete.id);
    if (employees && employees.length > 0) {
      error422(employees);
    } else {
      try {
        await axios.delete(
          `https://interview.frontend.equinesolutions.co/api/Departments/${depDelete.id}`
        );
      } catch (error) {
        console.error("Error", error.message);
      }
      const index = apiData.value.findIndex((dep) => depDelete.id === dep.id);
      if (index !== -1) {
        apiData.value.splice(index, 1);
      }
    }
  } catch (error) {
    console.error("Error:", error);
  }
};
</script>
