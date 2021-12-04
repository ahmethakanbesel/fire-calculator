<template>
  <v-app>
    <v-app-bar
        app
        color="primary"
        dark
    >
      Financial Independence Calculator
    </v-app-bar>


    <v-main class="grey lighten-3 fill-height">
      <v-container>
        <v-form
            style="padding-left: 20%;padding-right: 20%;padding-top: 10%;"
            ref="form"
            v-model="valid"
            lazy-validation
        >
          <v-row>
            <v-col>
              <v-text-field
                  v-model="currentPortfolio"
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
                  max="50"
                  min="-50"
                  value="0"
                  step="1"
                  persistent-hint
                  thumb-label="always"
                  @input="calculate"
              ></v-slider>
            </v-col>
            <v-col>
              <v-slider
                  v-model="expenseChangeRate"
                  label="Expenses change rate"
                  hint="Yearly change percent in expenses"
                  max="50"
                  min="-20"
                  value="0"
                  step="1"
                  persistent-hint
                  thumb-label="always"
                  @input="calculate"
              ></v-slider>
            </v-col>
          </v-row>
          <v-row>
            <v-col>
              <v-slider
                  v-model="currentInflationRate"
                  label="Inflation rate"
                  hint="Current inflation rate"
                  max="100"
                  min="0"
                  value="8"
                  step="1"
                  persistent-hint
                  thumb-label="always"
                  @input="calculate"
              ></v-slider>
            </v-col>
            <v-col>
              <v-slider
                  v-model="inflationChangeRate"
                  label="Inflation change rate"
                  hint="Yearly change percent in inflation"
                  max="50"
                  min="-50"
                  value="0"
                  step="1"
                  persistent-hint
                  thumb-label="always"
                  @input="calculate"
              ></v-slider>
            </v-col>
          </v-row>
          <v-row>
            <v-col>
              <v-slider
                  v-model="portfolioChangeRate"
                  label="Annual return on investment"
                  hint="Annual growth of portfolio"
                  max="100"
                  min="0"
                  value="8"
                  step="1"
                  persistent-hint
                  thumb-label="always"
                  @input="calculate"
              ></v-slider>
            </v-col>
            <v-col>
              <v-slider
                  v-model="portfolioGrowthChangeRate"
                  label="Annual change in investment return"
                  hint="Annual change percent in portfolio growth"
                  max="100"
                  min="-10"
                  value="8"
                  step="1"
                  persistent-hint
                  thumb-label="always"
                  @input="calculate"
              ></v-slider>
            </v-col>
            <v-col>
              <v-slider
                  v-model="withdrawalRate"
                  label="Withdrawal rate"
                  hint="Annual change percent in portfolio growth"
                  max="100"
                  min="-10"
                  value="8"
                  step="1"
                  persistent-hint
                  thumb-label="always"
                  @input="calculate"
              ></v-slider>
            </v-col>
          </v-row>
          <v-row align-content="center">
            <v-alert
                color="blue"
                type="info"
            >
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
  name: 'App',

  components: {},

  methods: {
    calculate() {
      var year = 0;
      var income = this.income(year);
      var expenses = this.expenses(year);
      var saving = income - expenses;
      var portfolio = this.currentPortfolio;
      var annualReturnAfterInflation = this.annualReturnAfterInflation(portfolio, year);
      var withdrawal = this.withdrawal(annualReturnAfterInflation, this.withdrawalRate);
      console.log("Year:" + year + " Portfolio Value:" + portfolio + " Annual Return After Inflation:" + annualReturnAfterInflation);
      while (saving > 0 && withdrawal < expenses) {
        portfolio = portfolio + annualReturnAfterInflation + saving;
        year++;
        income = this.income(year);
        expenses = this.expenses(year);
        saving = income - expenses;
        annualReturnAfterInflation = this.annualReturnAfterInflation(portfolio, year);
        withdrawal = this.withdrawal(annualReturnAfterInflation, this.withdrawalRate);
        console.log("Year:%d Portfolio:%d Income:%d Expenses:%d Saving:%d Portfolio Return:%d", year, portfolio, income, expenses, saving, annualReturnAfterInflation);
      }
      if (withdrawal > expenses) {
        console.log("FIRE! " + year);
      }
      return null;
    },
    portfolioAnnualReturn(year) {
      return (this.portfolioChangeRate / 100) * Math.pow(1 + (this.portfolioGrowthChangeRate / 100), year);
    },
    income(year) {
      return this.currentAnnualIncome * Math.pow(1 + (this.incomeChangeRate / 100), year);
    },
    expenses(year) {
      return this.currentAnnualExpenses * Math.pow(1 + (this.expenseChangeRate / 100), year);
    },
    inflationRate(year) {
      return (this.currentInflationRate / 100) * Math.pow(1 + (this.inflationChangeRate / 100), year);
    },
    annualReturn(portfolio, year) {
      return portfolio * this.portfolioAnnualReturn(year);
    },
    annualReturnAfterInflation(portfolio, year) {
      var annualReturn = this.annualReturn(portfolio, year)
      return annualReturn - (annualReturn * this.inflationRate(year))
    },
    withdrawal(balance, rate) {
      return balance * (rate / 100);
    }
  },

  data: () => ({
    retirementYear: 30,
    currentPortfolio: 100000,
    currentAnnualIncome: 100000,
    currentAnnualExpenses: 50000,
    currentInflationRate: 0,
    portfolioChangeRate: 5,
    portfolioGrowthChangeRate: 0,
    inflationChangeRate: 0,
    incomeChangeRate: 0,
    expenseChangeRate: 0,
    withdrawalRate: 4,
    copyText: 'Copy to clipboard',
    number: '',
    binary: '',
    binaryArray: [],
    hex: '',
    littleEndian: false
  }),
};
</script>
