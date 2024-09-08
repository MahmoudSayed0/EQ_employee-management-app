<template>
  <div class="border mt-3 p-2 rounded-2xl mb-5">
    <div class="flex justify-between items-center">
      <div class="relative w-full max-w-sm items-center">
        <Input
          v-model="searchTerm"
          placeholder="Search by Employee"
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
            <!-- <DialogDescription class="text-yellow-500">
              Make sure to fill all inputs
            </DialogDescription> -->
            <!-- <div class="grid grid-cols-4 items-center gap-4">
              <Label for="department" class="text-right"> ID </Label>
              <Input id="id" type="number" v-model="id" class="col-span-3" />
            </div> -->
            <div class="grid gap-4 py-4">
              <!-- <div class="grid grid-cols-4 items-center gap-4">
                <Label for="first_name" class="text-right"> Manager </Label>
                <Input id="first_name" v-model="manager" class="col-span-3" />
              </div> -->
              <div class="grid grid-cols-4 items-center gap-4">
                <Label for="department" class="text-right">
                  Employeer name
                </Label>
                <Input id="department" v-model="name" class="col-span-3" />
              </div>
            </div>
            <DialogFooter>
              <Button
                @click="
                  () => {
                    addEmployee(),
                      addedEmployee({
                        name: name,
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
          <TableHead>Employeer name</TableHead>
          <TableHead>id</TableHead>
          <TableHead>Department Name</TableHead>
          <TableHead class="w-[112px]">options</TableHead>
        </TableRow>
      </TableHeader>
      <TableBody>
        <TableRow v-for="(item, index) in filteredEmployees" :key="index">
          <TableCell class="font-medium dark:text-green-300 text-green-500">
            {{ item.name }}
          </TableCell>
          <TableCell>{{ item.id }}</TableCell>
          <TableCell class="dark:text-orange-300 text-orange-500">{{
            employeeDepartments[item.id]
          }}</TableCell>
          <!-- Update exist Employee -->
          <div class="flex items-center content-center">
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
                <div v-if="editingItem" class="p-3">
                  <form @submit.prevent="updateItem">
                    <div class="flex">
                      <Label class="text-xs"> Employeer name </Label>
                      <Input
                        v-model="editingItem.name"
                        type="text"
                        placeholder="Name"
                        class="mb-2" />
                    </div>
                    <div class="flex">
                      <Input
                        v-model="editingItem.departmentId"
                        type="number"
                        placeholder="Department ID"
                        class="" />
                      <Button
                        type="submit"
                        class="flex-none w-24 mx-2 text-xs"
                        :disabled="!editingItem.departmentId"
                        @click="
                          editingItem.departmentId
                            ? (assignEmployeeToDepartment(
                                editingItem.id,
                                editingItem.departmentId
                              ),
                              updatedToast())
                            : updatedToast()
                        "
                        >Assign to dep</Button
                      >
                      <Button
                        :disabled="!editingItem.departmentId"
                        class="flex-none text-xs bg-red-600 text-white"
                        @click="
                          () => {
                            editingItem.departmentId
                              ? (removeEmployeeFromDepartment(
                                  editingItem.id,
                                  editingItem.departmentId
                                ),
                                removeDepError(editingItem.departmentId))
                              : updatedToast();
                          }
                        ">
                        <Trash class="size-5" />
                      </Button>
                    </div>
                    <Button
                      class="w-full p-0 mt-5 text-xs"
                      type="submit"
                      @click="
                        () => {
                          updateItem(), updatedToast();
                        }
                      "
                      :disabled="
                        originalName === editingItem.name ||
                        editingItem.name == ''
                          ? true
                          : false
                      "
                      >Save</Button
                    >
                  </form>
                </div>
              </DropdownMenuContent>
            </DropdownMenu>
            <div class="ml-2">
              <AlertDialog>
                <AlertDialogTrigger as-child>
                  <Button variant="destructive">
                    <Trash class="size-5" />
                  </Button>
                </AlertDialogTrigger>
                <AlertDialogContent>
                  <AlertDialogHeader>
                    <AlertDialogTitle>Are you sure?</AlertDialogTitle>
                    <AlertDialogDescription>
                      This action cannot be undone. This will permanently delete
                      your account and remove your data from our servers.
                    </AlertDialogDescription>
                  </AlertDialogHeader>
                  <AlertDialogFooter>
                    <AlertDialogCancel>Cancel</AlertDialogCancel>
                    <AlertDialogAction
                      variant="destructive"
                      @click="
                        () => {
                          deleteEmployee(item),
                            removeEmployee({
                              name: item.name,
                            });
                        }
                      "
                      >Delete</AlertDialogAction
                    >
                  </AlertDialogFooter>
                </AlertDialogContent>
              </AlertDialog>
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
import { ref, computed, onMounted, watch, h, reactive } from "vue";
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
import { toast } from "vue-sonner";

const { toast } = useToast();

// const { toast } = useToast();

const props = defineProps({
  apiData: Array,
  page: Number, // Add this line
});

const apiData = ref([]);
const name = ref("");
const department = ref("");

const itemsPerPage = 10;
const currentPage = ref(1);
const searchTerm = ref("");

const paginatedItems = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage;
  const end = start + itemsPerPage;
  return apiData.value.slice(start, end);
});

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

function updatedToast() {
  toast({
    description: h(
      "div",
      { class: "text-md " },
      h(
        "div",
        { class: "text-green-500" },
        "Up to Date - refresh to see updates -"
      )
    ),
  });
}
function depNotExist(val) {
  toast({
    description: h(
      "div",
      { class: "text-md " },
      h("div", { class: "text-red-600" }, `Department does not exist: ${val}`)
    ),
  });
}
function noNameError() {
  toast({
    title: "name required",
  });
}
function addedDep() {
  toast({
    title: "Added Success",
  });
}

function removeEmployee(val) {
  toast({
    title: "Removed",
    description: h(
      "div",
      { class: "mt-2 w-[340px] rounded-md  p-4" },
      h(
        "div",
        { class: "text-red-500 font-bold" },
        `EmployeeName: ${JSON.stringify(val, null, 2)}`
      )
    ),
  });
}
watch(currentPage, (newPage, oldPage) => {
  console.log(newPage, oldPage);
});
onMounted(async () => {
  try {
    const response = await axios.get(
      "https://interview.frontend.equinesolutions.co/api/Employees/"
    );
    apiData.value = response.data;
    // console.log(apiData.value);
  } catch (error) {
    console.error("Error:", error);
  }
});

const state = reactive({
  employees: [],
  employeeDepartments: {},
});

const fetchEmployeeDepartments = async (employeeId: number) => {
  try {
    const response = await axios.get(
      `https://interview.frontend.equinesolutions.co/api/Employees/${employeeId}/departments?page=1&paginate=10`
    );
    state.employeeDepartments[employeeId] = response.data
      .map((e: any) => e.name)
      .join(", ");
  } catch (error) {
    console.error("Error fetching departments:", error);
  }
};

const fetchEmployees = async () => {
  try {
    const response = await axios.get(
      "https://interview.frontend.equinesolutions.co/api/Employees/"
    );
    state.employees = response.data;
    for (const employee of state.employees) {
      fetchEmployeeDepartments(employee.id);
    }
  } catch (error) {
    console.error("Error fetching employees:", error);
  }
};
fetchEmployees();

const filteredEmployees = computed(() => {
  if (!searchTerm.value) {
    return apiData.value;
  }
  return apiData.value.filter((employee) =>
    employee.name.toLowerCase().includes(searchTerm.value.toLowerCase())
  );
});
const employeeDepartments = computed(() => state.employeeDepartments);
const departments = ref([]);

function startEditing(item) {
  originalName.value = item.name;
  editingItem = { ...item };
}

function saveChanges() {
  // Save the changes...
  originalName.value = editingItem.name; // Update the original name
}
onMounted(async () => {
  try {
    const response = await axios.get(
      "https://interview.frontend.equinesolutions.co/api/Departments"
    );
    departments.value = response.data.map((department) => department.id);
  } catch (error) {
    console.error("Error fetching departments:", error);
  }
});

const assignEmployeeToDepartment = async (
  employeeId: number,
  departmentId: number
) => {
  if (!departments.value.includes(departmentId)) {
    depNotExist(departmentId);
    return;
  }
  try {
    const response = await axios.post(
      `https://interview.frontend.equinesolutions.co/api/Departments/${departmentId}/employees`,
      { employeeId: employeeId }
    );
    return response.data;
  } catch (error) {
    console.error("Error assigning employee to department:", error);
  }
};

const removeEmployeeFromDepartment = async (
  employeeId: number,
  departmentId: number
) => {
  try {
    const response = await axios.delete(
      `https://interview.frontend.equinesolutions.co/api/Departments/${departmentId}/employees/${employeeId}`
    );
    return response.data;
  } catch (error) {
    console.error("Error assigning employee to department:", error);
  }
};

const addEmployee = async () => {
  const newEmployee = {
    // id: Date.now(),
    // manager: manager.value,
    name: name.value,
  };
  if (!name.value) {
    return;
  }
  const existingEmployee = apiData.value.find(
    (employee) => employee.name === name.value
  );
  if (existingEmployee) {
    return;
  }
  try {
    const response = await axios.post(
      `https://interview.frontend.equinesolutions.co/api/Employees/`,
      newEmployee
    );
    apiData.value.unshift(response.data);
  } catch (error) {
    console.error("Error:", error);
  }
};

let editingItem = ref({ name: "" });
let originalName = ref("");

const isNameChanged = computed(() => {
  return name.value !== originalName.value;
});

watch(
  () => editingItem.value.name,
  (newName, oldName) => {
    console.log(
      `Name changed from ${oldName} to ${newName}`,
      editingItem.value.name
    );
    // Handle the name change...
  }
);

function editItem(item) {
  originalName.value = item.name;
  editingItem.value = { ...item };
}

async function updateItem() {
  try {
    const response = await axios.patch(
      `https://interview.frontend.equinesolutions.co/api/Employees/${editingItem.value.id}`,
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

const deleteEmployee = async (employeeToDelete) => {
  try {
    await axios.delete(
      `https://interview.frontend.equinesolutions.co/api/Employees/${employeeToDelete.id}`
    );
    const index = apiData.value.findIndex(
      (employee) => employee.id === employeeToDelete.id
    );
    if (index !== -1) {
      apiData.value.splice(index, 1);
    }
  } catch (error) {
    console.error("Error:", error);
  }
};
</script>
