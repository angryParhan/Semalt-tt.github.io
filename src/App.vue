<template>
  <section id="app">
    <div class="container">
      <div class="sitemaps">
        <div class="sitemaps__titleBlock">
          <div class="titleBlock__item1">
            <span class="sitemaps__description">Submitted sitemaps</span>
            <div class="info-img">
              <div class="sitemaps__info">
                <span>Information Block</span>
              </div>
            </div>
          </div>
          <div class="titleBlock__item2">
            <span class="numberOfselectedItems" v-if="tableBodyChecked.length">Selected sitemaps: <span class="selectedItems">{{ tableBodyChecked.length }}</span></span>
            <span :class="{recrawlBtn : tableBodyChecked.length}">Recrawl sitemap </span>
            <span :class="{removeBtn : tableBodyChecked.length}">Remove sitemap</span>
          </div>
        </div>

        <div class="sitemaps__filters">
          <div class="filters__title">
            <span>Filters:</span>
          </div>
          <div class="filters__items">
            <div
                class="filter__item first_item"
                ref="urlItem"
                :class="{activeFirstButton : selectedItem === 1}"
            >
              <input type="text" value="URL or its part" readonly class="firstInput" @click="activeState($event,1)">
              <div class="dropdownUrl" :class="{activeDropdown : selectedItem === 1}">
                <label>
                  <input type="radio" value="1" v-model.number="radioUrlModel">
                  <span>Sitemap contains</span>
                </label>
                <label>
                  <input type="radio" value="2" v-model.number="radioUrlModel">
                  <span>Sitemap doesn't contain</span>
                </label>
                <label>
                  <input type="radio" value="3" v-model.number="radioUrlModel">
                  <span>Exact match</span>
                </label>
                <hr>
                <div class="dropdownUrl__actions">
                  <button class="reset">Reset</button>
                  <button class="apply">Apply</button>
                </div>
              </div>
            </div>
            <div
                class="filter__item"
                ref="typesItem"
                @click="activeState($event,2)"
                :class="{activeButton : selectedItem === 2}"
            >
              <input type="text" value="All types" readonly>
            </div>
            <div
                class="filter__item"
                ref="statesItem"
                @click="activeState($event, 3)"
                :class="{activeButton : selectedItem === 3}"
            >
              <input type="text" value="Submitted" readonly>
            </div>
            <div class="filter__item date"
                 ref="dateItem"
                 @click="activeState($event, 4)"
                 :class="{activeButton : selectedItem === 4}"
            >
              <input type="text" value="2/12/19 - 2/12.." readonly>
            </div>
            <div class="filter__item"
                 ref="selecteItems"
                 @click="activeState($event, 5)"
                 :class="{activeButton : selectedItem === 5}"
            >
              <input type="text" value="All sitemaps" readonly>
              <div class="sitemapsFilter" :class="{activeDropdown : selectedItem === 5}">
                <label>
                  <input type="radio" value="1" v-model="radioSitemapsModel">
                  <span>Success</span>
                </label>
                <label>
                  <input type="radio" value="2" v-model="radioSitemapsModel">
                  <span>Couldn't fetch</span>
                </label>
                <label>
                  <input type="radio" value="3" v-model="radioSitemapsModel">
                  <span>Errors</span>
                </label>
              </div>
            </div>
          </div>
        </div>


        <div class="sitemaps__tableTitle">
          <span class="tableTitle__checkboxItem">
            <input id="tabeTitle_checkbox" class="checkboxItem" type="checkbox" v-model="selectAllModel">
            <label for="tabeTitle_checkbox"></label>
            Sitemap ({{ tableData.length }})
          </span>
          <span>Type</span>
          <span>Submitted</span>
          <span>Last check</span>
          <span>Status</span>
          <span>URLs</span>
          <span>Recrawl <br> sitemap</span>
          <span>Remove <br> sitemap</span>
        </div>
        <div class="sitemaps__tableBody" >
          <div class="tableBody__item" v-for="(item, i) in tableData" :key="i" :class="{checked : tableBodyChecked.includes(i)}">
            <div class="tableBody__checkboxItem">
              <input :id="`tableBody_checkbox${i + 1}`" class="checkboxItem" type="checkbox"  :value="i" v-model="tableBodyChecked">
              <label :for="`tableBody_checkbox${i + 1}`"></label>
              <span class="link"><a :href="item.path">/sitemap.xml</a></span>
            </div>
            <div><span v-if="item.isSitemapsIndex">Sitemap <br>index</span> <span v-else>False</span></div>
            <div>{{ item.lastSubmitted }} <br> {{ item.lastSubmittedYear }} </div>
            <div class="lastCheckDate">{{ item.lastCheck }} <br> {{ item.lastCheckYear }}</div>
            <div :class="{ error : item.errors > 0, success : item.errors === 0}">{{ item.status }}</div>
            <div><span class="wordBrake">{{ item.urls }}</span></div>
            <div><button class="recrewlBtn">Recrewl</button></div>
            <div>
              <svg class="bin" width="14" height="20" viewBox="0 0 14 20" fill="#9FA2B4" xmlns="http://www.w3.org/2000/svg" @click="deleteEl(i)">
                <path d="M1 17.7778C1 19 1.9 20 3 20H11C12.1 20 13 19 13 17.7778V6.66667C13 5.44444 12.1 4.44444 11 4.44444H3C1.9 4.44444 1 5.44444 1 6.66667V17.7778ZM13 1.11111H10.5L9.79 0.322222C9.61 0.122222 9.35 0 9.09 0H4.91C4.65 0 4.39 0.122222 4.21 0.322222L3.5 1.11111H1C0.45 1.11111 0 1.61111 0 2.22222C0 2.83333 0.45 3.33333 1 3.33333H13C13.55 3.33333 14 2.83333 14 2.22222C14 1.61111 13.55 1.11111 13 1.11111Z"/>
              </svg>
            </div>
          </div>
        </div>


      </div>



    </div>


  </section>
