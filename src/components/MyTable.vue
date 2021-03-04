<template>
  <div>
    <div class="w-50">
      <div class="d-flex">
        <div class="btn-group m-3">
          <button
            v-for="(years, currency) in currencies"
            :key="currency"
            @click="toggleCurrency(currency)"
            :class="isCurrencyToggled(currency) ? 'btn-primary' : 'btn-light'"
            type="button"
            class="btn btn-primary"
          >
            {{ currency }}
          </button>
        </div>

        <div class="btn-group m-3">
          <button
            v-for="(year, i) in years"
            :key="i"
            @click="toggleYear(year)"
            :class="isYearToggled(year) ? 'btn-primary' : 'btn-light'"
            type="button"
            class="btn btn-primary"
          >
            {{ year }}
          </button>
        </div>

        <div class="btn-group m-3">
          <button
            v-for="(param, i) in params"
            :key="i"
            @click="toggleParam(param)"
            :class="isParamToggled(param) ? 'btn-primary' : 'btn-light'"
            type="button"
            class="btn"
          >
            {{ param }}
          </button>
        </div>
      </div>
      <div class="d-flex justify-content-start">
        <div class="btn-group m-3">
          <input
            v-model="findCompany"
            type="text"
            class="form-control"
            placeholder="Filter by company name..."
          />
        </div>
      </div>
    </div>

    <table class="table table-sm">
      <thead>
        <tr>
          <td width="10%">
            Date sent
            <span @click="sortByDateSent" class="btn">
              {{ sortDateAsc ? "⬆️" : "⬇️" }}
            </span>
          </td>
          <td width="20%">
            Company
            <span @click="sortByCompany" class="btn">
              {{ sortCompanyAsc ? "⬆️" : "⬇️" }}
            </span>
          </td>
          <td v-for="(year, iY) in findYears" :key="iY" width="20%">
            <table width="100%" class="table-borderless">
              <tr>
                <td colspan="2">{{ year | formatYears }}</td>
                <td></td>
              </tr>
              <tr>
                <td>FIX</td>
                <td>FRN</td>
              </tr>
            </table>
          </td>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(item, id) in filteredByCurrency" :key="id">
          <td v-if="item.DateSent" width="20%">{{ item.DateSent | formatDate }}</td>
          <td v-else width="20%"></td>
          <td v-if="item.Quote !== null" colspan="4">
            <table width="100%" class="table-borderless">
              <tr v-for="(param, i) in sortedParams" :key="i">
                <td width="25%" v-if="param === display">
                  <strong>{{ item.Company }}</strong>
                </td>
                <td width="25%" v-else>{{ param }}</td>
                <td v-for="(year, y) in findYears" :key="y">
                  <table width="100%" class="table-borderless">
                    <tr>
                      <td v-if="containsStr(param, 'Spread')" width="50%">
                        {{ getQuoteData(item.Quote, year, "FIX", param) | formatSpread }}
                      </td>
                      <td v-else width="50%">
                        {{ getQuoteData(item.Quote, year, "FIX", param) | formatYield }}
                      </td>
                      <td v-if="containsStr(param, 'Spread')" width="50%">
                        {{ getQuoteData(item.Quote, year, "FRN", param) | formatSpread }}
                      </td>
                      <td v-else width="50%">
                        {{ getQuoteData(item.Quote, year, "FRN", param) | formatYield }}
                      </td>
                    </tr>
                  </table>
                </td>
              </tr>
            </table>
          </td>
          <td v-else width="20%">
            <strong>{{ item.Company }}</strong>
          </td>
        </tr>

        <!-- <tr v-for="item in filteredByCurrency" :key="item.Id">
          <td>
            <tr>
              <td>{{ item.DateSent }}</td>
            </tr>
            <tr>
              <td></td>
            </tr>
            <tr>
              <td></td>
            </tr>
          </td>
          <td v-if="item.Quote !== null" colspan="4">
            <tr v-for="(param, i) in sortedParams" :key="i">
              <td>{{ param === display ? item.Company : param }}</td>
              <td v-for="(year, iY) in findYears" :key="iY">
                <tr>
                  <td>
                    {{ getQuoteData(item.Quote, year, "FIX", param) }}
                  </td>
                  <td>
                    {{ getQuoteData(item.Quote, year, "FRN", param) }}
                  </td>
                </tr>
              </td>
            </tr>
          </td>
          <td v-else>{{ item.Company }}</td>
        </tr> -->

        <!-- <tr v-for="(year, i2) in findYears" :key="i2">
          <td>{{ display }}</td>
          <td>
            <tr>
              <td>
                {{ getAverage(year, "FIX", display) }}
              </td>
              <td>
                {{ getAverage(year, "FRN", display) }}
              </td>
            </tr>
          </td>
        </tr> -->
      </tbody>
    </table>
  </div>
</template>

<script>
import data from "@/mocks/data.json";
import moment from 'moment'

