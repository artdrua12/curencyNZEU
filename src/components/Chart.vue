<template>
  <div>
    <line-graf :options="chartOptions" :data="chartData" />
  </div>
</template>

<script>
import moment from "moment";
import { Line as LineGraf } from "vue-chartjs";
import {
  Chart as ChartJS,
  Title,
  Tooltip,
  Legend,
  BarElement,
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
} from "chart.js";

ChartJS.register(
  Title,
  Tooltip,
  Legend,
  BarElement,
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement
);

export default {
  name: "BarChart",
  components: { LineGraf },
  data() {
    return {
      chartOptions: {
        responsive: true,
      },
    };
  },

  computed: {
    forPeriod() {
      return this.$store.getters.get("forPeriod");
    },
    CurName() {
      return this.$store.getters.get("currency")[0]?.Cur_Name || 0;
    },
    dataArray() {
      return this.forPeriod.map((item) => item.Cur_OfficialRate);
    },
    labels() {
      return this.forPeriod.map((item) =>
        moment(item.Date).format("DD-MM-YYYY")
      );
    },
    chartData() {
      return {
        labels: this.labels,
        datasets: [{ label: this.CurName, data: this.dataArray }],
      };
    },
  },
};
</script>