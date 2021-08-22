<template>
  <div id="app">
    <div class="selectWrapper">
      <v-branch-select
        :options="branchesArr"
        @select="selectBranchOption"
        :branchSelected="branchSelected"
      ></v-branch-select>
      <v-region-select
        :options="regionsArr"
        @select="selectRegionOption"
        :regionSelected="regionSelected"
      ></v-region-select>
      <button @click="fillFilteredArr()" class="submitBtn">Показать</button>
    </div>
    <div v-for="element in bigArr" :key="element.name" class="block">
      <p class="block__vendorName">{{ element.name }}</p>
      <div class="block__objects">
        <p class="block__objects_element">Количество активных объектов: {{ element.activeStations }}</p>
        <p class="block__objects_element">Количество отключенных объектов: {{ element.disableStations }}</p>
      </div>
    </div>
  </div>
</template>

<script>
import vBranchSelect from './components/v-branch-select'
import vRegionSelect from './components/v-region-select'

export default {
  name: 'App',
  components: {
    vBranchSelect,
    vRegionSelect
  },
  data () {
    return {
      arr: [], //  общий массив
      filteredArr: [], //  массив отфильтрованный по вендорам
      activeStations: '',
      disabledStations: '',
      vendorTempArr: [],
      branchesArr: [], //  массив филиалов
      regionsArr: [], //  массив регионов с кодами и именами
      bigArr: [], //  финальный массив объектов с названием вендора, количеством активных и отключенных объектов
      // vendorList: ['HUAWEI', 'ZTE', 'NOKIA'], // default
      vendorList: [],
      branchSelected: 'Филиал',
      regionSelected: 'Регион',
      inputRegionValue: '',

      //  список филиалов и регионов
      getBranches () {
        return {
          CN: {
            branchCode: 'CN', //  код филиала
            branchName: 'Центральный филиал', // Имя филиала
            regions: [ // список регионов внутри филиала
              { name: 'Брянская область', code: 32 }, //  name: имя региона, code: код региона
              { name: 'Владимирская область', code: 33 },
              { name: 'Калужская область', code: 40 },
              { name: 'Курская область', code: 46 }
            ]
          },
          KV: {
            branchCode: 'KV',
            branchName: 'Кавказский филиал',
            regions: [
              { name: 'Республика Адыгея', code: 1 },
              { name: 'Республика Дагестан', code: 5 },
              { name: 'Краснодарский край', code: 23 },
              { name: 'Воронежская область', code: 36 }
            ]
          }
        }
      },
      //  regionCode – код региона, для которого будет сформирован псевдоответ API
      getObjectsByRegionCode (regionCode) {
        const vendorList = ['HUAWEI', 'ZTE', 'NOKIA']
        const statusList = ['ACTIVE', 'DISABLE']
        const response = {}
        const gri = (max) => {
          return Math.floor(Math.random() * Math.floor(max))
        }
        response[regionCode] = []
        for (let i = 0; i < 100; i++) {
          response[regionCode].push({ vendor: vendorList[gri(3)], name: 'CELL_' + (gri(999) + 1000) + '_' + gri(9), status: statusList[gri(2)] })
        }

        return response
      },
      /**
       * получает из общего массива arr названия всех вендоров,
       * добавляет их во временный массив
       * с помощью new Set удаляет дубликаты и добавляет названия в единичном экземпляре в массив vendorList
       * regionCode - код региона
       */
      getVendorList (regionCode) {
        const vendorTempArr = []
        this.vendorList = []
        this.arr[0][regionCode].forEach(el => {
          vendorTempArr.push(el.vendor)
        })
        this.vendorList = new Set(vendorTempArr)
      },
      /**
       * получает список всех объектов и записывает их в массив arr
       */
      getList () {
        this.arr = []
        this.arr.push(this.getObjectsByRegionCode(this.inputRegionValue))
        // this.fillFilteredArr()
        this.getVendorList(this.inputRegionValue)
      },
      /**
       * первым аргументом принимает необходимый массив объектов отфильтрованных по вендору,
       * вторым аргументом принимает статус 'ACTIVE' или 'DISABLE',
       * фильтрует по принятому статусу и возвращает длинну получившегося массива
       * @param el {array}
       * @param status {String}
       * @returns {number}
       */
      getStationsLength (el, status) {
        return el.filter(function (amount) {
          //  eslint-disable-next-line
          return amount.status == status
        }).length
      },
      /**
       *  фильтрует все объекты по вендорам и заполняет этими данными filteredArr
       */
      fillFilteredArr () {
        this.bigArr = []
        this.filteredArr = []
        this.vendorList.forEach(el => {
          this.filteredArr.push(
            this.arr[0][this.inputRegionValue].filter(function (item) {
              //  eslint-disable-next-line
              return item.vendor == el
            }))
        })
        this.fillingBigArr()
      },
      /**
       * небольшой костыль для объявления переменных для fillBigArr
       */
      fillingBigArr () {
        this.filteredArr.forEach(el => {
          const vendorName = el[0].vendor
          const activeStations = this.getStationsLength(el, 'ACTIVE')
          const disableStations = this.getStationsLength(el, 'DISABLE')
          this.fillBigArr(vendorName, activeStations, disableStations)
        })
      },
      /**
       *  заполняет финальный массив объектов bigArr названием вендора, и количеством объектов, отфильтрованных по статусу,
       *  перебирая отфильтрованный массив filteredArr
       *
       * @param vendorName  {String}
       * @param activeStations  {Number}
       * @param disableStations {Number}
       */
      fillBigArr (vendorName, activeStations, disableStations) {
        this.bigArr.push({ name: vendorName, activeStations: activeStations, disableStations: disableStations })
      }
    }
  },
  created () {
    // получает список филиалов
    this.branchesArr = [this.getBranches()]
  },
  methods: {
    selectBranchOption (option) {
      //  изменяет дефолтный пункт в select
      this.branchSelected = option.branchName
      //  записывает список регионов из выбранного филиала
      this.regionsArr = [this.branchesArr[0][option.branchCode].regions]
    },
    selectRegionOption (option) {
      //  изменяет дефолтный пункт в select
      this.regionSelected = option.name
      //  записывает код региона
      this.inputRegionValue = option.code
      this.getList()
    }
  }
}
</script>

<style lang="scss">
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

li {
  list-style: none;
}

#app {
  max-width: 1200px;
  margin: 60px auto 0;
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: black;
}

.selectWrapper {
  max-width: 1200px;
  width: 100%;
  display: flex;
  justify-content: flex-start;
}

.submitBtn {
  display: block;
  background: green;
  border: none;

  transition: background-color .2s ease-in-out;
  &:hover {
    background: darkgreen;
  }
}

.block {
  max-width: 1200px;
  margin: 50px 0;
  padding: 40px;
  text-align: left;
  border: 1px solid #ccc;
  font-weight: 500;
  color: #8EBAE2;

  &__objects {
    display: flex;
    justify-content: flex-start;
    margin-top: 20px;

    &_element {
      &:not(:last-child) {
        margin-right: 350px;
      }
    }
  }
}
</style>