</template>

<script>


export default {
  name: 'App',
  data () {
    return {
      selectedItem: undefined,
      radioUrlModel: undefined,
      radioSitemapsModel: undefined,
      selectAllModel: false,
      tableData: [],
      tableBodyChecked: []
    }
  },
  mounted () {
    fetch('https://semalt.net/dev/api/v1/example/test/')
      .then(response => {
        return response.json()
      })
      .then(data => {
        this.tableData = data.result.sitemap
        this.tableData.forEach(item => {
          item.urls = item.urls.toLocaleString()
          if (item.errors > 0) {
            item.status = `${item.errors} errors`
          } else {
            item.status = 'Success'
          }
          let newDate = new Date(item.lastCheck)
          item.lastCheck = newDate.toDateString().split(' ').slice(1,3).join(' ') + `,`
          item.lastCheckYear =  newDate.toDateString().split(' ').slice(3,4).join(' ')

          newDate = new Date(item.lastSubmitted)
          item.lastSubmitted = newDate.toDateString().split(' ').slice(1,3).join(' ') + `,`
          item.lastSubmittedYear = newDate.toDateString().split(' ').slice(3,4).join(' ')

        })
        console.log(this.tableData)
        console.log(Date.now().toString())
      })
  },
  watch: {
    selectAllModel (current, old) {
      console.log(current, old)
      if (current) {
        this.tableBodyChecked = [0, 1, 2]
      }
    },
    tableBodyChecked (current) {
      console.log(current.length)
      if (current.length !== this.tableData.length) {
        this.selectAllModel = false
      } else {
        this.selectAllModel = true
      }
    }
  },
  methods: {
    activeState (e, item) {
      if (this.selectedItem === item) {
        this.selectedItem = undefined
        return
      }
      this.selectedItem = item
    },
    deleteEl (idx) {
      this.tableData.splice(idx, 1);
    }
  }

}
</script>

