<template>
  <div class="home">
    <div class="home-header">
      <!-- Tabbar 标签栏 -->
      <!-- <Tabbar v-model="tabActived" /> -->

      <!-- 選擇時間區間與收尋 -->
      <div class="date-selector">
        <DateSelector v-model="dateRange[0]" />
        <span style="padding: 0 6px;">~</span>
        <DateSelector v-model="dateRange[1]" />
        <!-- <div class="ac-button">
          <van-icon name="search" color="white" size="17px" />
        </div> -->
      </div>
    </div>

    <div class="home-content">
      <div v-if="isLoading" class="loading">
        <van-loading />
      </div>
      <template v-for="(item, index) in filterList">
        <ListItem :key="index" :detail="item" @openEdit="openEdit" @openDelete="openDelete" />
      </template>
    </div>

    <HomeFooter :total="computeTotal" @openAdd="openAdd" />

    <van-dialog v-model="dialogShow" :title="dialogTitle" show-cancel-button @confirm="dialogConfirm">
      <van-cell-group>
        <van-field label="時間">
          <template #input>
            <DateSelector v-model="formData.date" />
          </template>
        </van-field>
      </van-cell-group>
      <van-cell-group>
        <van-field v-model="formData.costTitle" label="項目名稱" placeholder="消費項目" />
      </van-cell-group>
      <van-cell-group>
        <van-field v-model="formData.cost" type="digit" label="金額" placeholder="消費金額" :formatter="numberFormatter" />
      </van-cell-group>
      <van-cell-group>
        <van-field label="收入支出">
          <template #input>
            <SelectButtons v-model="formData.Income" :buttons="incomeBtns" />
          </template>
        </van-field>
      </van-cell-group>
    </van-dialog>
  </div>
</template>

<script>
// import Tabbar from '@/components/Tabbar'
import DateSelector from '@/components/DateSelector'
import ListItem from '@/components/ListItem'
import HomeFooter from '@/components/HomeFooter'
import SelectButtons from '@/components/SelectButtons'
import moment from 'moment'
import { db } from '@/utils/firebase.js'

const getTime = (date) => { return new Date(date).getTime() }

export default {
  name: 'Home',
  components: {
    // Tabbar,
    DateSelector,
    ListItem,
    HomeFooter,
    SelectButtons
  },
  data() {
    return {
      dateRange: [
        moment().subtract(1, 'month').format('YYYY/MM/DD'),
        moment().format('YYYY/MM/DD')
      ],
      listData: [],
      isLoading: false,
      dialogShow: false,
      dialogType: 'add',
      formDataDefault: {
        Income: 'out',
        cost: 0,
        costTitle: '',
        date: moment().format('YYYY/MM/DD'),
        key: ''
      },
      formData: {
        Income: 'out',
        cost: 0,
        costTitle: '',
        date: moment().format('YYYY/MM/DD'),
        key: ''
      },
      incomeBtns: [
        {name: '支出', val: 'out'},
        {name: '收入', val: 'in'}
      ],
    }
  },
  computed: {
    // 過濾日期區間與歸屬
    filterList() {
      return this.listData.filter(item => {
        return (
          getTime(item.date) >= getTime(this.dateRange[0]) &&
          getTime(item.date) <= getTime(this.dateRange[1])
        )
      })
    },
    // 計算總支出收入合計
    computeTotal() {
      const outTotalMap = this.filterList.filter(item => item.Income === 'out').map(item => item.cost)
      const inTotalMap = this.filterList.filter(item => item.Income === 'in').map(item => item.cost)
      let outTotal = outTotalMap.length !== 0 ? outTotalMap.reduce((a,b) => {return  a + b}) : 0
      let inTotal = inTotalMap.length !== 0 ? inTotalMap.reduce((a,b) => {return  a + b}) : 0
      return {outTotal, inTotal, total: outTotal - inTotal}
    },
    dialogTitle() {
      switch (this.dialogType) {
        case 'add':
          return '新增紀錄'
          break;
        case 'edit':
          return '編輯紀錄'
          break;
        default:
          return '請確認彈窗型別'
          break;
      }
    }
  },
  watch: {
    dateRange(val) {
      localStorage.startDate = val[0]
    }
  },
  created() {
    if (localStorage && localStorage.startDate) {
      this.dateRange[0] = localStorage.startDate
    }
  },
  mounted() {
    this.getData()
  },
  methods: {
    // 取得完整 list
    getData() {
      this.isLoading = true
      db.ref('travelList').once('value').then(res => {
        this.isLoading = false
        this.listData = this.setData(res.val())
      })
    },
    // 設定資料並過濾
    setData(data) {
      return Object.keys(data).map(key => {return {...data[key], key}}).sort((a, b) => {
        return getTime(a.date) < getTime(b.date) ? 1 : -1
      })
    },
    // 開啟新增彈窗
    openAdd() {
      this.formData = {...this.formDataDefault}
      this.dialogType = 'add'
      this.dialogShow = true
    },
    // 開啟編輯彈窗
    openEdit(itemData) {
      this.formData = {...itemData}
      this.dialogType = 'edit'
      this.dialogShow = true
    },
    // 開啟刪除彈窗
    openDelete(itemData) {
      this.$dialog.confirm({
        title: '刪除確認',
        message: `確定要刪除 ${itemData.costTitle}`,
      }).then(() => {
        this.isLoading = true
        db.ref('travelList').child(itemData.key).remove()
          .then(() => {
            this.isLoading = false
            this.$toast({message: 'succeeded', icon: 'success'})
            this.getData()
          })
          .catch((error) => {
            console.error("failed: " + error.message)
            this.$toast({message: 'failed', icon: 'cross'})
          })
      }).catch(() => {
        console.log('取消')
      })
    },
    // 確認送出新增編輯
    dialogConfirm() {
      this.isLoading = true
      if (this.dialogType === 'add') {
        db.ref('travelList').push().set({...this.formData})
          .then(() => {
            this.isLoading = false
            this.$toast({message: 'succeeded', icon: 'success'})
            this.getData()
          })
          .catch((error) => {
            console.error("failed: " + error.message)
            this.$toast('failed')
            this.$toast({message: 'failed', icon: 'cross'})
          })
      } else if (this.dialogType === 'edit') {
        db.ref('travelList').child(this.formData.key).set({...this.formData})
          .then(() => {
            this.isLoading = false
            this.$toast({message: 'succeeded', icon: 'success'})
            this.getData()
          })
          .catch((error) => {
            console.error("failed: " + error.message)
            this.$toast({message: 'failed', icon: 'cross'})
          })
      }
    },
    numberFormatter(value) {
      // 过滤输入的数字
      return Number(value)
    }
  }
};
</script>

<style lang="scss" scoped>
.home {
  display: flex;
  flex-direction: column;
  height: 100%
}

.date-selector {
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 20px;
}

.home-content {
  // display: flex;
  flex: 1;
  overflow-y: auto;;

  .ac-button {
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 8px;
    background: #1989fa;
    width: 20px;
    height: 20px;
    border-radius: 50%;
    margin-left: 10px;
  }
}

.loading {
  display: flex;
  justify-content: center;
}
</style>