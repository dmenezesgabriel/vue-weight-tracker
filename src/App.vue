<script setup lang="ts">
import { type Ref, ref, shallowRef, computed, watch, nextTick } from "vue";
import Chart from "chart.js/auto";

const weights = ref([]);
const weightChartElement = ref(null);
const weightChart = shallowRef(null);
const weightInput = ref(60.0);

const currentWeight = computed(() => {
  return [...weights.value].sort((a, b) => b.date - a.date)[0] || { weight: 0 };
});

const addWeight = () => {
  weights.value.push({
    weight: weightInput.value,
    date: new Date().getTime(),
  });
};

watch(
  weights,
  (newWeights) => {
    const ws = [...newWeights];

    if (weightChart.value) {
      weightChart.value.data.labels = ws
        .sort((a, b) => a.date - b.date)
        .map((w) => new Date(w.date).toLocaleDateString())
        .slice(-7);

      weightChart.value.data.datasets[0].data = ws
        .sort((a, b) => a.date - b.date)
        .map((w) => w.weight)
        .slice(-7);

      weightChart.value.update();

      return;
    }
    // Wait till dom is rendered
    nextTick(() => {
      weightChart.value = new Chart(weightChartElement.value.getContext("2d"), {
        type: "line",
        data: {
          labels: ws
            .sort((a, b) => a.date - b.date)
            .map((w) => new Date(w.date).toLocaleDateString()),
          datasets: [
            {
              label: "weight",
              data: ws.sort((a, b) => a.date - b.date).map((w) => w.weight),
              backgroundColor: "rgba(255, 105, 180, 0.2)",
              borderColor: "rgb(255, 105, 180)",
              borderWidth: 1,
              fill: true,
            },
          ],
        },
        options: { responsive: true, maintainAspectRatio: false },
      });
    });
  },
  { deep: true }
);
</script>

<template>
  <main>
    <h1>Weight Tracker</h1>

    <div class="current-box">
      <div class="current">
        <span>{{ currentWeight.weight }}</span>
        <small>Current weight (kg)</small>
      </div>
    </div>

    <form @submit.prevent="addWeight">
      <input type="number" step="0.1" v-model="weightInput" />
      <input type="submit" value="Add Weight" />
    </form>

    <div v-if="weights && weights.length > 0">
      <h2>Last 7 days</h2>
      <div class="canvas-box">
        <canvas ref="weightChartElement"></canvas>
      </div>
      <div class="weight-history">
        <h2>Weight History</h2>
        <ul>
          <li v-for="weight in weights">
            <span>{{ weight.weight }}kg</span>
            <small>{{ new Date(weight.date).toLocaleDateString() }}</small>
          </li>
        </ul>
      </div>
    </div>
  </main>
</template>

<style lang="scss">
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: monospace;
}

body {
  background-color: #eee;
}

main {
  display: flex;
  flex-direction: column;
  justify-content: center;
  padding: 1.5rem;

  h1 {
    font-size: 2rem;
    text-align: center;
    margin-bottom: 2rem;
  }

  h2 {
    margin-bottom: 1rem;
    color: #888;
    font-weight: 400;
  }

  .current-box {
    margin: 1rem auto;

    .current {
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;

      width: 200px;
      height: 200px;

      text-align: center;
      background-color: white;
      border-radius: 50%;
      box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
      border: 5px solid rgb(255, 105, 180);

      margin: auto 2rem;
      span {
        display: block;
        font-size: 2rem;
        font-weight: bold;
        margin-bottom: rem;
      }

      small {
        color: #888;
        font-style: italic;
      }
    }
  }

  form {
    display: flex;
    margin-bottom: 2rem;
    border: 1px solid #aaa;
    border-radius: 0.5rem;
    overflow: hidden;
    transition: 200ms linear;

    &:focus-within,
    &:hover {
      border-color: hotpink;
      border-width: 2px;
    }

    input[type="number"] {
      appearance: none;
      outline: none;
      border: none;
      background-color: white;
      flex: 1 1 0%;
      padding: 1rem 1.5rem;
      font-size: 1.25rem;
    }

    input[type="submit"] {
      appearance: none;
      outline: none;
      border: none;
      cursor: pointer;
      background-color: hotpink;

      padding: 0.5rem 1rem;

      color: white;
      font-size: 1.25rem;
      font-weight: 700;
      transition: 200ms linear;
      border-left: 3px solid transparent;

      &:hover {
        background-color: white;
        color: hotpink;
        border-left-color: hotpink;
      }
    }
  }
  .canvas-box {
    width: 100%;
    max-width: 720px;
    background-color: white;
    padding: 1rem;
    border-radius: 0.5rem;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
    margin-bottom: 2rem;
  }

  .weight-history {
    ul {
      list-style: none;
      padding: 0;
      margin: 0;

      li {
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding: 0.5rem;
        cursor: pointer;

        &:nth-child(even) {
          background-color: #dfdfdf;
        }

        &:hover {
          background-color: #f8f8f8;
        }

        &:last-of-type {
          border-bottom: none;
        }

        span {
          display: block;
          font-size: 1.25rem;
          font-weight: 700;
          margin-right: 1rem;
        }

        small {
          color: #888;
          font-style: italic;
        }
      }
    }
  }
}
</style>
