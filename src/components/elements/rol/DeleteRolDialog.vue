<script setup>
import { onMounted } from "vue";

const props = defineProps({
  isDialogVisible: {
    type: Boolean,
    required: true,
  },
  role_selected_deleted: {
    type: Object,
    required: true,
  },
});
const emit = defineEmits(["update:isDialogVisible", "addRole"]);
const dialogVisibleDeleted = (val) => {
  emit("update:isDialogVisible", val);
};

const role_selected_deleted = ref(null);

const success = ref(null);
const warning = ref(null);

const error_exists = ref(null);
const deleted = async () => {
  try {
    const response = await $api("/rol/" + role_selected_deleted.value.id, {
      method: "DELETE",
      onResponseError({ response }) {
        error_exists.value = response._data.error;
      },
    });

    const data = response;
    if (data.message == 200) {
      success.value = "Rol se ha eliminado correctamente";
      emit("addRole", true);
      dialogVisibleDeleted(false);
    }
  } catch (error) {
    warning.value = error.message || "Error al eliminar el rol";
  }
};
onMounted(() => {
  role_selected_deleted.value = props.role_selected_deleted;
});
</script>

<template>
  <VDialog
    :model-value="props.isDialogVisible"
    max-width="750"
    @update:model-value="dialogVisibleDeleted"
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
          <h4 class="text-h4 text-center mb-2" v-if="role_selected_deleted">
            Eliminar rol: {{ role_selected_deleted.id }}
          </h4>
          <p class="text-center">
            Eliminar un rol es una acción irreversible. Asegúrese de que desea
            continuar.
          </p>
        </div>
        <VAlert
          v-if="warning"
          type="warning"
          class="mt-4"
          dismissible
          @click:close="warning = null"
        >
          <strong>{{ warning }}</strong>
        </VAlert>
        <p v-if="role_selected_deleted" class="text-center">
          ¿Está seguro de que desea eliminar este rol
          {{ role_selected_deleted.name }}?
        </p>
        <VAlert type="error" class="my-2" v-if="error_exists">
          Error presentado: <strong>{{ error_exists }}</strong>
        </VAlert>
        <VAlert type="warning" class="my-2" v-if="success">
          <strong>{{ success }}</strong>
        </VAlert>
      </VCardText>
      <VCardActions class="d-flex justify-end">
        <VBtn color="error" @click="deleted"> Eliminar </VBtn>
      </VCardActions>
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
