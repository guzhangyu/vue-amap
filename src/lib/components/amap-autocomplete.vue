<template>
	<div class="el-vue-search-box-container"
	     @keydown.up="selectTip('up')"
	     @keydown.down="selectTip('down')">
	  <div class="search-box-wrapper">
	    <input type="text"
	      v-model="keyword"
	      @keyup.enter="tips=[]"
	      @input="autoComplete">
	  </div>
	  <div class="search-tips">
	    <ul>
	      <li v-for="(tip, index) in tips"
	        :key="index"
	        @click="changeTip(tip)"
	        @mouseover="selectedTip=index"
	        :class="{'autocomplete-selected': index === selectedTip}">{{tip.name}}</li>
	    </ul>
	  </div>
	</div>
</template>


<style lang="less">
  .el-vue-search-box-container {
    position: relative;
    width: 360px;
    height: 45px;
    background: #fff;
    box-shadow: 0 2px 2px rgba(0,0,0,.15);
    border-radius: 2px 3px 3px 2px;
    z-index: 10;
    .search-box-wrapper {
      position: absolute;
      display: flex;
      align-items: center;
      left: 0;
      top: 0;
      width: 100%;
      height: 100%;
      box-sizing: border-box;

      input {
        flex: 1;
        height: 20px;
        line-height: 20px;
        letter-spacing: .5px;
        font-size: 14px;
        text-indent: 10px;
        box-sizing: border-box;
        border: none;
        outline: none;
      }

      .search-btn {
        width: 45px;
        height: 100%;
        display: flex;
        align-items: center;
        justify-content: center;
        background: transparent;
        cursor: pointer;
      }
    }

    .search-tips {
      position: absolute;
      top: 100%;
      border: 1px solid #dbdbdb;
      background: #FFF;
      overflow: auto;

      ul {
        padding: 0;
        margin: 0;

        li {
          height: 40px;
          line-height: 40px;
          box-shadow: 0 1px 1px rgba(0,0,0,.1);
          padding: 0 10px;
          cursor: pointer;

          &.autocomplete-selected {
            background: #eee;
          }
        }
      }
    }
  }
</style>
<script>
import RegisterComponentMixin from '../mixins/register-component';
import {lazyAMapApiLoaderInstance} from '../services/injected-amap-api-instance';
export default {
  name: 'el-amap-autocomplete',
  mixins: [RegisterComponentMixin],
  props: ['autoCompleteOption', 'events', 'default', 'onResultSelect'],
  data() {
    return {
      keyword: this.default || '',
      tips: [],
      selectedTip: -1,
      loaded: false,
      adcode: null
    };
  },
  mounted() {
    let _loadApiPromise = lazyAMapApiLoaderInstance.load();
    _loadApiPromise.then(() => {
      this.loaded = true;
      // register init event
      this.events && this.events.init && this.events.init({
        autoComplete: this._autoComplete
      });
    });
  },
  computed: {
    _autoComplete() {
      if (!this.loaded) return;
      return new AMap.AutoComplete(this.autoCompleteOption || {});
    }
  },
  methods: {
	setCity(city) {
		this._autoComplete.setCity(city)
	},
	setCityLimit(cityLimit) {
		this._autoComplete.setCityLimit(cityLimit)
	},
    autoComplete() {
      if (!this.keyword || !this._autoComplete) return;
      this._autoComplete.search(this.keyword, (status, result) => {
        if (status === 'complete') {
          this.tips = result.tips
		  console.log(this.tips)
        }
      });
    },
    changeTip(tip) {
      this.adcode = tip.adcode;
      this.keyword = tip.name;
	  if (this.onResultSelect) {
		  this.onResultSelect(tip.adcode, tip.name, [tip.location.lng, tip.location.lat]);
	  }
	  this.tips = []
    },
    selectTip(type) {
      if (type === 'up' && this.selectedTip > 0) {
        this.selectedTip -= 1;
		const tip = this.tips[this.selectedTip];
        this.keyword = tip.name;
        this.adcode = tip.adcode;
		if (this.onResultSelect) {
			this.onResultSelect(tip.adcode, tip.name, [tip.location.lng, tip.location.lat]);
		}
      } else if (type === 'down' && this.selectedTip + 1 < this.tips.length) {
        this.selectedTip += 1;
	    const tip = this.tips[this.selectedTip];
	    this.keyword = tip.name;
	    this.adcode = tip.adcode;
	    if (this.onResultSelect) {
		  this.onResultSelect(tip.adcode, tip.name, [tip.location.lng, tip.location.lat]);
	    }
      }
    }
  }
};
</script>

