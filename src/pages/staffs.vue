<script setup>
import { ref } from "vue";

const headers = [
  { title: "ID", key: "id" },
  { title: "avatar", key: "imagen" },
  { title: "Nombre y apellido", key: "full_name" },
  { title: "Teléfono", key: "phone" },
  {
    title: "Documento",
    key: "document_full",
  },
  { title: "Rol", key: "role_name" },
  { title: "Correo", key: "email" },
  { title: "Acciones", key: "actions", sortable: false },
];
const searchQuery = ref("");
const isAddStaffDialogVisible = ref(false);
const isEditStaffDialogVisible = ref(false);
const isDeleteStaffDialogVisible = ref(false);
const user_selected_deleted = ref(null);
const data = ref([]);
const roles = ref([]);
onMounted(() => {
  paginador();
});
const paginador = async () => {
  const resp = await $api("/staff?search=" + (searchQuery.value ?? ""), {
    method: "GET",
    onResponseError({ response }) {
      console.error(response._data.error);
    },
  });
  data.value = resp.users.data;
  roles.value = resp.roles;
};

watch(isEditStaffDialogVisible, (event) => {
  if (event === false) {
    user_selected_deleted.value = null;
  }
});
watch(isDeleteStaffDialogVisible, (event) => {
  if (event === false) {
    user_selected_deleted.value = null;
  }
});
</script>
<template>
  <div>
    <VCardItem class="pb-4">
      <VCardTitle>Usuarios</VCardTitle>
    </VCardItem>
    <VCardText class="d-flex flex-wrap gap-4">
      <div class="d-flex align-center">
        <!-- 👉 Search  -->
        <VTextField
          v-model="searchQuery"
          placeholder="Buscar Staff"
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
          @click="isAddStaffDialogVisible = !isAddStaffDialogVisible"
        >
          Nuevo Usuario
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
      
      <template #item.imagen="{ item }">
        <div class="d-flex align-center">
          <VAvatar
            size="32"
            :color="item.avatar ? '' : 'primary'"
            :class="item.avatar ? '' : 'v-avatar-light-bg primary--text'"
            :variant="!item.avatar ? 'tonal' : undefined"
          >
            <VImg v-if="item.avatar" :src="item.avatar" />
            <span v-else class="text-sm">{{ avatarText(item.full_name) }}</span>
          </VAvatar>
        </div>
      </template>
      <template #item.document_full="{ item }">
        <div class="d-flex align-center">
          <div class="d-flex flex-column ms-3">
            <span
              class="d-block font-weight-medium text-high-emphasis text-truncate"
              >{{ item.num_document }}</span
            >
            <small>{{ item.type_document }}</small>
          </div>
        </div>
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
    <AddStaffDialog
      v-model:is-dialog-visible="isAddStaffDialogVisible"
      :rolesProp="roles"
      @addStaff="paginador"
    />
    <EditStaffDialog
      v-model:is-dialog-visible="isEditStaffDialogVisible"
      @editStaff="paginador"
    />
    <DeleteStaffDialog
      v-model:is-dialog-visible="isDeleteStaffDialogVisible"
      @deleteStaff="paginador"
    />
  </div>
</template>
