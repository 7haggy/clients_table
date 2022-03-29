<template>
  <b-form @submit.prevent="onSubmit" @reset="onReset">
    <b-input-group class="mt-3 mb-3">
      <b-form-input v-model.trim="fullNameVal" placeholder="ФИО"> </b-form-input>
      <b-form-datepicker v-model="birthDateVal" :max="new Date()" placeholder="Дата рожденья" locale="ru-ru"></b-form-datepicker>
      <b-button variant="outline-primary" type="submit">Поиск</b-button>
      <b-button variant="outline-danger" type="reset">Сброс</b-button>
    </b-input-group>
  </b-form>
</template>
<script>
export default {
  name: "SearchGroup",
  props: {
    fullName: {
      type: String,
      default: "",
    },
    birthDate: {
      type: String,
      default: "",
    },
  },
  data() {
    return {
      fullNameVal: this.fullName,
      birthDateVal: this.birthDate.split('.').reverse().join('-'),
    };
  },
  methods: {
    onSubmit() {
      const { fullNameVal, birthDateVal } = this;
      this.$emit("search", { fullName: fullNameVal, birthDate: birthDateVal });
    },
    onReset() {
      this.fullNameVal = "";
      this.birthDateVal = "";
      this.$emit("reset");
    },
  },
};
</script>