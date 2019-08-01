<template>
  <div class="container">
    <div class="section" v-if="!meta.isLocal">
      <div class="columns is-multiline">
        <h1 class="section__title section__title--logo column is-12">
          <img src="https://assets.mspimages.in/zariance/logo/logo.svg" />
          <span>Sales Intelligence - Prospect Explorer</span>
        </h1>
        <div class="filter-bar column is-12">
          <div class="filter-bar__item">
            <span class="filter-bar__title">Total Records:</span>
            <span
              class="filter-bar__option filter-bar__option--nc"
            >{{companies.meta.totalRecords.toLocaleString()}}</span>
          </div>
          <template v-for="(filterGroup, index) in namedSelectedFilters">
            <div class="filter-bar__item" :key="index" v-if="filterGroup.options.length>0">
              <span class="filter-bar__title">{{filterGroup.label}}</span>
              <span
                class="filter-bar__option"
                v-for="(option, index2) in filterGroup.options"
                :key="index2"
                @click="handleFilterClick(filterGroup.name, option.value)"
              >{{option.label}}</span>
            </div>
          </template>
        </div>
      </div>
      <div class="columns">
        <div class="filter column is-3">
          <div class="filter-group" v-for="(filter, index) in filters" :key="index">
            <div class="filter-title">{{filter.label}}</div>
            <div class="filter-search" v-if="filter.searchable">
              <input
                class="filter-search__input"
                placeholder="Search"
                v-model="filter.search"
                v-on:keyup="searchFilter(filter)"
              />
            </div>
            <div class="filter-item-wrapper">
              <label
                class="pure-material-checkbox"
                v-for="(option, index2) in filteredOptions(filter)"
                :key="index2"
              >
                <input
                  type="checkbox"
                  @change="handleFilterClick(filter.name, option.value)"
                  :checked="option.isSelected"
                />
                <span :title="option.label">{{option.label}}</span>
              </label>
            </div>
          </div>
        </div>
        <div class="column is-9 is-paddingless">
          <div class="segment-body is-marginless">
            <div
              class="company-item columns is-marginless"
              v-for="(company, index) in companies.data"
              :key="index"
            >
              <div class="company-item__image-wrapper column is-1">
                <img :src="company.image" class="company-item__image" v-if="company.image" />
                <FallbackIcon v-else :chars="company.name" />
              </div>
              <div class="company-item__info column is-11">
                <nuxt-link
                  :to="'/companies/'+company.url"
                  class="company-item__name is-lowercase"
                >{{company.name}} &rarr;</nuxt-link>
                <div class="company-item__description">{{company.description}}</div>
                <div class="company-item__kvp columns is-multiline">
                  <div class="company-item__kvp-key column is-6">
                    Website:
                    <a
                      v-if="company.url"
                      :href="'http://'+company.url"
                      target="_blank"
                      class="company-item__kvp-link has-text-weight-medium"
                    >{{company.url}}</a>
                    <span v-else>Not Available</span>
                  </div>
                  <div class="company-item__kvp-key column is-6">
                    Traffic:
                    <span
                      class="has-text-weight-medium"
                      v-if="company.month_6 && company.month_6>0"
                    >{{millify(company.month_6)}}</span>
                    <span v-else>Not Available</span>
                  </div>
                  <div class="company-item__kvp-key column is-6">
                    Headquarters:
                    <span
                      v-if="company.hq_location && company.hq_location.location"
                      class="has-text-weight-medium"
                    >{{company.hq_location.location}}</span>
                    <span v-else>Not Available</span>
                  </div>
                  <div class="company-item__kvp-key column is-6">
                    Year Founded:
                    <span
                      class="has-text-weight-medium"
                      v-if="company.foundation_year"
                    >{{company.foundation_year}}</span>
                    <span v-else>Not Available</span>
                  </div>
                </div>
              </div>
            </div>
          </div>
          <div class="segment-body paginate" v-if="!meta.isLocal">
            <template v-for="(item, index) in pagination">
              <div class="paginate-separator" :key="index+'sp'" v-if="showSeperator(index)">...</div>
              <div
                class="paginate-item"
                :class="{'paginate-item--disabled':item.isCurrentPage}"
                @click="navigateToPage(item.index)"
                :key="index"
              >{{item.index.toLocaleString()}}</div>
            </template>
          </div>
        </div>
      </div>
    </div>
    <div class="section" v-else>
      <div class="section__big-title">Loading...</div>
    </div>
  </div>
</template>

<script>
import FallbackIcon from "~/components/FallbackIcon.vue";

import style from "~/assets/style/index.sass";

import _ from "lodash";
import Axios from "axios";
import queryString from "query-string";