<style lang="scss">
  $base-buttons-color: #0288D1;

  @mixin info-block {
    background: #37474F;
    box-shadow: 0px 2px 6px rgba(0, 0, 0, 0.18);
    border-radius: 5px;
    color: #FAFAFA;
    padding: 5px;
    position: absolute;
    display: none;
  }

  @mixin padding-for-sections {
    padding-left: 20px;
    padding-right: 30px;
  }

  @mixin dropdown {
    position: absolute;
    top: 30px;
    left: 0;
    border: 1px solid #BAC9DB;
    box-shadow: 0px 2px 8px rgba(162, 164, 169, 0.46);
    border-radius: 5px;
    background: white;
    display: none;
    z-index: 2;
  }

  @mixin checkboxCustom($topDistance) {
    display: block;
    position: absolute;
    width: 18px;
    min-height: 18px;
    background-image: url("assets/checkbox-off.svg");
    background-repeat: no-repeat;
    top: $topDistance;
    left: 6px;
    line-height: 16px;
    cursor: pointer;
  }

  @mixin tableItem {
    display: flex;
    justify-content: space-between;
  }

  @mixin tableItems_content($height_value) {
    min-height: $height_value;
    display: inline-flex;
    align-items: center;
  }


  .container {
    max-width: 920px;
    margin: 0 auto;
  }

  /*Title block*/

  .sitemaps {
    margin-top: 3rem;
    border: 1px solid #C2CFE0;
    border-radius: 5px;
    font: {
      family: Roboto;
      style: normal;
      weight: normal;
      size: 14px;
    }
    line-height: 16px;
    letter-spacing: 0.01em;
  }

  .sitemaps__description {
    @include padding-for-sections;
    font:{
      weight: bold;
      size: 20px;
    }
    line-height: 23px;
    color: #37474F;
  }

  .info-img {
    width: 16px;
    height: 16px;
    background: url("assets/info-img.svg") no-repeat;
    position: absolute;
    top: 5px;
    right: 5px;
  }

  .info-img:hover .sitemaps__info {
    display: block;
  }

  .titleBlock__item1 {
    position: relative;
  }

  .sitemaps__info {
    @include info-block;
    font-size: 14px;
    width: 120px;
    top: -35px;
    right: -50px;
  }

  .selectedItems {
    font-weight: bold;
    color: #34495D !important;
  }

  .sitemaps__titleBlock {
    display: flex;
    padding: 20px 0;
    justify-content: space-between;
  }

  .titleBlock__item2 span {
    color: #C2CFE0;
    user-select: none;
  }

  .titleBlock__item2 span {
    margin-right: 20px;
  }

  span.numberOfselectedItems {
    color: #34495D;
  }

  span.recrawlBtn {
    color: #0288D1;
    cursor: pointer;
  }

  span.removeBtn {
    color: #FB6868;
    cursor: pointer;
  }

  /*End Title block*/

  /*Filters block start*/

  .sitemaps__filters {
    @include padding-for-sections;
    padding-top: 10px;
    padding-bottom: 10px;
    background: #EAEDF5;
  }

  .filters__title {
    color: #37474F;
    font-weight: 500;
    margin-bottom: 5px;
  }


  .filter__item {
    background: #FFFFFF;
    border: 1px solid #C2CFE0;
    border-radius: 5px;
    display: inline-block;
    width: 18%;
    padding: 5px;
    user-select: none;
    position: relative;
    cursor: pointer;
  }


  .filter__item:not(.first_item)::after {
    content: '';
    height: 100%;
    display: block;
    width: 23px;
    position: absolute;
    right: 0;
    top: 0;
    border-radius: 0px 5px 5px 0px;
    border-left: 1px solid #C2CFE0;
    background-repeat: no-repeat;
    background-image: url("assets/arrow-down.svg");
    background-position: center;
    z-index: 2;
  }

  .activeButton::after {
    background-image: url("assets/arrow-up.svg") !important;
    border-left: 1px solid $base-buttons-color !important;
  }

  .activeButton {
    border-color: $base-buttons-color;
  }

  .activeFirstButton {
    border-color: $base-buttons-color;
  }


  .date {
    padding-left: 20px;
  }

  .sitemaps__filters input[type=text] {
    background: transparent;
    border: none;
    cursor: pointer;
  }

  .sitemaps__filters input[type=text]:not(.firstInput) {
    color: #34495D;
  }

  .firstInput {
    color: #A5B5CB;
  }

  .dropdownUrl {
    @include dropdown;
    padding: 10px 5px;
    width: 160%;

    hr {
      border: 0.5px solid #EAECEF;
      box-shadow: none;
    }
  }

  .dropdownUrl label {
    display: block;
    padding: 10px 0 10px 45px;
  }

  .dropdownUrl label input[type=radio] ~ span {
    font-size: 14px;
    line-height: 16px;
    color: #37474F;
  }

  .dropdownUrl label input[type=radio] {
    visibility: hidden;
  }

  .dropdownUrl label span {
    position: relative;
  }

  .dropdownUrl label input[type=radio]:checked ~ span:before {
    background-image: url("assets/radio-on-img.svg");
  }

  .dropdownUrl label span::before {
    content: '';
    position: absolute;
    display: block;
    width: 18px;
    height: 18px;
    background-image: url("assets/radio-off-img.svg");
    z-index: 20;
    top: -2px;
    left: -40px;
  }

  .dropdownUrl__actions {
    display: flex;
    justify-content: space-between;
  }

  .dropdownUrl__actions button {
    background: none;
    border: none;
  }

  .reset {
    color: #FB6868;
    margin-left: 5px;
  }

  .apply {
    color: $base-buttons-color;
    margin-right: 5px;
  }

  .activeDropdown {
    display: block !important;
  }

  .sitemapsFilter {
    @include dropdown;
    width: 120%;
  }

  .sitemapsFilter label {
    display: block;
    padding: 5px;
    color: #34495D;
    cursor: pointer;
    &:hover {
      background: #F5F8FD;
    }
  }

  .sitemapsFilter label:first-child {
    border-top-right-radius: 5px;
    border-top-left-radius: 5px;
  }

  .sitemapsFilter label:last-child {
    border-bottom-right-radius: 5px;
    border-bottom-left-radius: 5px;
  }

  .sitemapsFilter input[type=radio] {
    visibility: hidden;
  }

  .filters__items {
    display: flex;
    justify-content: space-between;
  }

  /*Filters block end*/

  /*Sitemaps__tableTitle block start*/

  .sitemaps__tableTitle {
    @include padding-for-sections;
    @include tableItem;
    background: #F6F8FE;
  }

  .sitemaps__tableTitle span {
    @include tableItems_content(50px);
    font-weight: bold;
    font-size: 14px;
    color: #34495D;
  }

  .tableTitle__checkboxItem, .tableBody__checkboxItem {
    padding-left: 45px;
    position: relative;
  }

  input.checkboxItem[type=checkbox] {
    visibility: hidden;
  }

  .tableTitle__checkboxItem label {
   @include checkboxCustom(15px);
  }

  input.checkboxItem[type=checkbox]:checked + label {
    background-image: url("assets/checkbox-on.svg");
  }

  /*Sitemaps__tableTitle block end*/

  /*Table Body start*/

  .sitemaps__tableBody {
    border-bottom-right-radius: 5px;
    border-bottom-left-radius: 5px;
  }

  .tableBody__item div {
    @include tableItems_content(70px)
  }

  .tableBody__checkboxItem label {
    @include checkboxCustom(25px);
  }

  .link a {
    text-decoration: none;
    color: $base-buttons-color;
  }

  .link {
    position: relative;
  }

  .link a::after {
    content: '';
    height: 14px;
    width: 14px;
    background-image: url("assets/link.svg");
    background-repeat: no-repeat;
    margin-left: 10px;
    position: absolute;
    z-index: 1;

  }

  .tableBody__item {
    @include padding-for-sections;
    display: flex;
    flex-direction:row;
    justify-content: space-between;
  }


  .recrewlBtn {
    padding: 3px 10px;
    border-radius: 4px;
    border: 1px solid $base-buttons-color;
    color: $base-buttons-color;
    &:hover {
      background: #D2EFFF;
    }
    background: white;
  }

  .bin:hover {
    fill: #FB6868;
    cursor: pointer;
  }

  .bin {
    margin-right: 20px;
  }

  .tableBody__item div:not(.tableBody__checkboxItem) {
    text-align: center;
  }


  .wordBrake {
    word-break: break-word;
    text-align: left !important;
  }

  .tableBody__item {
    border-bottom: 1px solid #F2F4FB;
  }

  .lastCheckDate {
    padding-left: 26px;
  }

  .error {
    color: #FB6868;
  }

  .success {
    color: #2AC9A1;
  }

  .checked {
    background: #E7F6FF;
  }
  /*Table Body end*/



</style>
