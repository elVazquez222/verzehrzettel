s<!-- eslint-disable prettier/prettier -->
<template>
  <div class="verzehrZettelForm" :id="shadowKey">
    {{drinksList}}
    <div class="formHeader">
      <div class="formHeaderElement">Getränk:</div>
      <div class="formHeaderElement smallerColumn">Preis:</div>
      <div class="formHeaderElement">Personal:</div>
      <div class="formHeaderElement">Küche/Essen:</div>
      <div class="formHeaderElement">Chefs:</div>
      <div class="formHeaderElement">Verlust:</div>
      <div class="formHeaderElement">Werbung:</div>
      <div class="formHeaderElement">Geschl. Gesellschaft:</div>
      <div class="formHeaderElement">Pers. Verzehr:</div>
    </div>

    <div class="drinkRow" v-for="drink in drinksList" :class="[getExtraClass(drink)]" :key="drink.name">
      <span
        class="drinkColumn"
        :style="{
          borderLeft: `5px solid ${
            drink.categoryColor ? drink.categoryColor : 'grey'
          }`,
        }"
        >{{ drink.name }}</span
      >
      <span class="drinkColumn smallerColumn">{{ drink.displayPrice }}</span>
      <span class="drinkColumn drinksListByService">
        <span class="drinkField" v-if="drink.wastedByService && drink.wastedByService.count > 0">
          {{`${drink.wastedByService.count}`}}
          <span class="drinkFieldCost">
            {{`(${drink.wastedByService.costSum}€)`}}
          </span>
        </span>
      </span>
      <span class="drinkColumn drinksListByKitchen">
        <span class="drinkField" v-if="drink.wastedByKitchen && drink.wastedByKitchen.count > 0">
          {{`${drink.wastedByKitchen.count}`}}
          <span class="drinkFieldCost">
             {{`(${drink.wastedByKitchen.costSum}€)`}}
          </span>
        </span>
      </span>
      <span class="drinkColumn">{{ drink.wasted }}</span>
      <span class="drinkColumn">
        <span class="drinkField" v-if="drink.waste && drink.waste.count > 0">
          <span class="count">
            {{`${drink.waste.count}`}}
          </span>
          <span class="drinkFieldCost">
            {{`(${drink.waste.costSum} €)`}}
          </span>
        </span>
      </span>
      <span class="drinkColumn">
        <span class="drinkField" v-if="drink.giveAway && drink.giveAway.count > 0">
          {{`${drink.giveAway.count}`}}
          <span class="drinkFieldCost">
           {{`(${drink.giveAway.costSum}€)`}}
          </span>
        </span>
      </span>
      <span class="drinkColumn">{{ drink.wasted }}</span>
      <span class="drinkColumn">{{ drink.wasted }}</span>
    </div>

    <div class="summations">
      <span class="drinkColumn">Summe</span>
      <span class="drinkColumn smallerColumn"></span>
      <span class="drinkColumn">
        <span v-if="calculatedConsumption_service > 0">
          {{ Math.round((calculatedConsumption_service + Number.EPSILON) * 100) / 100 }}€
        </span>
      </span>
      <span class="drinkColumn">
        <span v-if="calculatedConsumption_kitchen > 0">
          {{ Math.round((calculatedConsumption_kitchen + Number.EPSILON) * 100) / 100 }}€
        </span>
      </span>
      <span class="drinkColumn"></span>
      <span class="drinkColumn">
        <span v-if="sumOfWasteValue > 0">
          {{ Math.round((sumOfWasteValue + Number.EPSILON) * 100) / 100 }}€
        </span>
      </span>
      <span class="drinkColumn">
        <span v-if="sumOfWasteValue > 0">
          {{ Math.round((sumOfGiveawaysValue + Number.EPSILON) * 100) / 100 }}€
        </span>
      </span>
      <span class="drinkColumn"></span>
      <span class="drinkColumn"></span>
    </div>


    <div class="noPrint tableFooter">
      <span class="resetBtn" @click="reset">Alles zurücksetzen</span>
      <div class="hideEmptyRowsToggle">
        <input type="checkbox" name="shouldHideEmptyRows" id="shouldHideEmptyRows" v-model="shouldHideEmptyRows" />
        <span class="shouldHideEmptyRowsLabel" @click="toggleShouldHideEmptyRows">
          Leere Zeilen ausblenden
        </span>
      </div>
    </div>

    <div class="noPrint">

      <div class="inputsAreaLabel">Dienstverzehr, Verlust, Werbung:</div>
      <div class="inputs">
        <div class="inputsGroup">
          <div class="inputLabelContainer">
            <label for="numberOfWorkers"># Service-Arbeiter </label>
            <input
              type="number"
              name="numberOfWorkers"
              v-model="numberOfWorkers_service"
            />
          </div>
          <div class="inputLabelContainer">
            <label for="numberOfWorkers"># Küchen-Arbeiter </label>
            <input
              type="number"
              name="numberOfWorkers"
              v-model="numberOfWorkers_kitchen"
            />
          </div>
        </div>

        <div class="inputsGroup">
          <div class="inputLabelContainer">
            <label for="umsatz">Umsatz </label>
            <input type="number" name="umsatz" v-model="sales" />
          </div>
          <div class="inputLabelContainer">
            <label for="hostel">Hostel </label>
            <input type="number" name="hostel" v-model="hostel" />
          </div>
        </div>
        <div class="calcNowCtaContainer">
          <div
            :class="[ 'button','noPrint']"
            @click="calculateWastes"
          >
            Ok
          </div>
        </div>
      </div>

      <div class="inputsAreaLabel">
        <span>Personalverzehr (außer Dienst): </span>
        <img alt="T&P" height="15" width="15" class="noPrint hintTrigger" src="../assets/info.png" @click="handleHintTriggerClick_staffDiscount" />
        <Transition>
          <div class="hint" v-if="shouldShowStaffDiscountHint">
            <img alt="T&P" height="15" width="15" class="noPrint hintTrigger" src="../assets/bulb.png" @click="handleHintTriggerClick_staffDiscount" />
            <span>Wenn ein Angestellter außerhalb des Deinstes etwas verzehrt, hier bitte den Namen und den Rabatt (den nicht gezahlten Teil des Originalpreises) angeben!</span>
          </div>
        </Transition>
      </div>
      <div class="inputs">
        <div class="inputsGroup">
          <div class="inputLabelContainer">
            <label for="numberOfWorkers">Name </label>
            <input
              type="text"
              name="staffDiscountName"
              v-model="staffDiscount_name"
              @change="handleStaffDiscountNameInput"
            />
          </div>
          <div class="inputLabelContainer">
            <label for="numberOfWorkers">Rabatt (€) </label>
            <input
              type="text"
              name="staffDiscountDiscount"
              v-model="staffDiscount_name"
              @change="handleStaffDiscountNameInput"
            />
          </div>

          <div class="calcNowCtaContainer">
            <div
              :class="[ 'button','noPrint']"
              @click="addStaffDiscountEntry"
            >
              Ok
            </div>
            <Transition>
              <span v-if="shouldShowStaffDiscountError" :style="{fontSize: '11px', color: '#8a2525', background: 'bisque', marginLeft: '4px', alignSelf: 'center'}">
                Chill! Das Feature ist noch nicht fertig..
              </span>
            </Transition>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref } from "vue";

