<template>
  <div>
    <el-row>backend url: {{url}}</el-row>
    <el-row v-if="!gameStart">
      <el-button type="primary" @click="begin">开局</el-button>
    </el-row>
    <el-row v-if="gameStart">
      <el-button type="primary" @click="refreshHeap">牌堆刷新</el-button>
      <el-button type="danger" @click="gameOver">游戏结束</el-button>
      <el-table
        :data="tableData" v-if="tableVis" size="mini" height="500"
        style="width: 100%" border>
        <el-table-column
          prop="name"
          label="卡牌"
          width="100">
        </el-table-column>
        <el-table-column>
          <template slot="header">
            <span>手牌 + 剩余牌堆（</span>
            <i class="icon-heart">：{{suitNum[1]}}</i>&emsp;&emsp;
            <i class="icon-diamonds">：{{suitNum[2]}}</i>&emsp;&emsp;
            <i class="icon-clubs">：{{suitNum[3]}}</i>&emsp;&emsp;
            <i class="icon-spades">：{{suitNum[0]}}</i>
            <span>）</span>
          </template>
          <template slot-scope="scope">
            <el-button-group>
              <el-button
                v-for="card in scope.row.current" :key="card.id"
                :icon="suitIcon[card.suit]" size="mini"
                @click="handleCurrent(scope.$index, scope.row.name, card)">
                {{card.point}}
              </el-button>
            </el-button-group>
          </template>
        </el-table-column>
        <el-table-column
          label="明置卡牌">
          <template slot-scope="scope">
            <el-button-group>
              <el-button
                v-for="card in scope.row.exposed" :key="card.id"
                :icon="suitIcon[card.suit]" size="mini"
                @click="handleExposed(scope.$index, scope.row.name, card)">
                {{card.point}}
              </el-button>
            </el-button-group>
          </template>
        </el-table-column>
        <el-table-column
          label="操作" width="135">
          <template slot-scope="scope" v-if="scope.$index==selected">
            <el-button-group v-if="toExpose">
              <el-button type="primary" size="mini" @click="exposeCurrent" plain>明置</el-button>
              <el-button type="danger" size="mini" @click="abandonCurrent" plain>弃置</el-button>
            </el-button-group>
            <el-button-group v-if="!toExpose">
              <el-button type="info" size="mini" @click="hideExposed" plain>暗置</el-button>
              <el-button type="danger" size="mini" @click="abandonExposed" plain>弃置</el-button>
            </el-button-group>
          </template>
        </el-table-column>
      </el-table>
    </el-row>
  </div>
</template>

<script>
export default {
  name: 'CardRecorder',
  props: {
    url: String
  },
  methods: {
    begin () {
      this.$axios({
        method: 'post',
        url: this.url
      }).then(function (res) {
        this.id = res.data
        this.gameStart = true
        this.getHeap()
      }.bind(this)).catch(function (err) {
        console.log(err)
      })
    },
    gameOver () {
      this.$axios({
        method: 'delete',
        url: this.url + this.id
      }).then(function (res) {
        console.log(res)
        this.gameStart = false
        this.tableVis = false
        this.id = 0
      }.bind(this)).catch(function (err) {
        console.log(err)
      })
    },
    getHeap () {
      this.tableVis = false
      this.$axios({
        method: 'get',
        url: this.url + this.id
      }).then(function (res) {
        console.log(this.id)
        const info = res.data
        this.tableData = []
        this.addASortOfCardsHeap(this.baseCards, info)
        this.addASortOfCardsHeap(this.equipmentCards, info)
        this.addASortOfCardsHeap(this.strategyCards, info)
        this.suitNum = info.suitNum
        console.log(this.suitNum)
        this.tableVis = true
      }.bind(this)).catch(function (err) {
        console.log(err)
      })
    },
    addASortOfCardsHeap (cards, info) {
      for (let card of cards) {
        this.tableData.push({
          name: card,
          current: this.addIdToEveryCard(info['currentHeap'][card]),
          exposed: this.addIdToEveryCard(info['currentExposedHeap'][card])
        })
      }
    },
    addIdToEveryCard (cards) {
      let id = 0
      let cards1 = []
      for (let card of cards) {
        cards1.push({
          'suit': card.suit,
          'point': card.point,
          'id': id++
        })
      }
      return cards1
    },
    handleCurrent (index, name, card) {
      this.handleCard(true, index, name, card)
    },
    handleExposed (index, name, card) {
      this.handleCard(false, index, name, card)
    },
    handleCard (toExpose, index, name, card) {
      this.toExpose = toExpose
      this.selected = index
      this.selectedCard = {
        'name': name,
        'suit': card.suit,
        'point': card.point
      }
    },
    exposeCurrent () {
      this.operateCard('/expose')
    },
    abandonCurrent () {
      this.operateCard('/abandon_current')
    },
    hideExposed () {
      this.operateCard('/hide')
    },
    abandonExposed () {
      this.operateCard('/abandon_exposed')
    },
    operateCard (cls) {
      if (this.selectedCard == null) { return }
      this.$axios({
        method: 'put',
        url: this.url + this.id + cls,
        data: this.selectedCard
      }).then(function () {
        this.selected = -1
        this.selectedCard = null
        this.getHeap()
      }.bind(this)).catch(function (err) {
        console.log(err)
      })
    },
    refreshHeap () {
      this.$axios({
        method: 'put',
        url: this.url + this.id + '/refresh'
      }).then(function () {
        this.getHeap()
      }.bind(this)).catch(function (err) {
        console.log(err)
      })
    }
  },
  data () {
    return {
      gameStart: false,
      tableVis: false,
      id: 0,
      baseCards: ['桃', '闪', '杀', '雷杀', '火杀'],
      equipmentCards: ['加一马', '减一马',
        '藤甲', '白银狮子', '仁王盾', '八卦阵',
        '麒麟弓', '朱雀羽扇', '丈八蛇矛', '贯石斧', '三尖两刃刀', '吴六剑', '寒冰剑', '雌雄双股剑', '青釭剑', '诸葛连弩'],
      strategyCards: ['兵粮寸断', '乐不思蜀', '闪电',
        '五谷丰登', '桃园结义', '万箭齐发', '南蛮入侵',
        '以逸待劳', '铁索连环',
        '远交近攻', '知己知彼', '火攻', '借刀杀人', '顺手牵羊', '过河拆桥', '无中生有', '决斗',
        '无懈可击', '无懈可击•国'],
      suitIcon: {
        'Spade': 'icon-spades',
        'Heart': 'icon-heart',
        'Diamond': 'icon-diamonds',
        'Club': 'icon-clubs'
      },
      tableData: [],
      suitNum: [],
      selected: -1,
      toExpose: true,
      selectedCard: null
    }
  }
}
</script>

<style scoped>
</style>
