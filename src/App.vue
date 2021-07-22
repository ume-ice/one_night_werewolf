<template>

  <Header
    :active-page="activePage"
  />

  <div class="content">

    <div v-show="activePage.page === 'mode'">
      <div class="form-group">
        <div class="form-title">
          ゲームマスター <div class="help-box">？</div>
        </div>
        <div class="form-content">
          <div>
            <label><input type="radio" v-model="mode" :value="true" disabled>あり (comming soon)</label>
          </div>
          <div>
            <label><input type="radio" v-model="mode" :value="false">なし</label>
          </div>
        </div>
      </div>

      <div class="form-group">
        <div class="form-title">
          投票方法 <div class="help-box">？</div>
        </div>
        <div class="form-content">
          <div>
            <label><input type="radio" v-model="isVoteInTurn" :value="true" disabled>順番に投票 (comming soon)</label>
          </div>
          <div>
            <label><input type="radio" v-model="isVoteInTurn" :value="false">追放する人を選択</label>
          </div>
        </div>
      </div>
    </div>

    <div v-show="activePage.page === 'player'">
      <div class="form-group">
        <div class="form-title">
          プレイヤー (4~6名)
        </div>
        <div class="form-content">
          <div v-for="(player, index) in playerList" :key="index">
            {{(index+1)}}.
            <input type="text" v-model="player.name" :placeholder="getPlayerName(index)">
            <a v-show="playerList.length > 4" href="#" @click.prevent.stop="deletePlayer(index)">×</a>
          </div>
          <a v-show="playerList.length < 6" href="#" @click.prevent.stop="addPlayer">プレイヤーを追加</a>
        </div>
      </div>
    </div>

    <div v-show="activePage.page === 'job'">
      <div class="form-group">
        <div class="form-title">
          役職 <div class="help-box">？</div>
        </div>
        <div class="form-content">
          <div>人狼..2枚</div>
          <div>占い..1枚</div>
          <div>怪盗..1枚</div>
          <div>村人..{{playerList.length - 2}}枚</div>
          <div>合計..{{playerList.length + 2}}枚</div>
        </div>
      </div>
    </div>

    <div v-show="activePage.page === 'night-start'">
      <div class="form-group">
        <div class="form-title">
          夜のターン
        </div>
        <div class="form-content">
          夜のターンが始まりました。
          <br>
          皆さんの役職を確認してくださいね。
        </div>
      </div>
    </div>

    <div v-show="activePage.page === 'night-turn'">
      <div class="form-group">
        <div class="form-title">
          夜のターン
        </div>
        <div class="form-content">
          <div>
            {{getPlayerName(playerIndex)}}さんですか？
          </div>
          <a v-show="!isShowJob" href="#" @click.prevent.stop="toggleIsShowJob(true)">役職を見る</a>
          <div v-show="isShowJob">
            あなたの役職は<span class="strong-text">{{getJobName(activePlayer.job)}}</span>です。

            <div v-show="activePlayer.job === 'fortune'" class="margin-top">
              <div v-if="isShowFotunedPlayer">
                <span v-if="fortunePlayerIndex === 'center'">余ったカードは<span class="strong-text">{{getJobName(emptyJobs[0])}}</span>と<span class="strong-text">{{getJobName(emptyJobs[1])}}</span>です</span>
                <span v-else><span class="strong-text">{{getPlayerName(fortunePlayerIndex)}}さん</span>は<span class="strong-text">{{getJobName(fortunedPlayer.job)}}</span>です。</span>
              </div>
              <div v-else>
                占う人を選択してください。
                <br>
                <select v-model="fortunePlayerIndex">
                  <option></option>
                  <option
                    v-for="(player, index) in playerList"
                    :key="index"
                    :value="index"
                    :disabled="index == playerIndex">{{getPlayerName(index)}}</option>
                  <option value="center">余ったカード</option>
                </select>
                <br>
                <a href="#" @click.prevent.stop="fortunePlayer(fortunePlayerIndex)">占う</a>
              </div>
            </div>

            <div v-show="activePlayer.job === 'thief'" class="margin-top">
              <div v-if="isChanged">
                <span class="strong-text">{{getPlayerName(changePlayerIndex)}}</span>さんと交換して<span class="strong-text">{{getJobName(activePlayer.finalJob)}}</span>になりました。
              </div>
              <div v-else>
                交換する人を選択してください。
                <br>
                <select v-model="changePlayerIndex">
                  <option></option>
                  <option
                    v-for="(player, index) in playerList"
                    :key="index"
                    :value="index"
                    :disabled="index == playerIndex">{{getPlayerName(index)}}</option>
                </select>
                <br>
                <a href="#" @click.prevent.stop="changePlayerJob()">交換する</a>
              </div>
            </div>

            <div v-show="activePlayer.job === 'werewolf'" class="margin-top">
              <span v-if="Number.isInteger(otherWereWolf)"><span class="strong-text">{{getPlayerName(otherWereWolf)}}さん</span>も<span class="strong-text">人狼</span>です。</span>
              <span v-else>他に<span class="strong-text">人狼のプレイヤーはいません</span></span>
            </div>

            <div v-show="isShowJob && (activePlayer.job !== 'fortune' || isShowFotunedPlayer)">
              「次へ」を押してください
            </div>

          </div>

        </div>
      </div>
    </div>

    <div v-show="activePage.page === 'talk'">
      <div class="form-group">
        <div class="form-title">
          話し合い
        </div>
        <div class="form-content">
          夜が明けましたね。
          <br>
          話し合いをスタートしましょう。
          <br>
          まだタイマーはないので終わったら次へを押してね。
        </div>
      </div>
    </div>

    <div v-show="activePage.page === 'vote'">
      <div class="form-group">
        <div class="form-title">
          投票
        </div>
        <div class="form-content">
          追放する人を選択してください。
          <div v-for="(player, index) in playerList" :key="index">
            <label>
              <input type="checkbox" v-model="player.isVote">{{getPlayerName(index)}}
            </label>
          </div>
        </div>
      </div>
    </div>

    <div v-show="activePage.page === 'result'">
      <div class="form-group">
        <div class="form-title">
          結果
        </div>
        <div class="form-content">
          {{executionPlayerName}}を追放しました。
          <div class="margin-top">
            {{winnerTeam === 'werewolf' ? '人狼' : '村人'}}チームの勝利です。
          </div>

          <div class="margin-top">
            <div v-for="(player,index) in playerList" :key="index">
              <span v-if="getPlayerResult(player.finalJob || player.job)" class="strong-text">勝ち</span>
              <span v-else class="strong-text" style="color: blue;">負け</span>
              {{getPlayerName(index)}},
              {{getJobName(player.job)}}
              {{player.finalJob ? '=> ' + getJobName(player.finalJob) : ''}}
            </div>
          </div>

        </div>
      </div>
    </div>

  </div>

  <Footer
    :active-page="activePage"
    :button-text="buttonText"
    :move-top-page="moveTopPage"
    :next-page="nextPage"
  />
