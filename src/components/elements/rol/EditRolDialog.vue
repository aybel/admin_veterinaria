<script setup>
import { onMounted } from "vue";

const props = defineProps({
  isDialogVisible: {
    type: Boolean,
    required: true,
  },
  role_selected: {
    type: Object,
    required: true,
  },
});
const emit = defineEmits(["update:isDialogVisible", "addRole"]);
const dialogVisibleUpdate = (val) => {
  emit("update:isDialogVisible", val);
};
const LIST_PERMISSIONS = PERMISOS;
const role_selected = ref(null);
const rolName = ref("");
const arrPermissions = ref([]);
const success = ref(null);
const warning = ref(null);
const addPerm = (permiso) => {
  if (arrPermissions.value.includes(permiso)) {
    arrPermissions.value = arrPermissions.value.filter((p) => p !== permiso);
  } else {
    arrPermissions.value.push(permiso);
  }
};
const error_exists = ref(null);
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
  try {
    const response = await $api("/rol/" + role_selected.value.id, {
      method: "PATCH",
      body: JSON.stringify({
        name: rolName.value,
        permissions: arrPermissions.value,
      }),
      onResponseError({ response }) {
        error_exists.value = response._data.error;
      },
    });

    const data = response;
    if (data.message == 403) {
      warning.value = data.message_text;
      return;
    } else if (data.message == 200) {
      success.value = "Rol editado correctamente";
      emit("addRole", true);
      setTimeout(() => {
        dialogVisibleUpdate(false);
      }, 1500);
    }
  } catch (error) {
    warning.value = error.message || "Error al editar el rol";
  }
};
onMounted(() => {
  role_selected.value = props.role_selected;
  rolName.value = role_selected.value ? role_selected.value.name : "";
  arrPermissions.value = role_selected.value.permissions_pluck || [];
});
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
          <h4 class="text-h4 text-center mb-2" v-if="role_selected">
            Editar rol: {{ role_selected.id }}
          </h4>
          <p class="text-center">
            Edita un rol existente y asigna los permisos correspondientes.
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
        <VAlert type="error" class="my-2" v-if="error_exists">
          Error presentado: <strong>{{ error_exists }}</strong>
        </VAlert>
        <VAlert type="success" class="my-2" v-if="success">
          <strong>{{ success }}</strong>
        </VAlert>
      </VCardText>
      <VCardActions class="d-flex justify-end">
        <!-- <VBtn variant="text" @click="emit('update:isDialogVisible', false)">
          Cancelar
        </VBtn> -->
        <VBtn color="primary" @click="store"> Editar </VBtn>
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
                  v-model="arrPermissions"
                  v-for="perm in item.permisos"
                  :value="perm.permiso"
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