export default {
  name: "Technology",
  components: {
    FallbackIcon
  },
  watch: {},
  computed: {
    meta() {
      return this.$store.state.list.meta;
    },
    pagination() {
      const totalRecords = this.companies.meta.totalRecords,
        pageSize = this.companies.meta.pageSize,
        currentPage = Math.ceil(
          (this.companies.meta.currentIndex + 1) / this.companies.meta.pageSize
        ),
        pageCount = Math.ceil(totalRecords / pageSize);

      const paginationSet = new Set([1, pageCount]);
      let lb = currentPage - 1;
      let ub = currentPage + 1;

      ub = Math.max(ub, 3);
      lb = Math.min(lb, pageCount - 2);

      for (let i = lb; i <= ub; i++) {
        if (i < 1 || i > pageCount) continue;
        paginationSet.add(i);
      }

      let paginationArray = Array.from(paginationSet)
        .sort((a, b) => {
          return a - b;
        })
        .map(item => {
          return {
            index: item,
            isCurrentPage: item === currentPage
          };
        });
      console.log(paginationArray);
      return paginationArray;
    },
    queryParam() {
      return this.toQueryString(this.selectedFilters);
    },
    filters() {
      const localFilterObj = _.cloneDeep(this.$store.state.list.filters);

      let res = localFilterObj.map(filterGroup => {
        filterGroup.options = filterGroup.options.map(option => {
          if (
            this.selectedFilters[filterGroup.name] &&
            this.selectedFilters[filterGroup.name].indexOf(option.value) > -1
          ) {
            option.isSelected = true;
          } else {
            option.isSelected = false;
          }
          return option;
        });
        return filterGroup;
      });
      return res;
    },
    namedSelectedFilters: function() {
      return this.filters.reduce((result, filterGroup) => {
        if (
          this.selectedFilters[filterGroup.name] &&
          this.selectedFilters[filterGroup.name].length > 0
        ) {
          let filterItem = {
            name: filterGroup.name,
            label: filterGroup.label,
            options: filterGroup.options.filter(
              option =>
                this.selectedFilters[filterGroup.name].indexOf(option.value) >
                -1
            )
          };
          result.push(filterItem);
        }
        return result;
      }, []);
      /*
      return _.map(this.selectedFilters, (filterGroupName, filterValues)=>{
        let filterItem = {
          name: filterGroupName
        }
        filterItem.options = filterValues.map(filterValue=>{
          return {
            value: filterValue
          }
        })
        this.filters.map(filterGroup=>{

        });
      })
      */
      return _.reduce(
        this.selectedFilters,
        (result, value, key) => {
          this.filters.map(filterGroup => {
            if (filterGroup.name === key) {
              result[filterGroup.label] = [];
              filterGroup.options.map(option => {
                value.map(selectedValue => {
                  if (selectedValue === option.value) {
                    result[filterGroup.label].push(option.label);
                  }
                });
              });
            }
          });
          return result;
        },
        {}
      );
    },
    companies() {
      return this.$store.state.list.companies;
    }
  },
  mounted() {
    this.readQueryParams();
    this.loadData();
  },
  methods: {
    navigateToPage: function(pageIndex) {
      this.selectedFilters = _.extend(_.cloneDeep(this.selectedFilters), {
        page: pageIndex
      });
      this.loadNewData();
    },
    loadNewData: function() {
      this.$router.push("?" + this.queryParam);
      this.loadData();
    },
    searchFilter: function(filter) {
      this.$store.commit("list/updateSearchTerm", filter);
    },
    millify: function(number) {
      const _number = Number(number);
      if (isNaN(_number)) return number;
      if (_number < 1e3) return _number.toLocaleString();

      const units = ["k", "M", "B", "T"];
      const order = Math.floor(Math.log(_number) / Math.log(1000));
      const unitname = units[order - 1];
      const num = Number((_number / 1000 ** order).toPrecision(3));

      return num.toLocaleString() + unitname;
    },
    handleFilterClick: function(filterGroupName, optionValue) {
      if (this.selectedFilters[filterGroupName]) {
        let currentOptionIndex = this.selectedFilters[filterGroupName].indexOf(
          optionValue
        );

        if (currentOptionIndex > -1) {
          this.selectedFilters[filterGroupName].splice(currentOptionIndex, 1);
          if (this.selectedFilters[filterGroupName].length === 0) {
            delete this.selectedFilters[filterGroupName];
          }
        } else {
          this.selectedFilters[filterGroupName] = [
            ...this.selectedFilters[filterGroupName],
            optionValue
          ];
        }
      } else {
        this.selectedFilters = _.extend(_.cloneDeep(this.selectedFilters), {
          [filterGroupName]: [optionValue]
        });
      }
      this.$router.push("?" + this.queryParam);
      this.loadData();
    },
    loadData: function() {
      this.$store.dispatch("list/fetchData", {
        queryParam: this.queryParam
      });

      this.$store.dispatch("list/fetchFilters", {
        queryParam: this.queryParam
      });
    },
    readQueryParams: function() {
      this.selectedFilters = _.reduce(
        this.$route.query,
        (result, value, key) => {
          result[key] = value.split(",");
          return result;
        },
        {}
      );
    },
    showSeperator: function(index) {
      let flag = false;
      if (this.pagination[index - 1]) {
        let diff =
          this.pagination[index].index - this.pagination[index - 1].index;
        if (diff > 1) {
          flag = true;
        }
      }
      return flag;
    },
    sanitizeName: function(s) {
      return s.replace(/_/g, " ");
    },
    filteredOptions: function(filter) {
      const lowerCaseSearch = filter.search.toLowerCase();
      return filter.options.filter(option => {
        return option.label.toLowerCase().indexOf(lowerCaseSearch) > -1;
      });
    },
    toQueryString: function(obj) {
      return _.map(obj, function(v, k) {
        return encodeURIComponent(k) + "=" + encodeURIComponent(v);
      }).join("&");
    },
    fetchFilters: async function() {
      const { data } = await Axios.get(
        "https://www.zariance.com/api/list_of_filters.php?show=all&v=2"
      );
      const queryParams = this.readQueryParams();
      this.filters = data.map(filterGroup => {
        filterGroup.search = "";
        filterGroup.options = filterGroup.options.map(option => {
          if (
            queryParams[filterGroup.name] &&
            queryParams[filterGroup.name].indexOf(option.value) > -1
          ) {
            option.isSelected = true;
          } else {
            option.isSelected = false;
          }
          return option;
        });
        return filterGroup;
      });
    }
  },
  data: function() {
    return {
      selectedFilters: {
        page: 0
      }
    };
  }
};
</script>

