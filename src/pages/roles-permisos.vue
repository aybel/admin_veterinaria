<script setup lang="ts">
import EditRolDialog from "@/components/elements/rol/EditRolDialog.vue";
import { onMounted, watch } from "vue";

//import data from "@/views/js/jsdatatable.js";
const data = ref([]);
const paginador = async () => {
  const resp = await $api("/rol?search=" + (searchQuery.value ?? ""), {
    method: "GET",
    onResponseError({ response }) {
      console.error(response._data.error);
    },
  });
  data.value = resp.roles;
};
const editItem = (item: any) => {
  role_selected.value = item;
  isEditRolDialogVisible.value = true;
};
const deleteItem = (item: any) => {
  role_selected_deleted.value = item;
  isDeleteRolDialogVisible.value = true;
};
onMounted(() => {
  paginador();
});
const headers = [
  { title: "id", key: "id" },
  { title: "Rol", key: "name" },
  { title: "Permisos", key: "permissions_pluck" },
  { title: "Fecha", key: "created_at" },
  { title: "Acciones", key: "actions", sortable: false },
];

const searchQuery = ref("");
const isAddRolDialogVisible = ref(false);
const isEditRolDialogVisible = ref(false);
const isDeleteRolDialogVisible = ref(false);
const role_selected = ref(null);
const role_selected_deleted = ref(null);

watch(isEditRolDialogVisible, (event) => {
  if (event === false) {
    role_selected.value = null;
  }
});
watch(isDeleteRolDialogVisible, (event) => {
  if (event === false) {
    role_selected_deleted.value = null;
  }
});
</script>

<template>
  <div>
    <VCard class="mb-6">
      <VCardItem class="pb-4">
        <VCardTitle>Roles y permisos</VCardTitle>
      </VCardItem>
      <VCardText class="d-flex flex-wrap gap-4">
        <div class="d-flex align-center">
          <!-- 👉 Search  -->
          <VTextField
            v-model="searchQuery"
            placeholder="Buscar rol"
            style="inline-size: 200px"
            density="compact"
            class="me-3"
            @keyup.enter="paginador"
          />
        </div>

        <VSpacer />

        <div class="d-flex gap-x-4 align-center">
          <VBtn
            color="primary"
            prepend-icon="ri-add-line"
            @click="isAddRolDialogVisible = !isAddRolDialogVisible"
          >
            Agregar rol
          </VBtn>
        </div>
      </VCardText>
      <VDataTable
        :headers="headers"
        :items="data"
        :items-per-page="5"
        class="text-no-wrap"
      >
        <template #item.id="{ item }">
          <span class="text-sm">{{ item.id }}</span>
        </template>
        <template #item.permissions_pluck="{ item }">
          <ul>
            <li
              v-for="(permission, index) in item.permissions_pluck"
              :key="index"
            >
              <span class="text-sm">{{ permission }}</span>
            </li>
          </ul>
        </template>
        <!-- acciones-->
        <template #item.actions="{ item }">
          <div class="d-flex gap-2">
            <VBtn
              color="primary"
              size="small"
              prepend-icon="ri-edit-line"
              @click="editItem(item)"
            >
              Editar
            </VBtn>
            <VBtn
              color="error"
              size="small"
              prepend-icon="ri-delete-bin-5-line"
              @click="deleteItem(item)"
            >
              Eliminar
            </VBtn>
          </div>
        </template>
      </VDataTable>
    </VCard>
    <AddRolDialog
      v-model:is-dialog-visible="isAddRolDialogVisible"
      @addRole="paginador"
    />
    <EditRolDialog
      v-if="role_selected"
      v-model:is-dialog-visible="isEditRolDialogVisible"
      :role_selected="role_selected"
      @addRole="paginador"
    />
    <DeleteRolDialog
      v-if="role_selected_deleted"
      v-model:is-dialog-visible="isDeleteRolDialogVisible"
      :role_selected_deleted="role_selected_deleted"
      @addRole="paginador"
    />
  </div>
</template>