</template>

<script>
import Header from '@/components/Header.vue'
import Footer from '@/components/Footer.vue'
export default {
  components: {
    Header,
    Footer,
  },
  data () {
    return {
      buttonText: '次へ',
      page: 1,
      mode: false,
      isVoteInTurn: false,
      pageObj: {
        1: {page: 'mode', name: ''},
        2: {page: 'player', name: 'プレイヤー'},
        3: {page: 'job', name: '役職'},
        4: {page: 'night-start', name: '夜のターン'},
        5: {page: 'night-turn', name: '役職確認'},
        6: {page: 'talk', name: '話し合い'},
        7: {page: 'vote', name: '投票'},
        8: {page: 'result', name: '結果'},
      },
      playerList: [],
      emptyJobs: [],
      jobs: [
        'werewolf',
        'werewolf',
        'fortune',
        'thief',
        'villager',
        'villager',
        'villager',
        'villager',
      ],
      playerIndex: 0,
      fortunePlayerIndex: null,
      changePlayerIndex: null,
      isShowJob: false,
      isShowFotunedPlayer: false,
      isChanged: false,
    }
  },
  mounted () {
    for (let i =0; i < 4; i++) {
      this.addPlayer()
    }
  },
  computed: {
    activePage () {
      return this.pageObj[this.page]
    },
    activePlayer () {
      return this.playerList[this.playerIndex] || {}
    },
    fortunedPlayer () {
      return this.playerList[this.fortunePlayerIndex] || {}
    },
    otherWereWolf () {
      if (this.activePlayer.job !== 'werewolf') {
        return null
      }
      let final = null
      this.playerList.forEach((player, index) => {
        if (player.job === 'werewolf' && index != this.playerIndex) {
          final = index
        }
      })
      return final
    },
    executionPlayerIndexes () {
      let final = []
      this.playerList.forEach((player, index) => {
        if (player.isVote) {
          final.push(index)
        }
      })
      return final
    },
    executionPlayerName () {
      return this.executionPlayerIndexes.map((playerIndex) => {
        return this.getPlayerName(playerIndex) + 'さん'
      }).join(',')
    },
    winnerTeam () {
      let winner = 'werewolf'
      let player
      this.executionPlayerIndexes.forEach((playerIndex) => {
        player = this.playerList[playerIndex]
        if ((player.finalJob || player.job) === 'werewolf') {
          winner = 'villager'
        }
      })
      return winner
    }
  },
  methods: {
    moveTopPage () {
      if (this.activePage.page === 'mode' || !confirm('トップページに戻りますか？')) {
        return
      }
      this.page = 1
      this.playerIndex = 0
      this.toggleIsShowJob(false)
      this.fortunePlayerIndex = null
      this.isShowFotunedPlayer = false
      this.changePlayerIndex = null
      this.isChanged = false
      this.emptyJobs = []
      this.resetJob()
    },
    nextPage () {
      let confirmText = ''
      if (this.activePage.page === 'night-start') {
        this.shuffleJob()
      } else if (this.activePage.page === 'night-turn') {
        if (!this.isShowJob) {
          alert('自分の役職を確認してください。')
          return
        }
        if (this.playerList[this.playerIndex].job === 'fortune' && !this.isShowFotunedPlayer) {
          alert('占う人を選択してください。')
          return
        }
        if (this.playerList[this.playerIndex].job === 'thief' && !this.isChanged) {
          alert('交換する人を選択してください。')
          return
        }
        if (this.playerList[this.playerIndex+1]) {
          this.toggleIsShowJob(false)
          this.playerIndex++
          return
        }
      } else if (this.activePage.page === 'talk') {
        confirmText = '話し合いを終了して投票しますか？'
      } else if (this.activePage.page === 'vote') {
        if (!this.executionPlayerIndexes.length) {
          alert('追放する人を選択してください。')
          return
        }
        confirmText = this.executionPlayerName + 'を追放します。よろしいですか。'
      }

      if (confirmText) {
        if (!confirm(confirmText)) {
          return
        }
      }

      if (this.pageObj[this.page+1]) {
        this.page++
      }
    },
    addPlayer () {
      this.playerList.push({name: '', job:'', finalJob:'', isVote: false})
    },
    deletePlayer (index) {
      this.playerList.splice(index, 1)
    },
    getPlayerName (index) {
      return this.playerList[index] && this.playerList[index].name || ('プレイヤー' + (index+1))
    },
    getJobName (job) {
      if (job === 'werewolf') {
        return '人狼'
      } else if (job === 'fortune') {
        return '占い師'
      } else if (job === 'thief') {
        return '怪盗'
      }
      return '村人'
    },
    shuffleJob () {
      let shuffledIndexes = []
      let min = 0
      let max = this.playerList.length + 1
      let ramdomIndex
      while (shuffledIndexes.length < 6) {
        ramdomIndex = Math.floor( Math.random() * (max + 1 - min) ) + min
        if (shuffledIndexes.indexOf(ramdomIndex) === -1) {
          shuffledIndexes.push(ramdomIndex)
        }
      }
      shuffledIndexes.forEach((jobIndex, playerIndex) => {
        if (this.playerList[playerIndex]) {
          this.playerList[playerIndex].job = this.jobs[jobIndex]
        } else {
          this.emptyJobs.push(this.jobs[jobIndex])
        }
      })
    },
    resetJob () {
      this.playerList.forEach((player) => {
        player.job = ''
        player.finalJob = ''
        player.isVote = false
      })
    },
    toggleIsShowJob (isShowJob) {
      this.isShowJob = isShowJob
    },
    fortunePlayer () {
      if (!this.playerList[this.fortunePlayerIndex] && this.fortunePlayerIndex != 'center') {
        alert('プレイヤーを選択してください')
        return
      }

      this.isShowFotunedPlayer = true
    },
    changePlayerJob () {
      if (!this.playerList[this.changePlayerIndex]) {
        alert('プレイヤーを選択してください')
        return
      }
      let thiefIndex = this.playerIndex
      this.playerList[this.changePlayerIndex].finalJob = 'thief'
      this.playerList[thiefIndex].finalJob = this.playerList[this.changePlayerIndex].job
      this.isChanged = true
    },
    getPlayerResult (job) {
      if (job === 'werewolf') {
        return this.winnerTeam === 'werewolf' ? true : false
      } else {
        return this.winnerTeam === 'villager' ? true : false
      }
    }
  }
}
</script>

<style>
html {
  height: 100%;
}
body {
  background: #ECEFF1;
  position: relative;
  max-width: 600px;
  margin: auto;
  color: black;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

.content {
  margin-top: 130px;
  background: #ECEFF1;
  padding-bottom: 80px;
}

.form-group {
  background: #fff;
  margin: 30px;
}

.form-title {
  background: #455A64;
  padding: 10px;
  text-align: left;
  font-size: 18px;
  color: #fff;
  font-weight: bold;
}

.help-box {
  display: inline-block;
  font-size: 12px;
  margin-left: 8px;
  text-decoration: underline;
}

.form-content {
  text-align: left;
  padding: 20px;
}

.margin-top {
  margin-top: 30px;
}

.strong-text {
  font-weight: bold;
  color: #e8204e;
}
</style>