import { drinks } from "../assets/drinks.js";
import infoIcon from "../assets/info.png";

export default {
  name: "VerzehrZettelForm",
  props: {
    msg: String,
  },
  setup() {
    let rowEven = true;

    const shadowKey = ref(Math.random());
    const consumptionByEachWorker = 15;
    const consumptionByWorkersSplit = 3;
    const wasteAndGiveawaysSplit = 5;
    const percentageOfWasteBySales = Math.random() + 6; // Zahl zwischen 6 und 7
    const percentageOfGiveawaysBySales = 10 - percentageOfWasteBySales;
    const drinksList = ref([...drinks]);
    const shouldHideEmptyRows = ref(false);
    const staffDiscountEntries = ref([{name: "Steve", discount: "6.55"}]);

    const numberOfWorkers_service = ref(0);
    const numberOfWorkers_kitchen = ref(0);
    const calculatedConsumption_service = ref(0);
    const calculatedConsumption_kitchen = ref(0);
    const sumOfConsumptionByWorkers_service = ref(null);
    const sumOfConsumptionByWorkers_kitchen = ref(null);

    const sales = ref(0);
    const hostel = ref(0);
    const calculatedWaste = ref(0);
    const calculatedGiveaways = ref(0);
    const sumOfWasteValue = ref(null);
    const sumOfGiveawaysValue = ref(null);

    const salesAndHostelButtonDisabled = ref(true);
    const salesButtonDisabled = ref(true);

    const shouldShowStaffDiscountHint = ref(false);
    const shouldShowStaffDiscountError = ref(false);

    const getRandomIndex = (min = 0, max = drinksList.value.length - 1) => {
      const index = Math.round(Math.random() * (max - min) + min);
      return index;
    };

    const getRandomDrink = () => {
      const drink = drinksList.value[getRandomIndex()];
      return drink;
    };

    const calculateWastes = () => {
      drinksList.value = [...drinks];
      setTimeout(() => {
        calcConsumptionByWorkers();
        calcWasteAndGiveaways();
      }, 1);
    };

    const calcConsumptionByWorkers = () => {
      sumOfConsumptionByWorkers_service.value =
        numberOfWorkers_service.value * consumptionByEachWorker;
      calculatedConsumption_service.value = 0;

      // Service
      if (numberOfWorkers_service.value > 0) {
        const splitOneProduct_service = getRandomDrink();
        drinksList.value[splitOneProduct_service.index] = {
          ...splitOneProduct_service,
          wastedByService: { count: 0, costSum: 0 },
        };
        const splitTwoProduct_service = getRandomDrink();
        drinksList.value[splitTwoProduct_service.index] = {
          ...splitTwoProduct_service,
          wastedByService: { count: 0, costSum: 0 },
        };
        const splitThreeProduct_service = getRandomDrink();
        drinksList.value[splitThreeProduct_service.index] = {
          ...splitThreeProduct_service,
          wastedByService: { count: 0, costSum: 0 },
        };
        do {
          const randomNumber = getRandomIndex(0, consumptionByWorkersSplit);
          switch (randomNumber) {
            case 1:
              calculatedConsumption_service.value +=
                splitOneProduct_service.price;
              drinksList.value[
                splitOneProduct_service.index
              ].wastedByService.costSum =
                Math.round(
                  (drinksList.value[
                    splitOneProduct_service.index
                  ].wastedByService.costSum +=
                    splitOneProduct_service.price + Number.EPSILON) * 100
                ) / 100;
              drinksList.value[
                splitOneProduct_service.index
              ].wastedByService.count += 1;
              break;
            case 2:
              calculatedConsumption_service.value +=
                splitTwoProduct_service.price;
              drinksList.value[
                splitTwoProduct_service.index
              ].wastedByService.costSum =
                Math.round(
                  (drinksList.value[
                    splitTwoProduct_service.index
                  ].wastedByService.costSum +=
                    splitTwoProduct_service.price + Number.EPSILON) * 100
                ) / 100;
              drinksList.value[
                splitTwoProduct_service.index
              ].wastedByService.count += 1;
              break;
            case 3:
              calculatedConsumption_service.value +=
                splitThreeProduct_service.price;
              drinksList.value[
                splitThreeProduct_service.index
              ].wastedByService.costSum =
                Math.round(
                  (drinksList.value[
                    splitThreeProduct_service.index
                  ].wastedByService.costSum +=
                    splitThreeProduct_service.price + Number.EPSILON) * 100
                ) / 100;
              drinksList.value[
                splitThreeProduct_service.index
              ].wastedByService.count += 1;
              break;
          }
        } while (
          calculatedConsumption_service.value <
          sumOfConsumptionByWorkers_service.value
        );
      }

      // Küche
      if(numberOfWorkers_kitchen.value > 0) {
        sumOfConsumptionByWorkers_kitchen.value =
          numberOfWorkers_kitchen.value * consumptionByEachWorker;
        calculatedConsumption_kitchen.value = 0;

        const splitOneProduct_kitchen = getRandomDrink();
        drinksList.value[splitOneProduct_kitchen.index] = {
          ...splitOneProduct_kitchen,
          wastedByKitchen: { count: 0, costSum: 0 },
        };
        const splitTwoProduct_kitchen = getRandomDrink();
        drinksList.value[splitTwoProduct_kitchen.index] = {
          ...splitTwoProduct_kitchen,
          wastedByKitchen: { count: 0, costSum: 0 },
        };
        const splitThreeProduct_kitchen = getRandomDrink();
        drinksList.value[splitThreeProduct_kitchen.index] = {
          ...splitThreeProduct_kitchen,
          wastedByKitchen: { count: 0, costSum: 0 },
        };

        do {
          const randomNumber = getRandomIndex(0, consumptionByWorkersSplit);
          switch (randomNumber) {
            case 1:
              calculatedConsumption_kitchen.value +=
                splitOneProduct_kitchen.price;
              drinksList.value[
                splitOneProduct_kitchen.index
              ].wastedByKitchen.costSum =
                Math.round(
                  (drinksList.value[
                    splitOneProduct_kitchen.index
                  ].wastedByKitchen.costSum +=
                    splitOneProduct_kitchen.price + Number.EPSILON) * 100
                ) / 100;
              drinksList.value[
                splitOneProduct_kitchen.index
              ].wastedByKitchen.count += 1;
              break;
            case 2:
              calculatedConsumption_kitchen.value +=
                splitTwoProduct_kitchen.price;
              drinksList.value[
                splitTwoProduct_kitchen.index
              ].wastedByKitchen.costSum =
                Math.round(
                  (drinksList.value[
                    splitTwoProduct_kitchen.index
                  ].wastedByKitchen.costSum +=
                    splitTwoProduct_kitchen.price + Number.EPSILON) * 100
                ) / 100;
              drinksList.value[
                splitTwoProduct_kitchen.index
              ].wastedByKitchen.count += 1;
              break;
            case 3:
              calculatedConsumption_kitchen.value +=
                splitThreeProduct_kitchen.price;
              drinksList.value[
                splitThreeProduct_kitchen.index
              ].wastedByKitchen.costSum =
                Math.round(
                  (drinksList.value[
                    splitThreeProduct_kitchen.index
                  ].wastedByKitchen.costSum +=
                    splitThreeProduct_kitchen.price + Number.EPSILON) * 100
                ) / 100;
              drinksList.value[
                splitThreeProduct_kitchen.index
              ].wastedByKitchen.count += 1;
              break;
          }
        } while (
          calculatedConsumption_kitchen.value <
          sumOfConsumptionByWorkers_kitchen.value
        );
      }
    };

    const calcWasteAndGiveaways = () => {
      const salesWithoutHostel = sales.value - hostel.value;
      calculatedWaste.value = Math.floor(salesWithoutHostel / 100 * percentageOfWasteBySales);
      calculatedGiveaways.value = Math.floor(salesWithoutHostel / 100 * percentageOfGiveawaysBySales);

      // Verlust
      if(salesWithoutHostel > 0) {
        sumOfWasteValue.value = 0;
        const splitProductWaste_1 = getRandomDrink();
        drinksList.value[splitProductWaste_1.index] = {
          ...splitProductWaste_1,
          waste: { count: 0, costSum: 0 },
        };
        const splitProductWaste_2 = getRandomDrink();
        drinksList.value[splitProductWaste_2.index] = {
          ...splitProductWaste_2,
          waste: { count: 0, costSum: 0 },
        };
        const splitProductWaste_3 = getRandomDrink();
        drinksList.value[splitProductWaste_3.index] = {
          ...splitProductWaste_3,
          waste: { count: 0, costSum: 0 },
        };
        const splitProductWaste_4 = getRandomDrink();
        drinksList.value[splitProductWaste_4.index] = {
          ...splitProductWaste_4,
          waste: { count: 0, costSum: 0 },
        };
        const splitProductWaste_5 = getRandomDrink();
        drinksList.value[splitProductWaste_5.index] = {
          ...splitProductWaste_5,
          waste: { count: 0, costSum: 0 },
        };

        do {
          const randomNumber = getRandomIndex(0, wasteAndGiveawaysSplit);
          switch (randomNumber) {
            case 1:
              sumOfWasteValue.value +=
                splitProductWaste_1.price;
              drinksList.value[
                splitProductWaste_1.index
              ].waste.costSum =
                Math.round(
                  (drinksList.value[
                    splitProductWaste_1.index
                  ].waste.costSum +=
                    splitProductWaste_1.price + Number.EPSILON) * 100
                ) / 100;
              drinksList.value[
                splitProductWaste_1.index
              ].waste.count += 1;
              break;
            case 2:
              sumOfWasteValue.value +=
                splitProductWaste_2.price;
              drinksList.value[
                splitProductWaste_2.index
              ].waste.costSum =
                Math.round(
                  (drinksList.value[
                    splitProductWaste_2.index
                  ].waste.costSum +=
                    splitProductWaste_2.price + Number.EPSILON) * 100
                ) / 100;
              drinksList.value[
                splitProductWaste_2.index
              ].waste.count += 1;
              break;
            case 3:
              sumOfWasteValue.value +=
                splitProductWaste_3.price;
              drinksList.value[
                splitProductWaste_3.index
              ].waste.costSum =
                Math.round(
                  (drinksList.value[
                    splitProductWaste_3.index
                  ].waste.costSum +=
                    splitProductWaste_3.price + Number.EPSILON) * 100
                ) / 100;
              drinksList.value[
                splitProductWaste_3.index
              ].waste.count += 1;
              break;
            case 4:
              sumOfWasteValue.value +=
                splitProductWaste_4.price;
              drinksList.value[
                splitProductWaste_4.index
              ].waste.costSum =
                Math.round(
                  (drinksList.value[
                    splitProductWaste_4.index
                  ].waste.costSum +=
                    splitProductWaste_4.price + Number.EPSILON) * 100
                ) / 100;
              drinksList.value[
                splitProductWaste_4.index
              ].waste.count += 1;
              break;
            case 5:
              sumOfWasteValue.value +=
                splitProductWaste_5.price;
              drinksList.value[
                splitProductWaste_5.index
              ].waste.costSum =
                Math.round(
                  (drinksList.value[
                    splitProductWaste_5.index
                  ].waste.costSum +=
                    splitProductWaste_5.price + Number.EPSILON) * 100
                ) / 100;
              drinksList.value[
                splitProductWaste_5.index
              ].waste.count += 1;
              break;
          }
        } while (
          sumOfWasteValue.value <
          calculatedWaste.value
        );

      // Werbung
        sumOfGiveawaysValue.value = 0;
        const splitProductGiveAway_1 = getRandomDrink();
        drinksList.value[splitProductGiveAway_1.index] = {
          ...splitProductGiveAway_1,
          giveAway: { count: 0, costSum: 0 },
        };
        const splitProductGiveAway_2 = getRandomDrink();
        drinksList.value[splitProductGiveAway_2.index] = {
          ...splitProductGiveAway_2,
          giveAway: { count: 0, costSum: 0 },
        };
        const splitProductGiveAway_3 = getRandomDrink();
        drinksList.value[splitProductGiveAway_3.index] = {
          ...splitProductGiveAway_3,
          giveAway: { count: 0, costSum: 0 },
        };
        const splitProductGiveAway_4 = getRandomDrink();
        drinksList.value[splitProductGiveAway_4.index] = {
          ...splitProductGiveAway_4,
          giveAway: { count: 0, costSum: 0 },
        };
        const splitProductGiveAway_5 = getRandomDrink();
        drinksList.value[splitProductGiveAway_5.index] = {
          ...splitProductGiveAway_5,
          giveAway: { count: 0, costSum: 0 },
        };

        do {
          const randomNumber = getRandomIndex(0, wasteAndGiveawaysSplit);
          switch (randomNumber) {
            case 1:
              sumOfGiveawaysValue.value +=
                splitProductGiveAway_1.price;
              drinksList.value[
                splitProductGiveAway_1.index
              ].giveAway.costSum =
                Math.round(
                  (drinksList.value[
                    splitProductGiveAway_1.index
                  ].giveAway.costSum +=
                    splitProductGiveAway_1.price + Number.EPSILON) * 100
                ) / 100;
              drinksList.value[
                splitProductGiveAway_1.index
              ].giveAway.count += 1;
              break;
            case 2:
              sumOfGiveawaysValue.value +=
                splitProductGiveAway_2.price;
              drinksList.value[
                splitProductGiveAway_2.index
              ].giveAway.costSum =
                Math.round(
                  (drinksList.value[
                    splitProductGiveAway_2.index
                  ].giveAway.costSum +=
                    splitProductGiveAway_2.price + Number.EPSILON) * 100
                ) / 100;
              drinksList.value[
                splitProductGiveAway_2.index
              ].giveAway.count += 1;
              break;
            case 3:
              sumOfGiveawaysValue.value +=
                splitProductGiveAway_3.price;
              drinksList.value[
                splitProductGiveAway_3.index
              ].giveAway.costSum =
                Math.round(
                  (drinksList.value[
                    splitProductGiveAway_3.index
                  ].giveAway.costSum +=
                    splitProductGiveAway_3.price + Number.EPSILON) * 100
                ) / 100;
              drinksList.value[
                splitProductGiveAway_3.index
              ].giveAway.count += 1;
              break;
            case 4:
              sumOfGiveawaysValue.value +=
                splitProductGiveAway_4.price;
              drinksList.value[
                splitProductGiveAway_4.index
              ].giveAway.costSum =
                Math.round(
                  (drinksList.value[
                    splitProductGiveAway_4.index
                  ].giveAway.costSum +=
                    splitProductGiveAway_4.price + Number.EPSILON) * 100
                ) / 100;
              drinksList.value[
                splitProductGiveAway_4.index
              ].giveAway.count += 1;
              break;
            case 5:
              sumOfGiveawaysValue.value +=
                splitProductGiveAway_5.price;
              drinksList.value[
                splitProductGiveAway_5.index
              ].giveAway.costSum =
                Math.round(
                  (drinksList.value[
                    splitProductGiveAway_5.index
                  ].giveAway.costSum +=
                    splitProductGiveAway_5.price + Number.EPSILON) * 100
                ) / 100;
              drinksList.value[
                splitProductGiveAway_5.index
              ].giveAway.count += 1;
              break;
          }
        } while (
          sumOfGiveawaysValue.value <
          calculatedGiveaways.value
        );
      }
    };

    const handleHintTriggerClick_staffDiscount = () => {
      shouldShowStaffDiscountHint.value = !shouldShowStaffDiscountHint.value;
    }

    const addStaffDiscountEntry = () => {
      if (!shouldShowStaffDiscountError.value) {
        setTimeout(() => {
          shouldShowStaffDiscountError.value = false;
        }, 2500)
      }
      shouldShowStaffDiscountError.value = true;
    }

    const toggleShouldHideEmptyRows = () => {
      shouldHideEmptyRows.value = !shouldHideEmptyRows.value;
    }

    const getExtraClass = (drink) => {
      if((drink.wastedByKitchen !== undefined && drink.wastedByKitchen.costSum > 0)
          || (drink.wastedByService !== undefined && drink.wastedByService.count > 0)
          || (drink.giveAway !== undefined && drink.giveAway.costSum > 0)
          || (drink.waste !== undefined && drink.waste.costSum > 0)) {
            console.log(drink);
            console.log((drink.wastedByKitchen !== undefined && drink.wastedByKitchen.costSum > 0));
            console.log((drink.wastedByService !== undefined && drink.wastedByService.count > 0));
            console.log((drink.giveAway !== undefined && drink.giveAway.costSum > 0));
            console.log((drink.waste !== undefined && drink.waste.costSum > 0));
        rowEven = !rowEven
        return rowEven ? 'highlightedRow_light' : 'highlightedRow_dark';
      } else if(shouldHideEmptyRows.value){
        return 'hideMe'
      }
      return null;
    }

    const reset = () => {
      location.reload();
    }

    return {
      shadowKey,
      sales,
      hostel,
      numberOfWorkers_service,
      numberOfWorkers_kitchen,
      drinksList,
      staffDiscountEntries,
      salesAndHostelButtonDisabled,
      salesButtonDisabled,
      calculatedConsumption_service,
      calculatedConsumption_kitchen,
      sumOfWasteValue,
      sumOfGiveawaysValue,
      shouldShowStaffDiscountHint,
      shouldShowStaffDiscountError,
      shouldHideEmptyRows,

      calculateWastes,
      handleHintTriggerClick_staffDiscount,
      getExtraClass,
      addStaffDiscountEntry,
      toggleShouldHideEmptyRows,
      reset
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

input {
  text-align: right;
}

label {
  position: absolute;
  font-size: 11px;
  left: 8px;
  top: 4px;
}

.tableFooter {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-top: 5px;
}

.hideEmptyRowsToggle {
  display: inline-flex;
  align-items: center;
}

.shouldHideEmptyRowsLabel {
  cursor: pointer;
  font-size: 12px;
}

.inputLabelContainer {
  position: relative;
}

.drinkField {
  position: relative;
  display: block;
}

.drinkFieldCost {
  position: absolute;
  top: 1px;
  right: 1px;
  font-size: 10px;
  background: rgba(255, 255, 255, .5);
  padding: 0 1px;
}

.highlightedRow_light {
  background: rgba(118, 221, 118, 0.2);
}

.highlightedRow_dark {
  background: rgba(16, 158, 16, 0.23);
}

.verzehrZettelForm {
  margin: 1rem 0;
  width: 95vw;
  max-width: 1400px;
}
.formHeader {
  text-align: center;
  font-size: 13px;
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
  padding-right: 4px;
  text-align: center;
}

.inputsAreaLabel {
  margin-top: 2rem;
}

.hintTrigger {
  cursor: pointer;
}

.hint {
  display: flex;
  align-items: center;
  gap: 4px;
  background: cadetblue;
  padding: 4px;
  font-size: 12px;
  max-width: fit-content;
}

.drinkRow {
  max-height: 20px;
  transition: max-height 200ms cubic-bezier(0, 0, 0.86,-0.15);
}

.hideMe {
  max-height: 0px;
  visibility: hidden;
}

.inputs {
  display: flex;
  align-items: baseline;
  gap: 2rem;
  border: 2px solid cadetblue;
  padding: 6px;
  width: fit-content;
}

.resetBtn {
  cursor: pointer;
  font-size: 11px;
  color: royalblue;
  text-decoration: underline;
}

.calcNowCtaContainer {
  display: flex;
  justify-content: center;
}

.inputsGroup {
  gap: 10px;
  display: flex;
  justify-content: center;
}

.summations {
  display: flex;
  border-left: 4px solid black;
  border-top: 1px solid black;
}

.button {
  display: inline;
  margin-left: 5px;
  cursor: pointer;
  font-size: 12px;
  padding: 6px 4px;
  border-radius: 4px;
  background: #181849;
  color: white;
  width: 100px;
  height: 12px;
  align-self: center;
  text-align: center;
}
.button:hover {
  background: #0a0a36;
}
.button.disabled {
  background: lightgrey;
  color: grey;
}

.v-enter-active {
  transition: height 50ms ease;
}
.v-leave-active {
  transition: height 30ms ease;
}

.v-enter-from,
.v-leave-to {
  height: 0;
}

@media print {
  .drinkColumn {
    border-left: 1px solid black !important;
  }
  .salesAndHostel,
  .workers,
  .summations {
    border: none !important;
  }
}
</style>