export default {
  name: "my-table",
  data() {
    return {
      findCompany: "",
      findCurrency: "USD",
      findYears: [],
      display: "",

      sortDateAsc: true,
      sortCompanyAsc: true,

      // count: 0,
      // amount: 0,
      // average: 0,

      params: ["Spread", "Yield", "3MLSpread"],
      data: [],
    };
  },
  methods: {
    getQuoteData(quote, years, couponType, returnValue) {
      let data = quote.find(
        (el) => el.Years === years && el.CouponType === couponType
      );

      if (data) {
        switch (returnValue) {
          case "Yield":
            return data.Yield;
          case "Spread":
            return data.Spread;
          case "3MLSpread":
            return data["3MLSpread"];
          default:
            return null;
        }
      }
    },

    containsStr(str, substr) {

      if (str === '' || substr === '') return false
      return str.includes(substr)
    },

    isYearToggled(year) {
      return this.findYears.find((el) => el === year) ? true : false;
    },

    toggleYear(year) {
      // console.log(year);

      if (this.isYearToggled(year)) {
        this.findYears = this.findYears.filter((item) => item !== year);
      } else {
        this.findYears.push(year);
        this.findYears.sort((a, b) => a - b);
      }

      // console.log(this.findYears);
    },

    toggleCurrency(currency) {
      this.findCurrency = currency;
      this.findYears = this.years;
    },
    isCurrencyToggled(currency) {
      return this.findCurrency === currency ? true : false;
    },
    toggleParam(param) {
      this.display = param;
    },
    isParamToggled(param) {
      return this.display === param;
    },

    sortByDateSent() {
      if (this.sortDateAsc) {
        this.data.sort(
          (a, b) => a.DateSent < b.DateSent || a.Preferred < b.Preferred
        );
        this.sortDateAsc = false;
      } else {
        this.data.sort(
          (a, b) => a.DateSent > b.DateSent || a.Preferred > b.Preferred
        );
        this.sortDateAsc = true;
      }

      this.data.sort((a, b) => (a.Quote === null) - (b.Quote === null));
    },

    sortByCompany() {
      if (this.sortCompanyAsc) {
        this.data.sort((a, b) => a.Company < b.Company);
        this.sortCompanyAsc = false;
      } else {
        this.data.sort((a, b) => a.Company > b.Company);

        this.sortCompanyAsc = true;
      }

      this.data.sort((a, b) => (a.Quote === null) - (b.Quote === null));
    },

    _getAverage(years, couponType, display) {
      // const years = 10;
      // const couponType = "FIX";
      // const display = "Spread";
      // var amount = 0;
      // var count = 0;

      // console.log(
      this.data.forEach((el) => {
        if (el.Quote) {
          // console.log(amount, count);
          // console.log(el.Quote)
          el.Quote.forEach((q) => {
            if (q.Years === years && q.CouponType == couponType) {
              this.amount += q[display];
              this.count++;

              // console.log(amount, count);
            }
            // console.log(q.Years, q.CouponType, q.Spread);
          });
        }
      });

      if (this.count > 0) return this.amount / this.count;
      else return null;
    },
  },

  computed: {
    currencies() {
      let tmp = new Object();

      this.data.forEach((el) => {
        if (el.Quote) {
          el.Quote.forEach((q) => {
            let years = q.Years;
            let currency = q.Currency;

            if (!(currency in tmp)) {
              tmp[currency] = [];
            }

            if (!tmp[currency].find((el) => el === years))
              tmp[currency].push(years);
          });
        }
      });

      for (let prop in tmp) {
        tmp[prop].sort((a, b) => a - b);
      }

      return tmp;
    },

    years() {
      return this.currencies[this.findCurrency];
    },

    sortedParams() {
      let tmpDisplay = this.display;
      let tmpParams = [...this.params];
      tmpParams = tmpParams.filter((el) => el !== this.display);

      tmpParams.unshift(tmpDisplay);

      return tmpParams;
    },

    filteredCompanies() {
      let filter = new RegExp(this.findCompany, "i");
      return this.data.filter((el) => el.Company.match(filter));
    },

    filteredByCurrency() {
      let filter = new RegExp(this.findCurrency.toUpperCase(), "i");

      let data = this.filteredCompanies.map((el) => {
        let tmp = Object.assign({}, el);
        let qTmp = el.Quote;
        if (qTmp) {
          qTmp = qTmp.filter((item) => {
            return item.Currency.match(filter);
          });
        }

        tmp.Quote = qTmp;

        return tmp;
      });

      return data;
    },
  },

  filters: {
    formatYears: function(value) {
      return value + 'YRS'
    },
    formatDate: function(value) {
      return moment(value).format('DD-MMM-YY');
    },
    formatSpread: function(value) {
      if (isNaN(+value) || value === null) return 

      if (value > 0) value = '+' + value
      return value + 'bp'
    },
    formatYield: function(value) {
      if (isNaN(+value) || value === null) return 

      return +value.toFixed(3) + '%'
      // return value
    }
  },
  created() {
    this.data = data.Items;

    this.findYears = this.years;

    this.display = this.params[0];

    this.sortByDateSent();


  },
};
</script>

<style scoped>
.table-nested,
.table-nested thead tr th,
.table-nested tbody tr td {
  /* border: 1px solid #000; */
  border: none im !important;
  border-collapse: collapse;
}
</style>