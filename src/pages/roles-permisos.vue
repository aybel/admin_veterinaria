<script setup lang="ts">
import data from "@/views/js/jsdatatable.js";

const headers = [
  { title: "NAME", key: "fullName" },
  { title: "EMAIL", key: "email" },
  { title: "DATE", key: "startDate" },
  { title: "SALARY", key: "salary" },
  { title: "AGE", key: "age" },
  { title: "STATUS", key: "status" },
];

const resolveStatusVariant = (status: number) => {
  if (status === 1) return { color: "primary", text: "Current" };
  else if (status === 2) return { color: "success", text: "Professional" };
  else if (status === 3) return { color: "error", text: "Rejected" };
  else if (status === 4) return { color: "warning", text: "Resigned" };
  else return { color: "info", text: "Applied" };
};
const searchQuery = ref("");
const isAddRolDialogVisible = ref(false);
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
        show-select
        class="text-no-wrap"
      >
        <!-- full name -->
        <template #item.fullName="{ item }">
          <div class="d-flex align-center">
            <VAvatar
              size="32"
              :color="item.avatar ? '' : 'primary'"
              :class="item.avatar ? '' : 'v-avatar-light-bg primary--text'"
              :variant="!item.avatar ? 'tonal' : undefined"
            >
              <VImg v-if="item.avatar" :src="item.avatar" />
              <span v-else class="text-sm">{{
                avatarText(item.fullName)
              }}</span>
            </VAvatar>
            <div class="d-flex flex-column ms-3">
              <span
                class="d-block font-weight-medium text-high-emphasis text-truncate"
                >{{ item.fullName }}</span
              >
              <small>{{ item.post }}</small>
            </div>
          </div>
        </template>

        <!-- status -->
        <template #item.status="{ item }">
          <VChip
            :color="resolveStatusVariant(item.status).color"
            class="font-weight-medium"
            size="small"
          >
            {{ resolveStatusVariant(item.status).text }}
          </VChip>
        </template>
      </VDataTable>
    </VCard>
    <AddRolDialog v-model:is-dialog-visible="isAddRolDialogVisible" />
  </div>
</template>
