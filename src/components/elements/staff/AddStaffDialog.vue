<script setup>
import { set } from "@vueuse/core";
import { ref } from "vue";
import { onMounted, watch } from "vue";
import { VFileInput } from "vuetify/components";

const error_exists = ref(null);
const warning = ref(null);
const success = ref(null);
const dialogVisibleUpdate = ref(false);
const FILE_AVATAR = ref(null);
const IMAGEN_PREVIZUALIZA = ref(null);
const props = defineProps({
  isDialogVisible: {
    type: Boolean,
    default: false,
  },
  roles: {
    type: Object,
    required: true,
    default: () => ({}),
  },
});
const form = ref({
  name: null,
  surname: null,
  phone: null,
  type_document: "INE",
  n_document: null,
  birthday: null,
  role_id: null,
  email: null,
  password: null,
  desgination: null,
});
const roles = ref([]);

const emit = defineEmits(["update:isDialogVisible", "addStaff"]);

const store = async () => {
  warning.value = null;
  success.value = null;
  try {
    const resp = await $api("/staff", {
      method: "POST",
      body: form,
      onResponseError({ response }) {
        console.log(response._data.error);
        error_exists.value = response._data.error;
      },
    });
    if (resp.message == 403) {
      warning.value = resp.message_text;
      return;
    } else {
      success.value = "Usuario creado correctamente.";
      setTimeout(() => {
        success.value = null;
        warning.value = null;
        emit("update:isDialogVisible", false);
        emit("addStaff", true);
      }, 1500);
    }
  } catch (error) {
    console.log("Error al validar los campos:", error);
    error_exists.value = error;
    return;
  }
};

const loadFile = ($event) => {
  if ($event.target.files[0].type.indexOf("image") < 0) {
    FILE_AVATAR.value = null;
    IMAGEN_PREVIZUALIZA.value = null;
    warning.value = "Solo se permiten archivos de imagen.";
    return;
  }
  warning.value = "";
  FILE_AVATAR.value = $event.target.files[0];
  let reader = new FileReader();
  reader.readAsDataURL(FILE_AVATAR.value);
  reader.onloadend = () => (IMAGEN_PREVIZUALIZA.value = reader.result);
};

onMounted(() => {
  roles.value = props.roles;
  console.log("Roles loaded:", roles.value);
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
          <h4 class="text-h4 text-center mb-2">Agregar usuario</h4>
          <p class="text-center">Agrega un nuevo usuario.</p>
        </div>

        <VRow>
          <VCol cols="12">
            <VTextField
              label="Nombre:"
              placeholder="Ingrese el nombre del usuario"
              required
              v-model="form.name"
            />
          </VCol>
          <VCol cols="12">
            <VTextField
              label="Apellido:"
              placeholder="Ingrese el apellido del usuario"
              required
              v-model="form.surname"
            />
          </VCol>
          <VCol cols="12">
            <VTextField
              label="Teléfono:"
              type="number"
              placeholder="Ingrese el teléfono del usuario"
              required
              v-model="form.phone"
            />
          </VCol>
          <VCol cols="12">
            <VSelect
              label="Tipo de documento:"
              :items="['INE', 'Pasaporte', 'Licencia']"
              v-model="form.type_document"
              required
            />
          </VCol>
          <VCol cols="12">
            <VTextField
              label="Número de documento:"
              placeholder="Ingrese el número de documento del usuario"
              required
              type="number"
              v-model="form.n_document"
            />
          </VCol>
          <VCol cols="12">
            <app-date-time-picker
              v-model="form.birthday"
              :format="'DD-MM-YYYY'"
              class="mt-2"
              lang="es"
              label="Fecha de nacimiento:"
              placeholder="Seleccione la fecha de nacimiento del usuario"
            />
          </VCol>
          <VCol cols="12">
            <VSelect
              label="Rol:"
              :items="roles"
              v-model="form.role_id"
              required
            />
          </VCol>
          <VCol cols="12">
            <VTextField
              label="Correo electrónico:"
              placeholder="Ingrese el correo electrónico del usuario"
              type="email"
              required
              v-model="form.email"
            />
          </VCol>
          <VCol cols="12">
            <VTextField
              label="Contraseña:"
              placeholder="Ingrese la contraseña del usuario"
              type="password"
              required
              v-model="form.password"
            />
          </VCol>
          <VCol cols="12">
            <VTextField
              label="Designación:"
              placeholder="Ingrese la designación del usuario"
              v-model="form.designation"
            />
          </VCol>
          <VCol cols="12">
            <VFileInput
              label="Imagen de avatar del usuario:"
              placeholder="Seleccione un archivo"
              v-model="form.file"
              @change="loadFile($event)"
            />
          </VCol>
          <VCol cols="12">
            <VImg
              v-if="IMAGEN_PREVIZUALIZA"
              :src="IMAGEN_PREVIZUALIZA"
              alt="Previsualización de imagen"
              class="mt-2 text-center"
              style="max-width: 50%; height: auto"
            />
            <span v-else class="text-sm text-secondary">
              No hay imagen seleccionada
            </span>
          </VCol>
        </VRow>
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
        <VBtn color="primary" @click="store"> Guardar </VBtn>
      </VCardActions>
    </VCard>
  </VDialog>
</template>
