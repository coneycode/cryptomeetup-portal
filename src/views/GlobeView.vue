<template>
  <div class="home">
    <!--暂无找到Globe加载完成的回调,利用css层级关系 Globe加载完成覆盖掉Loading-->
    <Loading loadText="loading ..." :zIndex="0" />
    <Globe v-model="activeCountryCode" :countryPrice="countriesPriceMap" />
    <div :class="['country-detail', {'is-active': activeCountryCode}]">
      <div class="globe-control">
        <button class="globe-control-item button is-hidden-mobile is-white is-small is-rounded is-outlined"
          v-show="activeCountryCode !== null"
          @click="clearGlobeFocus()"
        >
          <b-icon icon="arrow-left" size="is-small" />&nbsp;{{$t('back')}}
        </button>
        <div class="mobile-back-button globe-control-item is-hidden-tablet"
          v-show="activeCountryCode !== null"
          @click="clearGlobeFocus()"
        >
          <b-icon icon="arrow-left" />
        </div>
        <b-select class="country-select globe-control-item is-inverted" v-model="activeCountryCode" :placeholder="$t('filter_country_or_region')" icon="filter" size="is-small" rounded>
          <option v-for="code in allCountriesCodes" :value="code" :key="code">{{getCountryName(code)}}</option>
        </b-select>
      </div>
      <div class="country-content" v-if="activeCountryCode">
        <section class="section">
          <section class="section content" v-if="activeCountryCode && landInfo[activeCountryCode]">
            <h1 class="title">Sponsor</h1>
            <p>This country is brought to you by @{{ landInfo[activeCountryCode].owner}}.</p>
            <p><a @click="popupPaymentModal()">Pay {{ $API.getNextPrice(landInfo[activeCountryCode]) | price }} to be the new sponsor</a></p>
          </section>
          <h1 class="title">Meetups in <b> {{getCountryName(activeCountryCode)}} </b></h1>
          <div v-if="activeCountryCode === 'CHN'">
            <MeetupBox v-for="(item,key) in meetupList" :key="key" :data="item"></MeetupBox>
          </div>
          <template v-else>
            <p>There is no meetup.</p>
          </template>
        </section>
      </div>
    </div>
  </div>
</template>

<script>
import * as CountryCode from 'i18n-iso-countries';
import { mapState } from 'vuex';
import * as config from '@/config';
import Land from '@/util/land';
import Globe from '@/components/Globe.vue';
import SponsorPaymentModal from '@/components/SponsorPaymentModal.vue';
import MeetupBox from '@/components/MeetupBox.vue';
import Loading from '@/components/Loading.vue';

CountryCode.registerLocale(require('i18n-iso-countries/langs/en.json'));
CountryCode.registerLocale(require('i18n-iso-countries/langs/zh.json'));
CountryCode.registerLocale(require('i18n-iso-countries/langs/ja.json'));

export default {
  name: 'globe-view',
  components: {
    Globe,
    MeetupBox,
    Loading,
  },
  data() {
    return {
      allCountriesCodes: Object.keys(CountryCode.getAlpha3Codes()),
      countriesPriceMap: {},
      activeCountryCode: null,
      payByPhone: false,
      meetupList: [
        {
          imgurl: 'http://www.xiha.top/upload/default/20181102/9b2baa40e6f5f867729e6a74487ece36.png',
          title: '密码之锥"--2018 CHS·全球区块链应用探索大会',
          date: '11/08 周四',
          location: '杭州',
        }, {
          imgurl: 'https://res.tuoluocaijing.cn/20181022191930-d4mt.jpg?imageView2/3/w/760/h/100/q/75|imageslim',
          title: '中国 DAPP 开发者大会',
          date: '11/09 周五',
          location: '北京',
        },
      ],
    };
  },
  created() {
    this.updateCountryPriceMap();
  },
  mounted() {
    this.$store.commit('ui/setLatestBuyerVisible', true);
  },
  beforeDestroy() {
    this.$store.commit('ui/setLatestBuyerVisible', false);
  },
  computed: {
    ...mapState(['landInfo']),
  },
  methods: {
    clearGlobeFocus() {
      this.activeCountryCode = null;
    },
    getCountryName(countryCode) {
      return CountryCode.getName(countryCode, this.$i18n.locale);
    },
    popupPaymentModal() {
      const referrer = localStorage.getItem(config.referrerStorageKey);
      const landId = Land.countryCodeToLandId(this.activeCountryCode);
      const memo = ['buy_land', String(landId)];
      if (referrer) {
        memo.push(referrer);
      }
      this.$modal.open({
        parent: this,
        component: SponsorPaymentModal,
        hasModalCard: true,
        props: {
          countryName: this.getCountryName(this.activeCountryCode),
          transaction: {
            to: 'cryptomeetup',
            amount: this.$API.getNextPrice(this.landInfo[this.activeCountryCode]),
            memo: memo.join(' '),
          },
        },
      });
    },
    updateCountryPriceMap(landInfo2) {
      const landInfo = landInfo2 || this.landInfo;
      const priceMap = {};
      if (landInfo) {
        Object
          .values(landInfo)
          .forEach((land) => {
            priceMap[land.code] = land.price;
          });
      }
      this.countriesPriceMap = priceMap;
    },
  },
  watch: {
    landInfo(landInfo) {
      this.updateCountryPriceMap(landInfo);
    },
    activeCountryCode(code) {
      this.$store.commit('ui/setNavBurgerVisible', code === null);
    },
  },
};
</script>

<style lang="sass" scoped>
.home
  position: absolute
  left: 0
  top: 0
  width: 100%
  height: 100%

.country-detail
  position: absolute
  right: 0
  top: 0
  height: 100%
  z-index: 2
  pointer-events: none
  transition: background .5s ease-out
  width: 550px
  display: flex
  flex-direction: column

  &.is-active
    pointer-events: auto
    background: rgba(#000, 0.8)

  +mobile
    width: 100%

.country-content
  flex: 1
  margin: 2rem
  overflow: auto

  .section
    padding-left: 0
    padding-right: 0
    padding-top: 0

  +mobile
    margin: 1rem

.globe-control
  margin: 2rem
  z-index: 1
  display: flex
  flex-direction: row
  justify-content: flex-end
  align-items: center

  &-item
    margin-left: 1rem
    pointer-events: auto

  +mobile
    height: $app-nav-height
    margin: 0

.mobile-back-button
  width: $app-nav-height
  height: $app-nav-height
  margin: 0
  display: flex
  justify-content: center
  align-items: center

.country-select
  +mobile
    margin: 0 0.5rem 0 0
    width: calc(100vw - #{$app-nav-height} - 0.5rem)
</style>
