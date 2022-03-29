<template>
  <div class="container" id="app">
    <search-group
      :full-name="filters.fullName"
      :birth-date="filters.birthDate"
      @search="onSearch"
      @reset="onReset"
    />
    <div class="mt-3" v-if="showRemoveBtn">
      <b-button-group>
        <b-button variant="danger" @click="onRemove">Удалить</b-button>
        <b-button
          variant="warning"
          :disabled="disableBtnEdit"
          v-b-modal.modal-client
        >
          Редактировать
        </b-button>
      </b-button-group>
    </div>
    <modal-create-user :active-user="selectedRows[0]" @save-user="onSaveUser" />
    <b-button
      v-show="!selectedRows.length"
      variant="primary"
      v-b-modal.modal-client
    >
      Добавить клиента
    </b-button>
    <b-table
      id="persons-table"
      striped
      selectable
      :per-page="perPage"
      :current-page="filters.page"
      :items="searchResult"
      :fields="fields"
      :sort-compare="sortColumn"
      :sort-by="filters.sortBy"
      :sort-desc="isSortDesc"
      @sort-changed="
        ({ sortBy, sortDesc }) => {
          this.setQueryParams('sortBy', filters.sortBy);
          this.setQueryParams('sortDesc', filters.sortDesc);
        }
      "
      @row-selected="(items) => (selectedRows = items)"
    ></b-table>
    <b-pagination
      v-show="rows > 1"
      v-model="filters.page"
      aria-controls="persons-table"
      :total-rows="rows"
      :per-page="perPage"
      @input="(page) => setQueryParams('page', filters.page)"
    ></b-pagination>
  </div>
</template>

<script>
import { faker } from "@faker-js/faker";

import SearchGroup from "./components/SearchGroup.vue";
import ModalCreateUser from "./components/ModalCreateUser.vue";

const PERSONS_COUNT = 50;
const ELEMENS_ON_THE_PAGE = 10;

const FIELDS = [
  {
    key: "id",
    sortable: true,
  },
  {
    key: "fullName",
    label: "ФИО",
    sortable: true,
  },
  {
    key: "birthDate",
    label: "День рожденья",
    sortable: true,
    formatter: (value) => value.toLocaleDateString("ru-ru"),
  },
];

export default {
  components: { SearchGroup, ModalCreateUser },
  name: "App",
  data() {
    return {
      persons: (() => {
        const persons = [];
        for (let i = 0; i < PERSONS_COUNT; i++) {
          persons.push({
            id: i,
            fullName: faker.fake(
              "{{name.lastName}}, {{name.firstName}} {{name.suffix}}"
            ),
            birthDate: faker.date.past(),
          });
        }
        return persons;
      })(),
      selectedRows: [],
      filters: {
        sortBy: "id",
        sortDesc: false,
        page: 1,
        fullName: "",
        birthDate: "",
      },
      queryParams: new URLSearchParams(window.location.search),
    };
  },
  computed: {
    searchResult() {
      return this.persons.filter((el) => {
        return (
          el?.fullName
            ?.toLowerCase()
            ?.includes(this.filters.fullName?.toLowerCase()) &&
          el?.birthDate
            ?.toLocaleDateString("ru-ru")
            ?.includes(this.filters.birthDate)
        );
      });
    },
    showRemoveBtn() {
      return this.selectedRows?.length;
    },
    isSortDesc() {
      return this.filters.sortDesc == "true";
    },
    fields() {
      return FIELDS;
    },
    rows() {
      return this.searchResult.length;
    },
    perPage() {
      return ELEMENS_ON_THE_PAGE;
    },
    disableBtnEdit() {
      return this.selectedRows.length !== 1;
    },
  },
  created() {
    Array.from(this.queryParams.keys()).forEach((element) => {
      this.filters[element] = this.queryParams.get(element);
    });
    this.onSearch(this.filters);
  },
  methods: {
    onSearch({ fullName, birthDate }) {
      const formatedDate =
        birthDate && new Date(birthDate)?.toLocaleDateString("ru-ru");
      this.setQueryParams("fullName", fullName);
      this.setQueryParams("birthDate", formatedDate);
      this.filters.fullName = fullName;
      this.filters.birthDate = formatedDate;
    },
    onRemove() {
      this.$bvModal
        .msgBoxConfirm("Удалить выбранных сотрудников?")
        .then((value) => {
          if (!value) return;
          const removeItemsId = this.selectedRows.map((el) => el.id);
          this.persons = this.persons.filter((el) => {
            return !removeItemsId.includes(el.id);
          });
        })
        .catch((err) => {
          console.log(err);
        });
    },
    onReset() {
      this.filters.birthDate = "";
      this.filters.fullName = "";
      history.replaceState(null, null, "/");
    },
    onSaveUser(user) {
      if (user.id === null) {
        this.persons.unshift({
          ...user,
          id:
            Math.max.apply(
              Math,
              this.persons.map((o) => {
                return o.id;
              })
            ) + 1,
          birthDate: new Date(user.birthDate),
        });
      } else {
        this.persons = this.persons.map((el) => {
          if (el.id === user.id) {
            el = {
              ...user,
              birthDate: new Date(user.birthDate),
            };
          }
          return el;
        });
      }
    },
    setQueryParams(key, value) {
      if (value) {
        this.queryParams.set(key, value);
      } else {
        this.queryParams.delete(key);
      }
      const queries = this.queryParams.toString();
      history.replaceState(null, null, queries ? "?" + queries : "/");
    },
    sortColumn(a, b, key) {
      if (key !== "birthDate") return false;
      return a[key].getTime() - b[key].getTime();
    },
  },
};
</script>

<style>
</style>