<style lang="sass">
.section__title--logo
  line-height: 36px
  vertical-align: text-bottom
  img
    line-height: 36px
    margin-right: 5px
    vertical-align: text-bottom
  span
    line-height: 36px
    display: inline-block
    vertical-align: bottom
    font-size: 1.6em
.filter
    border-right: 1px solid #dfe1e8
    color: #444
    font-weight: 300
    &-bar
      border: 1px solid #dfe1e8
      padding: 13px
      border-radius: 5px
      margin: 0 0 30px 0
      &__item
        display: inline-block
        & + &
          margin-left: 20px
      &__title
        text-transform: capitalize
        font-weight: 500
      &__option
        display: inline-block
        margin: 0 5px
        font-weight: 300
        font-size: 14px
        cursor: pointer
        &:after
          content: '\00D7'
          display: inline-block
          height: 16px
          width: 16px
          font-size: 18px
          margin-left: 3px
        &--nc
          &:after
            display: none
              

    &-title
        font-size: 16px
        font-weight: 500
        margin-bottom: 10px
        color: #111
    &-search
      border-bottom: 1px solid #dfe1e8
      margin-bottom: 10px
      &__input
        border: 0
        font-size: 14px
        padding: 5px 0
        outline: none
        color: #666
        width: 100%
    &-item
        border-left: 5px solid transparent
        padding: 5px 10px
        display: block
        & + &
            margin-top: 5px
        &:hover:not(.filter-item--active)
            border-color: rgba(26, 115, 232, .1)
        &--active
            border-color: #1a73e8
            font-weight: 500
            color: #111
        &-wrapper
          max-height: 150px
          overflow-y: scroll
    &-group
        & + &
            margin-top: 20px
    
</style>

<style>
.pure-material-checkbox {
  z-index: 0;
  position: relative;
  display: block;
  padding: 0;
  color: rgba(var(--pure-material-onsurface-rgb, 68, 68, 68), 0.87);
  font-family: var(
    --pure-material-font,
    "Roboto",
    "Segoe UI",
    BlinkMacSystemFont,
    system-ui,
    -apple-system
  );
  font-size: 16px;
  line-height: 1.5;
}

/* Input */
.pure-material-checkbox > input {
  appearance: none;
  -moz-appearance: none;
  -webkit-appearance: none;
  z-index: -1;
  position: absolute;
  left: -10px;
  top: -8px;
  display: block;
  margin: 0;
  border-radius: 50%;
  width: 40px;
  height: 40px;
  background-color: rgba(var(--pure-material-onsurface-rgb, 0, 0, 0), 0.6);
  box-shadow: none;
  outline: none;
  opacity: 0;
  transform: scale(1);
  pointer-events: none;
  transition: opacity 0.3s, transform 0.2s;
}

