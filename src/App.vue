<template>
  <v-app>
    <v-app-bar app color="primary" dark>
      Financial Independence Calculator
    </v-app-bar>

    <v-main class="grey lighten-3 fill-height">
      <v-container>
        <v-form
          style="padding-left: 20%; padding-right: 20%; padding-top: 10%"
          ref="form"
          lazy-validation
        >
          <v-row>
            <v-col>
              <v-text-field
                v-model="startingPortfolio"
                type="number"
                label="Current portfolio value"
                placeholder="Eg. 124500"
                outlined
                @input="calculate"
              ></v-text-field>
            </v-col>
            <v-col>
              <v-text-field
                v-model="currentAnnualIncome"
                type="number"
                label="Current annual income"
                placeholder="Eg. 43850"
                outlined
                @input="calculate"
              ></v-text-field>
            </v-col>
            <v-col>
              <v-text-field
                v-model="currentAnnualExpenses"
                type="number"
                label="Current annual expenses"
                placeholder="Eg. 12450"
                outlined
                @input="calculate"
              ></v-text-field>
            </v-col>
          </v-row>
          <v-row>
            <v-col>
              <v-slider
                v-model="incomeChangeRate"
                label="Income change rate"
                hint="Yearly change percent in income"
                max="200"
                min="0"
                value="0"
                step="10"
                persistent-hint
                thumb-label="always"
                @change="calculate"
              ></v-slider>
            </v-col>
            <v-col>
              <v-slider
                v-model="expenseChangeRate"
                label="Expenses change rate"
                hint="Yearly change percent in expenses"
                max="200"
                min="0"
                value="0"
                step="10"
                persistent-hint
                thumb-label="always"
                @change="calculate"
              ></v-slider>
            </v-col>
          </v-row>
          <v-row class="my-12">
            <v-col>
              <v-slider
                v-model="portfolioChangeRate"
                label="Annual return on investment"
                hint="Annual growth of portfolio after inflation"
                max="200"
                min="0"
                value="5"
                step="1"
                persistent-hint
                thumb-label="always"
                @change="calculate"
              ></v-slider>
            </v-col>
            <v-col>
              <v-slider
                v-model="withdrawalRate"
                label="Withdrawal rate"
                hint="The percent that a retiree should withdraw from retirement savings each year"
                max="25"
                min="1"
                value="4"
                step="1"
                persistent-hint
                thumb-label="always"
                @change="calculate"
              ></v-slider>
            </v-col>
          </v-row>
          <v-row>
            <v-btn
              block
              color="primary"
              class="my-0"
              x-large
              @click="calculate"
            >
              Calculate
            </v-btn>
          </v-row>
          <v-row align-content="center">
            <v-alert color="blue" type="info" class="pa-md-4 mx-lg-auto my-5">
              You can retire in {{ retirementYear }} years.
            </v-alert>
          </v-row>
        </v-form>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
export default {
  name: "App",

  components: {},

  methods: {
    calculate() {
      let year = 0;
      let income = this.income(year);
      let expenses = this.expenses(year);
      let saving = income - expenses;
      this.currentPortfolio = parseFloat(this.startingPortfolio);
      let annualReturn = this.annualReturn(this.currentPortfolio);
      let withdrawal = this.withdrawal(
        this.currentPortfolio,
        this.withdrawalRate
      );
      while (saving > 0 && withdrawal < expenses) {
        console.log(this.currentPortfolio, annualReturn, saving);
        this.currentPortfolio += annualReturn + saving;
        year++;
        income = this.income(year);
        expenses = this.expenses(year);
        saving = income - expenses;
        annualReturn = this.annualReturn(this.currentPortfolio);
        withdrawal = this.withdrawal(
          this.currentPortfolio,
          this.withdrawalRate
        );
        if (withdrawal > expenses) {
          this.retirementYear = year;
          console.log(
            "FIRE in %d years, Income: %d, Expenses: %d, Portfolio: %d, Withdrawal: %d, Annual Return: %d",
            year,
            income,
            expenses,
            this.currentPortfolio,
            withdrawal,
            annualReturn
          );
          break;
        }
      }
    },
    income(year) {
      return (
        this.currentAnnualIncome *
        Math.pow(1 + this.incomeChangeRate / 100, year)
      );
    },
    expenses(year) {
      return (
        this.currentAnnualExpenses *
        Math.pow(1 + this.expenseChangeRate / 100, year)
      );
    },
    annualReturn(portfolio) {
      return portfolio * (this.portfolioChangeRate / 100);
    },
    withdrawal(balance, rate) {
      return balance * (rate / 100);
    },
  },

  data: () => ({
    retirementYear: "?",
    startingPortfolio: 100000,
    currentPortfolio: null,
    currentAnnualIncome: 50000,
    currentAnnualExpenses: 35000,
    portfolioChangeRate: 5,
    incomeChangeRate: 0,
    expenseChangeRate: 0,
    withdrawalRate: 4,
  }),

  created() {
    this.calculate();
  },
};
</script>
