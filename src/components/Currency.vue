
<template>
  <div class="tables">
    <v-card class="fullWidth">
      <v-card-title>
        Текущая валюта
        <v-spacer></v-spacer>
        <v-btn
          color="primary"
          @click="calculate"
          :disabled="currency.length == 0"
        >
          Рассчитать</v-btn
        >
      </v-card-title>
      <v-data-table
        hide-default-footer
        :loading="loading"
        loading-text="Загрузка... Пожалуста подождите"
        :headers="headers"
        :items="currency"
      >
        <template v-slot:[`item.Date`]="{ item }">
          {{
            Intl.DateTimeFormat("ru-RU", {
              day: "numeric",
              month: "long",
              year: "numeric",
            }).format(new Date(item.Date))
          }}
        </template>
        <template v-slot:[`item.actions`]="{ item }">
          <v-icon color="red" @click="deleteItem(item)"> mdi-delete </v-icon>
        </template>
        <template v-slot:no-data> Добавте данные кнопочкой ADD </template>
      </v-data-table>
    </v-card>

    <v-card>
      <v-menu v-model="menu1" :close-on-content-click="false">
        <template v-slot:activator="{ on, attrs }">
          <v-text-field
            color="green"
            width="200px"
            small
            :value="startDate"
            label="Дата начала"
            readonly
            v-bind="attrs"
            v-on="on"
            @click:clear="date = null"
            class="px-3"
          ></v-text-field>
        </template>
        <v-date-picker
          header-color="green"
          v-model="dateStart"
          @change="menu1 = false"
          locale="ru-ru"
        ></v-date-picker> </v-menu
    ></v-card>

    <v-card>
      <v-menu v-model="menu2" :close-on-content-click="false">
        <template v-slot:activator="{ on, attrs }">
          <v-text-field
            :value="endDate"
            label="Дата окончания"
            readonly
            v-bind="attrs"
            v-on="on"
            @click:clear="date = null"
            class="px-3"
          ></v-text-field>
        </template>
        <v-date-picker
          v-model="dateEnd"
          @change="menu2 = false"
          locale="ru-ru"
        ></v-date-picker>
      </v-menu>
    </v-card>

    <v-card class="fullWidth"><Chart /></v-card>
  </div>
</template>

<script>
import { mapActions } from "vuex";
import Chart from "./Chart.vue";
import moment from "moment";

export default {
  name: "ChartCurrency",
  components: {
    Chart,
  },
  data() {
    return {
      dateStart: "",
      dateEnd: "",
      menu1: false,
      menu2: false,
      initCurrency: ["USD"],
      loading: false,
      headers: [
        {
          text: "Дата",
          align: "start",
          sortable: false,
          value: "Date",
        },
        { text: "Код валюты", value: "Cur_Abbreviation" },
        { text: "Название", value: "Cur_Name" },
        { text: "Курс", value: "Cur_OfficialRate" },
        { text: "", value: "actions", sortable: false },
      ],
    };
  },
  async mounted() {
    this.loading = true;
    await this.getAllCurrency();
    for (let abbreviation of this.initCurrency) {
      const findItem = this.allCurrency.find(
        (item) => item.Cur_Abbreviation == abbreviation
      );
      if (findItem) {
        this.isDublicate(findItem); // проверка на дубликат перед добавлением
      }
    }
    this.loading = false;
  },
  computed: {
    currency() {
      return this.$store.getters.get("currency");
    },
    allCurrency() {
      return this.$store.getters.get("allCurrency");
    },
    startDate() {
      return this.dateStart ? moment(this.dateStart).format("YYYY-MM-DD") : "";
    },
    endDate() {
      return this.dateEnd ? moment(this.dateEnd).format("YYYY-MM-DD") : "";
    },
  },
  methods: {
    ...mapActions(["getAllCurrency", "isDublicate", "getTimePeriodCurrency"]),
    deleteItem(itemDelete) {
      const position = this.currency.findIndex((item) => item == itemDelete);
      if (position || position == 0) {
        this.currency.splice(position, 1);

        this.$store.commit("set", {
          name: "currency",
          value: this.currency,
        });

        //обнуление графика
        this.$store.commit("set", {
          name: "forPeriod",
          value: [],
        });
      }
    },
    calculate() {
      if (!this.dateStart || !this.dateEnd) {
        this.$store.commit("set", {
          name: "message",
          value: {
            color: "error",
            text: "Необходимо выбрать даты начала и конца ",
            run: true,
          },
        });
        return;
      }
      if (this.dateStart > this.dateEnd) {
        this.$store.commit("set", {
          name: "message",
          value: {
            color: "error",
            text: "Дату начала больше даты конца",
            run: true,
          },
        });
        return;
      }

      this.getTimePeriodCurrency({
        cur_id: this?.currency[0]?.Cur_ID,
        startDate: this.startDate,
        endDate: this.endDate,
      });
    },
  },
};
</script>

<style scoped>
.tables {
  display: grid;
  grid-template-columns: auto auto;
  grid-template-rows: auto auto 1fr;
  grid-row-gap: 20px;
  grid-column-gap: 10px;
  overflow: hidden;
}
.fullWidth {
  grid-column: -1/1;
}
</style>
