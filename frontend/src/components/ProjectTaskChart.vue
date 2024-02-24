<template>
  <div>
    <ApexChart
      v-if="ready"
      :options="chartOptions"
      :series="raw(series)"
    ></ApexChart>
  </div>
</template>

<script>
import ApexChart from "vue3-apexcharts";
import { toRaw } from "vue";

export default {
  name: "ProjectTaskChart",
  components: { ApexChart },
  props: ["project", "forPerson", "user"],
  data() {
    return {
      ready: false,
      chartOptions: {
        chart: {
          height: 250,
          type: "rangeBar",
        },
        plotOptions: {
          bar: {
            horizontal: true,
          },
        },
        xaxis: {
          type: "datetime",
        },
      },
      series: [{ data: [] }],
    };
  },
  mounted() {
    let filter = this.forPerson ? "worker=" : "project=";
    fetch("/task?" + filter + this.project + "&limit=10", {
      method: "GET",
    })
      .then((res) => {
        res
          .json()
          .then((data) => {
            this.series[0].data = data.map((task) => ({
              x: task.name,
              y: [
                new Date(task.startDate).getTime(),
                task.endDate
                  ? new Date(task.endDate).getTime()
                  : new Date().getTime(),
              ],
              fillColor: task.endDate ? "#00E396" : "#FEB019",
            }));
            this.ready = true;
          })
          .catch((err) => console.error(err.message));
      })
      .catch((err) => console.error(err.message));
  },
  methods: {
    raw(series) {
      return toRaw(series);
    },
  },
};
</script>
