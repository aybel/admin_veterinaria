<script setup>
const props = defineProps({
  isDialogVisible: {
    type: Boolean,
    required: true,
  },
});
const emit = defineEmits(["update:isDialogVisible"]);
const dialogVisibleUpdate = (val) => {
  emit("update:isDialogVisible", val);
};
const LIST_PERMISSIONS = PERMISOS;
const rolName = ref("");
const arrPermissions = ref([]);
const warning = ref(null);
const addPerm = (permiso) => {
  if (arrPermissions.value.includes(permiso)) {
    arrPermissions.value = arrPermissions.value.filter((p) => p !== permiso);
  } else {
    arrPermissions.value.push(permiso);
  }
};

const store = async () => {
  warning.value = null;
  if (rolName.value.trim() === "") {
    warning.value = "El nombre del rol es obligatorio.";
    return;
  }
  if (arrPermissions.value.length === 0) {
    warning.value = "Debe seleccionar al menos un permiso.";
    return;
  }
};
</script>

<template>
  <VDialog
    :model-value="props.isDialogVisible"
    max-width="750"
    @update:model-value="dialogVisibleUpdate"
  >
    <VCard class="refer-and-earn-dialog pa-3 pa-sm-11">
      <!-- 👉 dialog close btn -->
      <DialogCloseBtn
        variant="text"
        size="default"
        @click="emit('update:isDialogVisible', false)"
      />

      <VCardText class="pa-5">
        <div class="mb-6">
          <h4 class="text-h4 text-center mb-2">Agregar rol</h4>
            <p class="text-center">
                Agrega un nuevo rol y asigna los permisos correspondientes.
            </p>
        </div>
        <VTextField
          label="Rol:"
          placeholder="Ingrese el nombre del rol"
          required
          v-model="rolName"
        />
        <VAlert
          v-if="warning"
          type="warning"
          class="mt-4"
          dismissible
          @click:close="warning = null"
        >
         <strong>{{ warning }}</strong>
        </VAlert>
      </VCardText>
      <VCardActions class="d-flex justify-end">
        <!-- <VBtn variant="text" @click="emit('update:isDialogVisible', false)">
          Cancelar
        </VBtn> -->
        <VBtn color="primary" @click="store">
          Guardar
        </VBtn>
      </VCardActions>
      <VCardText class="pa-5">
        <VTable>
          <thead>
            <tr>
              <th class="text-uppercase">Módulo</th>
              <th class="text-uppercase">Permisos</th>
            </tr>
          </thead>

          <tbody>
            <tr v-for="item in LIST_PERMISSIONS" :key="item.permisos">
              <td>
                {{ item.name }}
              </td>
              <td>
                <VCheckbox
                  v-for="perm in item.permisos"
                  :key="perm.permiso"
                  :label="perm.name"
                  class="me-2"
                  @click="addPerm(perm.permiso)"
                />
              </td>
            </tr>
          </tbody>
        </VTable>
      </VCardText>
    </VCard>
  </VDialog>
</template>

<style lang="scss">
.refer-link-input {
  .v-field--appended {
    padding-inline-end: 0;
  }

  .v-field__append-inner {
    padding-block-start: 0.125rem;
  }
}
</style>
