<template>
  <div class="verzehrZettelForm">
    <div class="formHeader">
      <div class="formHeaderElement">Getränk:</div>
      <div class="formHeaderElement smallerColumn">Preis:</div>
      <div class="formHeaderElement">Personal:</div>
      <div class="formHeaderElement">Küche/Essen:</div>
      <div class="formHeaderElement">Chefs:</div>
      <div class="formHeaderElement">Verlust:</div>
      <div class="formHeaderElement">Werbung:</div>
      <div class="formHeaderElement">Geschl. Gesellschaft:</div>
    </div>

    <div class="drinkRow" v-for="drink in drinks" :key="drink.name">
      <span
        class="drinkColumn"
        :style="{
          borderLeft: `5px solid ${
            drink.categoryColor ? drink.categoryColor : 'grey'
          }`,
        }"
        >{{ drink.name }}</span
      >
      <span
        class="drinkColumn smallerColumn"
        :style="{ textAlign: 'right', paddingRight: '4px' }"
        >{{ drink.displayPrice }}</span
      >
      <span class="drinkColumn drinksByService">
        {{
          drink.wastedByService &&
          `${drink.wastedByService.count} (${drink.wastedByService.costSum}€)`
        }}
      </span>
      <span class="drinkColumn drinksByKitchen">
        {{
          drink.wastedByKitchen &&
          `${drink.wastedByKitchen.count} (${drink.wastedByKitchen.costSum}€)`
        }}
      </span>
      <span class="drinkColumn">{{ drink.wasted }}</span>
      <span class="drinkColumn">{{ drink.wasted }}</span>
      <span class="drinkColumn">{{ drink.wasted }}</span>
      <span class="drinkColumn">{{ drink.wasted }}</span>
    </div>

    <div class="inputs">
      <div class="workers">
        <div>
          <label for="numberOfWorkers">Anzahl Service-Arbeiter: </label>
          <input
            type="number"
            name="numberOfWorkers"
            v-model="numberOfWorkers_service"
            @change="handleNumberOfWorkersChange"
          />
        </div>
        <div>
          <label for="numberOfWorkers">Anzahl Küchen-Arbeiter: </label>
          <input
            type="number"
            name="numberOfWorkers"
            v-model="numberOfWorkers_kitchen"
            @change="handleNumberOfWorkersChange"
          />
          <div
            id="numberOfWorkersSubmit"
            :class="['button', 'noPrint', workersButtonDisabled && 'disabled']"
            @click="handleNumberOfWorkersSubmit"
          >
            Ok
          </div>
        </div>
      </div>

      <div class="salesAndHostel">
        <div>
          <label for="umsatz">Umsatz: </label>
          <input type="number" name="umsatz" v-model="sales" />
        </div>
        <div>
          <label for="hostel">Hostel: </label>
          <input type="number" name="hostel" v-model="hostel" />
          <div
            id="numberOfWorkersSubmit"
            :class="[
              'button',
              'noPrint',
              salesAndHostelButtonDisabled && 'disabled',
            ]"
          >
            Ok
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref } from "vue";

import { drinks } from "../assets/drinks.js";

