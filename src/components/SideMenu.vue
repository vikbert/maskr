<template>
  <div id="menu" :class="{ close: toggle }">
    <div class="nav">
      <div class="nav_logo">
        <img src="@/assets/logo.png" alt="" />
      </div>
      <div class="nav_title">Mask Radar</div>
      <div class="nav_toggler" @click="toggle = !toggle">
        <img src="@/assets/ic_toggler@2x.png" alt="" />
      </div>
    </div>
    <perfect-scrollbar
      :options="scrollOptions"
      ref="scroll"
      @ps-scroll-y="onScroll"
    >
      <div class="search">
        <div class="select">
          <select name="citys" v-model="selectedCity">
            <option
              v-for="(item, key) in cities.counties"
              :key="key"
              :value="item"
            >
              {{ item }}
            </option>
          </select>
          <div class="select_arrow"></div>
        </div>
        <div class="select">
          <select name="districts" v-model="selectedDistrict">
            <option value="">---Regions---</option>
            <option
              v-for="(item, key) in getDistricts"
              :key="key"
              :value="item"
            >
              {{ item }}
            </option>
          </select>
          <div class="select_arrow"></div>
        </div>
      </div>
      <div class="title_day">
        <div class="text_big">Mask</div>
        <div class="text">{{ selectedCity }}</div>
        <!-- <div class="info">
          <a
            href="https://g0vhackmd.blob.core.windows.net/g0v-hackmd-images/upload_9d7620679dcf6fd6e5b9bad48dacbf85"
            target="_blank"
          ><img
              src="@/assets/ic_help@2x.png"
              alt=""
            /></a>
        </div> -->
      </div>
      <div class="data_info">
        <div class="info">
          <span
            >State of the mask inventory in {{ selectedCity }}
            {{ selectedDistrict }}</span
          >
          <span>updated {{ updateTime }}</span>
        </div>
        <button class="button" @click="$parent.getMaskData()">Refresh</button>
      </div>
      <div class="cards" v-if="selectedCity === 'Rostock'">
        <div class="card-list">
          <Card
            v-for="item in data"
            :key="item.properties.id"
            :data="item"
            @click.native="updateShow(item)"
          />
        </div>
      </div>
      <div class="cards no-data" v-else>
        <h3>No Geo Data</h3>
        <span>Please select "Rostock"</span>
        <img src="@/assets/no_data.svg" alt="" width="50%">
      </div>
    </perfect-scrollbar>
    <div class="page_top" v-if="scroll > 500 && !toggle" @click="scrollToTop">
      <a href="#"></a>
    </div>
  </div>
