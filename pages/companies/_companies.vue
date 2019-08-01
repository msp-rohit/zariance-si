<template>
  <div>
    <div class="container">
      <CompanyHeader :company="company.basic" />
      <div class="segment columns">
        <template v-for="(metric, index) in topMetrics">
          <div class="metric column is-3" :key="'top'+index" v-if="metric.value && metric.label">
            <span
              class="metric__icon icon is-left"
              :style="'color:'+metric.color"
              v-html="metric.icon"
            ></span>
            <span class="metric__data">
              <span class="metric__value">{{(metric.value.location || metric.value) | millify}}</span>
              <span class="metric__key">{{metric.label}}</span>
            </span>
          </div>
        </template>
        <template v-for="(metric, index) in finMetrics">
          <div class="metric column is-3" :key="'fin'-index" v-if="metric.value && metric.label">
            <span
              class="metric__icon icon is-left"
              :style="'color:'+metric.color"
              v-html="metric.icon"
            ></span>
            <span class="metric__data">
              <span class="metric__value">{{metric.value | millify}}</span>
              <span class="metric__key">{{metric.label}}</span>
            </span>
          </div>
        </template>
      </div>
      <PillNavigation :route="navigation" />
      <NuxtChild :company="company" :key="$route.params.companies" />
    </div>
  </div>
</template>
<script>
import CompanyHeader from "~/components/CompanyHeader.vue";
import PillNavigation from "~/components/PillNavigation.vue";
import style from "~/assets/style/index.sass";

const PropIcons = {
  iconSet: {
    website_traffic: `<svg style="width:24px;height:24px;transform:scale(1.25)" viewBox="0 0 24 24"><path fill="#1973e7" d="M17.9,17.39C17.64,16.59 16.89,16 16,16H15V13A1,1 0 0,0 14,12H8V10H10A1,1 0 0,0 11,9V7H13A2,2 0 0,0 15,5V4.59C17.93,5.77 20,8.64 20,12C20,14.08 19.2,15.97 17.9,17.39M11,19.93C7.05,19.44 4,16.08 4,12C4,11.38 4.08,10.78 4.21,10.21L9,15V16A2,2 0 0,0 11,18M12,2A10,10 0 0,0 2,12A10,10 0 0,0 12,22A10,10 0 0,0 22,12A10,10 0 0,0 12,2Z" /></svg>`,
    headquarters: '<i class="material-icons md-30">location_on</i>',
    employee_count: '<i class="material-icons md-30">face</i>',
    listing_status: '<i class="material-icons md-30">location_city</i>',
    funding: `<svg style="width:24px;height:24px" viewBox="0 0 24 24"><path fill="#000000" d="M7,15H9C9,16.08 10.37,17 12,17C13.63,17 15,16.08 15,15C15,13.9 13.96,13.5 11.76,12.97C9.64,12.44 7,11.78 7,9C7,7.21 8.47,5.69 10.5,5.18V3H13.5V5.18C15.53,5.69 17,7.21 17,9H15C15,7.92 13.63,7 12,7C10.37,7 9,7.92 9,9C9,10.1 10.04,10.5 12.24,11.03C14.36,11.56 17,12.22 17,15C17,16.79 15.53,18.31 13.5,18.82V21H10.5V18.82C8.47,18.31 7,16.79 7,15Z" /></svg>`,
    default: '<i class="material-icons md-30">face</i>'
  },
  get: function(propName) {
    return this.iconSet[propName] || this.iconSet["default"];
  }
};

export default {
  components: {
    CompanyHeader,
    PillNavigation
  },
  name: "CompanyPage",
  created() {
    this.$store.dispatch("company/fetchCompanyInfo", {
      companyName: this.companyName
    });
  },
  filters: {
    millify: number => {
      const _number = Number(number);
      if (isNaN(_number)) return number;
      if (_number < 1e3) return _number.toLocaleString();

      const units = ["k", "M", "B", "T"];
      const order = Math.floor(Math.log(_number) / Math.log(1000));
      const unitName = units[order - 1];
      const num = Number((_number / 1000 ** order).toPrecision(3));

      // output number remainder + unitName
      return num.toLocaleString() + unitName;
    }
  },
  computed: {
    navigation() {
      return this.subRoutes.map(route => {
        route.link = `/companies/${this.companyName}/${route.component}`;
        return route;
      });
    },
    company() {
      return this.$store.state.company.data;
    },
    topMetrics() {
      return this.company.business.topMetrics.map(metric => {
        metric.label = metric.key.replace("_", " ");
        metric.icon = PropIcons.get(metric.key);
        return metric;
      });
    },
    finMetrics() {
      return this.company.business.financialMetrics.map(metric => {
        metric.label = metric.key.replace("_", " ");
        metric.icon = PropIcons.get(metric.key);
        return metric;
      });
    }
  },
  data: function() {
    return {
      companyName: this.$route.params.companies,
      subRoutes: [
        {
          label: "Technical Stack",
          component: ""
        },
        {
          label: "HR Profile",
          component: "hr"
        },
        {
          label: "Business Profile",
          component: "business",
          disabled: true
        },
        {
          label: "News",
          component: "news",
          disabled: true
        }
      ]
    };
  }
};
</script>

<style lang="sass">
.coming-soon
    position: relative
    &:before
        display: block
        content: ''
        width: 100%
        height: 100%
        position: absolute
        background: rgba(255,255,255,.8)
        z-index: 2
    &:after
        display: block
        content: ''
        width: 100%
        height: 100%
        position: absolute
        content: 'Coming Soon'
        font-size: 48px
        font-weight: 500
        color: #333
        text-align: center
        top: 150px
        z-index: 2

</style>