export default {
  name: "VerzehrZettelForm",
  props: {
    msg: String,
  },
  setup() {
    const consumptionByEachWorker = 15;
    const consumptionByWorkersSplit = 3;

    const numberOfWorkers_service = ref(0);
    const numberOfWorkers_kitchen = ref(0);
    const sumOfConsumptionByWorkers_service = ref(null);
    const sumOfConsumptionByWorkers_kitchen = ref(null);

    const sales = ref(0);
    const hostel = ref(0);

    const workersButtonDisabled = ref(true);
    const salesAndHostelButtonDisabled = ref(true);
    const salesButtonDisabled = ref(true);

    const getRandomIndex = (min = 0, max = drinks.length - 1) => {
      const index = Math.round(Math.random() * (max - min) + min);
      return index;
    };

    const getRandomDrink = () => {
      const drink = drinks[getRandomIndex()];
      return drink;
    };

    const handleNumberOfWorkersChange = () => {
      workersButtonDisabled.value = false;
    };

    const handleNumberOfWorkersSubmit = () => {
      workersButtonDisabled.value = true;
      calcConsumptionByWorkers();
    };

    const calcConsumptionByWorkers = () => {
      sumOfConsumptionByWorkers_service.value =
        numberOfWorkers_service.value * consumptionByEachWorker;
      let calculatedConsumption_service = 0;

      const splitOneProduct_service = getRandomDrink();
      drinks[splitOneProduct_service.index] = {
        ...splitOneProduct_service,
        wastedByService: { count: 0, costSum: 0 },
      };
      const splitTwoProduct_service = getRandomDrink();
      drinks[splitTwoProduct_service.index] = {
        ...splitTwoProduct_service,
        wastedByService: { count: 0, costSum: 0 },
      };
      const splitThreeProduct_service = getRandomDrink();
      drinks[splitThreeProduct_service.index] = {
        ...splitThreeProduct_service,
        wastedByService: { count: 0, costSum: 0 },
      };
      // Service
      do {
        const randomNumber = getRandomIndex(0, consumptionByWorkersSplit);
        switch (randomNumber) {
          case 1:
            calculatedConsumption_service += splitOneProduct_service.price;
            drinks[splitOneProduct_service.index].wastedByService.costSum =
              Math.round(
                (drinks[
                  splitOneProduct_service.index
                ].wastedByService.costSum +=
                  splitOneProduct_service.price + Number.EPSILON) * 100
              ) / 100;
            drinks[splitOneProduct_service.index].wastedByService.count += 1;
            break;
          case 2:
            calculatedConsumption_service += splitTwoProduct_service.price;
            drinks[splitTwoProduct_service.index].wastedByService.costSum =
              Math.round(
                (drinks[
                  splitTwoProduct_service.index
                ].wastedByService.costSum +=
                  splitTwoProduct_service.price + Number.EPSILON) * 100
              ) / 100;
            drinks[splitTwoProduct_service.index].wastedByService.count += 1;
            break;
          case 3:
            calculatedConsumption_service += splitThreeProduct_service.price;
            drinks[splitThreeProduct_service.index].wastedByService.costSum =
              Math.round(
                (drinks[
                  splitThreeProduct_service.index
                ].wastedByService.costSum +=
                  splitThreeProduct_service.price + Number.EPSILON) * 100
              ) / 100;
            drinks[splitThreeProduct_service.index].wastedByService.count += 1;
            break;
        }
      } while (
        calculatedConsumption_service < sumOfConsumptionByWorkers_service.value
      );
      // Küche
      sumOfConsumptionByWorkers_kitchen.value =
        numberOfWorkers_kitchen.value * consumptionByEachWorker;
      let calculatedConsumption_kitchen = 0;

      const splitOneProduct_kitchen = getRandomDrink();
      drinks[splitOneProduct_kitchen.index] = {
        ...splitOneProduct_kitchen,
        wastedByKitchen: { count: 0, costSum: 0 },
      };
      const splitTwoProduct_kitchen = getRandomDrink();
      drinks[splitTwoProduct_kitchen.index] = {
        ...splitTwoProduct_kitchen,
        wastedByKitchen: { count: 0, costSum: 0 },
      };
      const splitThreeProduct_kitchen = getRandomDrink();
      drinks[splitThreeProduct_kitchen.index] = {
        ...splitThreeProduct_kitchen,
        wastedByKitchen: { count: 0, costSum: 0 },
      };

      do {
        const randomNumber = getRandomIndex(0, consumptionByWorkersSplit);
        switch (randomNumber) {
          case 1:
            calculatedConsumption_kitchen += splitOneProduct_kitchen.price;
            drinks[splitOneProduct_kitchen.index].wastedByKitchen.costSum =
              Math.round(
                (drinks[
                  splitOneProduct_kitchen.index
                ].wastedByKitchen.costSum +=
                  splitOneProduct_kitchen.price + Number.EPSILON) * 100
              ) / 100;
            drinks[splitOneProduct_kitchen.index].wastedByKitchen.count += 1;
            break;
          case 2:
            calculatedConsumption_kitchen += splitTwoProduct_kitchen.price;
            drinks[splitTwoProduct_kitchen.index].wastedByKitchen.costSum =
              Math.round(
                (drinks[
                  splitTwoProduct_kitchen.index
                ].wastedByKitchen.costSum +=
                  splitTwoProduct_kitchen.price + Number.EPSILON) * 100
              ) / 100;
            drinks[splitTwoProduct_kitchen.index].wastedByKitchen.count += 1;
            break;
          case 3:
            calculatedConsumption_kitchen += splitThreeProduct_kitchen.price;
            drinks[splitThreeProduct_kitchen.index].wastedByKitchen.costSum =
              Math.round(
                (drinks[
                  splitThreeProduct_kitchen.index
                ].wastedByKitchen.costSum +=
                  splitThreeProduct_kitchen.price + Number.EPSILON) * 100
              ) / 100;
            drinks[splitThreeProduct_kitchen.index].wastedByKitchen.count += 1;
            break;
        }
      } while (
        calculatedConsumption_kitchen < sumOfConsumptionByWorkers_kitchen.value
      );
      console.log(sumOfConsumptionByWorkers_kitchen);
    };

    return {
      sales,
      hostel,
      numberOfWorkers_service,
      numberOfWorkers_kitchen,
      drinks,
      workersButtonDisabled,
      salesButtonDisabled,

      handleNumberOfWorkersChange,
      handleNumberOfWorkersSubmit,
    };
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
button {
  margin-left: 5px;
  cursor: pointer;
}

label {
  display: block;
}

.verzehrZettelForm {
  margin: 1rem 0;
}
.formHeader {
  text-align: center;
}
.drinkRow,
.formHeader {
  display: flex;
}
.formHeaderElement,
.drinkColumn {
  flex: 2;
}
.drinkColumn {
  border: 1px solid black;
  text-align: center;
}
.smallerColumn {
  flex: 1;
}

.inputs {
  margin-top: 2rem;
  display: flex;
  justify-content: space-between;
  gap: 2rem;
}

.workers {
  gap: 10px;
  padding: 6px;
}

.salesAndHostel {
  gap: 10px;
  padding: 6px;
}
.workers,
.salesAndHostel {
  display: flex;
  justify-content: center;
  flex: 1;
  border: 2px solid cadetblue;
}

.button {
  display: inline;
  margin-left: 5px;
  cursor: pointer;
  font-size: 12px;
  padding: 6px 4px;
  border-radius: 4px;
  background: #2020cc;
  color: white;
}
.button.disabled {
  background: lightgrey;
  color: grey;
}

@media print {
  .drinkColumn {
    border-left: 1px solid black !important;
  }
  .salesAndHostel,
  .workers {
    border: none !important;
  }
}
</style>