</template>
<script>
import { PerfectScrollbar } from 'vue2-perfect-scrollbar';
import 'vue2-perfect-scrollbar/dist/vue2-perfect-scrollbar.css';
import cities from '../data-de';
import Card from './Card';
export default {
  components: { Card, PerfectScrollbar },
  props: ['data'],
  data() {
    return {
      cities: cities,
      toggle: false,
      day: true,
      selectedCity: 'Rostock',
      selectedDistrict: '',
      updateTime: 'None',
      scrollOptions: {
        minScrollbarLength: 30,
      },
      scroll,
    };
  },
  created() {
    const day = new Date().getDay();
    if (day % 2 != 0) {
      this.day = true;
    } else {
      this.day = false;
    }
    this.updateTime = this.getUpdateTime();
    if (window.innerWidth < 760) this.toggle = true;
  },
  methods: {
    updateShow(item) {
      if (window.screen.availWidth < 768) this.toggle = true;
      this.$emit('update-show', item);
    },
    getUpdateTime() {
      for (let i = 0; i < this.data.length; i++) {
        const time = this.data[i].properties.updated;
        if (time != '') {
          return time;
        }
      }
    },
    onScroll(e) {
      this.scroll = e.target.scrollTop;
    },
    scrollToTop() {
      this.$refs.scroll.$el.scrollTo({
        top: 0,
        behavior: 'smooth',
      });
    },
  },
  computed: {
    getDistricts: function () {
      return [];

      // if (this.selectedCity == "") {
      //   return [];
      // }
      // const data = this.cities.districts[
      //   cities.counties.indexOf(this.selectedCity)
      // ];
      // return data[0];
    },
    filterData: function () {
      if (this.selectedDistrict == '') {
        return this.data
          .filter(
            (item) =>
              //item.properties.county.replace(/臺/g, "台") == this.selectedCity
              item.properties.county == this.selectedCity,
          )
          .sort((a, b) => {
            return b.properties.mask_adult - a.properties.mask_adult;
          });
      } else {
        return this.data
          .filter((item) => {
            //const address = item.properties.address.replace(/臺/g, "台");
            const address = item.properties.address;
            return (
              /*item.properties.county.replace(/臺/g, "台") ==
                  this.selectedCity && address.includes(this.selectedDistrict)*/
              item.properties.county == this.selectedCity &&
              address.includes(this.selectedDistrict)
            );
          })
          .sort((a, b) => {
            return b.properties.mask_adult - a.properties.mask_adult;
          });
      }
    },
  },
  watch: {
    selectedCity: function () {
      this.selectedDistrict = '';
      this.$refs.scroll.$el.scrollTop = 0;
    },
    selectedDistrict: function () {
      this.$refs.scroll.$el.scrollTop = 0;
    },
  },
};
</script>
<style lang="scss">
.select {
  position: relative;
  display: inline-block;
  margin-bottom: 15px;
  width: 100%;
}
.select select {
  display: inline-block;
  width: 343px;
  cursor: pointer;
  padding: 15px 20px;
  padding-top: 13px;
  outline: 0;
  border: 1px solid #34495e33;
  border-radius: 10px;
  background: #ffffff;
  font: 16px/16px Noto Sans TC;
  line-height: 22px;
  color: #34495e;
  appearance: none;
  -webkit-appearance: none;
  -moz-appearance: none;
}
.select select::-ms-expand {
  display: none;
}
.select select:hover,
.select select:focus {
  color: #34495e;
  background: #f5f5f5;
}
.select select:disabled {
  opacity: 0.5;
  pointer-events: none;
}
.select_arrow {
  position: absolute;
  top: 17px;
  right: 30px;
  width: 0px;
  height: 0px;
  border: solid #7b7b7b;
  border-width: 0 3px 3px 0;
  display: inline-block;
  padding: 3px;
  transform: rotate(45deg);
  -webkit-transform: rotate(45deg);
}
.select select:hover ~ .select_arrow,
.select select:focus ~ .select_arrow {
  border-color: #34495e;
}
.select select:disabled ~ .select_arrow {
  border-top-color: #cccccc;
}
.center {
  display: flex;
  justify-content: center;
  align-items: center;
}
.close {
  transform: translateX(-310px);
  height: 65px;
}
#menu {
  position: absolute;
  top: 0px;
  left: 0px;
  z-index: 10;
  width: 375px;
  height: 100%;
  background: #fafafa;
  transition-property: all;
  transition-duration: 300ms;
  transition-delay: 0s;
  transition-timing-function: ease;
  font: 16px/24px Noto Sans TC;
  overflow: hidden;
  &.close {
    transform: translateX(-310px);
    height: 65px;
  }
  > .nav {
    @extend .center;
    height: 65px;
    width: 375px;
    background: #ffffff;
    > .nav_logo {
      @extend .center;
      margin-left: 16px;
      margin-right: 10px;
    }
    > .nav_title {
      @extend .center;
      margin-right: auto;
      padding-bottom: 2px;
    }
    > .nav_toggler {
      @extend .center;
      height: 65px;
      width: 65px;
      margin-right: 3px;
    }
  }
  > .ps {
    height: calc(100% - 65px);
  }
  .search {
    margin-top: 16px;
  }
  .title_day {
    display: flex;
    justify-content: flex-start;
    align-items: flex-end;
    padding: 0px 24px;
    height: 54px;
    text-align: left;
    > .text_big {
      font: Bold 36px/54px Noto Sans TC;
      letter-spacing: 0;
      color: #34495e;
    }
    > .text {
      margin-left: 8px;
      padding-bottom: 7px;
      font: Regular 16px/24px Noto Sans TC;
      letter-spacing: 0;
      color: #34495e;
    }
    > .info {
      margin-left: 8px;
      margin-bottom: 7px;
      height: 24px;
      width: 24px;
    }
  }
  .data_info {
    margin-top: 10px;
    padding: 0px 24px;
    @extend .center;
    > .info {
      @extend .center;
      flex-direction: column;
      align-items: flex-start;
      margin-right: auto;
      span {
        text-align: left;
        font: 12px/18px Noto Sans CJK TC;
        letter-spacing: 0;
        color: #566778;
        opacity: 1;
      }
    }
    button {
      height: 36px;
      width: 96px;
      border: 2px solid #34495e;
      border-radius: 100px;
      background: #fafafa;
      opacity: 1;
      font: 14px/20px Noto Sans TC;
      letter-spacing: 0;
      color: #34495e;
      outline: none;
      cursor: pointer;

      &:hover {
        background: #dedede;
      }
    }
  }
  .cards {
    @extend .center;
    flex-direction: column;
    // height: 100%;
    margin-top: 20px;

    position: relative;
    .card-list {
      margin-top: 10px;
      height: 100%;
    }
    .card {
      margin-bottom: 20px;
    }
  }
  .no-data {
      margin-top: 60px;
      img {
        margin-top: 60px;
      }
  }
}
.page_top {
  width: 40px;
  height: 40px;
  position: absolute;
  right: 10px;
  bottom: 10px;
  background: #70777c;
  opacity: 0.6;
  border-radius: 50%;
  a {
    position: relative;
    display: block;
    width: 40px;
    height: 40px;
    text-decoration: none;
  }
  a::before {
    content: '▲';
    font-size: 14px;
    color: #b5b5b5;
    position: absolute;
    width: 25px;
    height: 25px;
    top: -2px;
    bottom: 0;
    right: 0;
    left: 0;
    margin: auto;
    text-align: center;
  }
}
</style>
