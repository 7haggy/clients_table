<template>
  <b-modal id="modal-client" size="lg" hide-header hide-footer>
    <h3>{{ title }}</h3>
    <b-form @submit.prevent="save">
      <b-input-group class="mt-3 mb-3">
        <b-form-input
          v-model.trim="user.fullName"
          required
          placeholder="ФИО"
          :class="{ 'outline-error': !user.fullName }"
        >
        </b-form-input>
        <b-form-datepicker
          v-model="user.birthDate"
          placeholder="Дата рождения"
          locale="ru-ru"
          :max="new Date()"
        ></b-form-datepicker>
      </b-input-group>
      <b-button-group>
        <b-button variant="primary" type="submit">Сохранить</b-button>
        <b-button variant="danger" @click="$bvModal.hide('modal-client')">
          Отменить
        </b-button>
      </b-button-group>
    </b-form>
  </b-modal>
</template>
<script>
export default {
  name: "ModalCreateUser",
  props: {
    activeUser: {
      type: Object,
      default: () => ({
        id: null,
        fullName: "",
        birthDate: new Date(),
      }),
    },
  },
  data() {
    return {
      user: { ...this.activeUser },
    };
  },
  watch: {
    activeUser() {
      this.user = { ...this.activeUser };
    },
  },
  computed: {
    title() {
      return this.activeUser ? "Редактировать клиента" : "Добавить клиента";
    },
  },
  methods: {
    save() {
      this.$emit("save-user", this.user);
      this.$bvModal.hide("modal-client");
      this.user = {
        id: null,
        fullName: "",
        birthDate: "",
      };
    },
  },
};
</script>