/* Span */
.pure-material-checkbox > span {
  display: inline-block;
  width: 100%;
  cursor: pointer;
  user-select: none;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

/* Box */
.pure-material-checkbox > span::before {
  content: "";
  display: inline-block;
  box-sizing: border-box;
  margin: 3px 11px 3px 1px;
  border: solid 2px; /* Safari */
  border-color: rgba(var(--pure-material-onsurface-rgb, 0, 0, 0), 0.6);
  border-radius: 2px;
  width: 18px;
  height: 18px;
  vertical-align: top;
  transition: border-color 0.2s, background-color 0.2s;
}

/* Checkmark */
.pure-material-checkbox > span::after {
  content: "";
  display: block;
  position: absolute;
  top: 5px;
  left: 2px;
  width: 10px;
  height: 5px;
  border: solid 2px transparent;
  border-right: none;
  border-top: none;
  transform: translate(3px, 4px) rotate(-45deg);
}

/* Checked, Indeterminate */
.pure-material-checkbox > input:checked,
.pure-material-checkbox > input:indeterminate {
  background-color: rgb(var(--pure-material-primary-rgb, 33, 150, 243));
}

.pure-material-checkbox > input:checked + span::before,
.pure-material-checkbox > input:indeterminate + span::before {
  border-color: rgb(var(--pure-material-primary-rgb, 33, 150, 243));
  background-color: rgb(var(--pure-material-primary-rgb, 33, 150, 243));
}

.pure-material-checkbox > input:checked + span::after,
.pure-material-checkbox > input:indeterminate + span::after {
  border-color: rgb(var(--pure-material-onprimary-rgb, 255, 255, 255));
}

.pure-material-checkbox > input:indeterminate + span::after {
  border-left: none;
  transform: translate(4px, 3px);
}

/* Hover, Focus */
.pure-material-checkbox:hover > input {
  opacity: 0.04;
}

.pure-material-checkbox > input:focus {
  opacity: 0.12;
}

.pure-material-checkbox:hover > input:focus {
  opacity: 0.16;
}

/* Active */
.pure-material-checkbox > input:active {
  opacity: 1;
  transform: scale(0);
  transition: transform 0s, opacity 0s;
}

.pure-material-checkbox > input:active + span::before {
  border-color: rgb(var(--pure-material-primary-rgb, 33, 150, 243));
}

.pure-material-checkbox > input:checked:active + span::before {
  border-color: transparent;
  background-color: rgba(var(--pure-material-onsurface-rgb, 0, 0, 0), 0.6);
}

/* Disabled */
.pure-material-checkbox > input:disabled {
  opacity: 0;
}

.pure-material-checkbox > input:disabled + span {
  color: rgba(var(--pure-material-onsurface-rgb, 0, 0, 0), 0.38);
  cursor: initial;
}

.pure-material-checkbox > input:disabled + span::before {
  border-color: currentColor;
}

.pure-material-checkbox > input:checked:disabled + span::before,
.pure-material-checkbox > input:indeterminate:disabled + span::before {
  border-color: transparent;
  background-color: currentColor;
}
</style>

<style lang="sass">
$box-shadow: 0 1px 6px 1px rgba(92,78,62,.28)

.company-item
    border-radius: 5px
    margin: 0
    padding: 15px
    position: relative
    &__name
        font-size: 20px
        font-weight: 500
        line-height: 1
    &__date
        color: #666
        font-size: 14px
        margin-top: 8px
        font-weight: 300
    &__kvp
        color: #666
        font-size: 14px
        font-weight: 300
        margin-top: 10px
        &-key
          padding: 3px 12px
        &-link
          color: #3273dc
          &:hover
            text-decoration: underline
    &__description
        color: #666 
        font-weight: 300
        font-size: 14px
        margin-top: 10px
    & + &
        border-top: 1px solid #dfe1e8
.sdbr
    border-left: 1px solid #dfe1e8
.section
  &__title
    font-size: 22px
    color: #111
    margin-bottom: 20px

.paginate
  display: flex
  justify-content: center
  &-item
    border: 1px solid #444
    color: #444
    min-width: 35px
    padding: 3px 8px
    text-align: center
    border-radius: 3px
    display: inline-block
    cursor: pointer
    transition: .1s ease-in-out
    margin: 0 5px
    &:not(&--disabled):hover
      background-color: #1a73e8
      color: #fff
      border-color: #1a73e8
    &--disabled
      cursor: default
      background: #f2f3f5
      border-color: #999
      color: #666

.section__big-title
  text-align: center
  font-size: 30px
  margin-top: 100px
</style>