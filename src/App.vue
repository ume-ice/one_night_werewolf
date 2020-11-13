<template>
  <div class="header">
    ワンナイト人狼 {{activePage.name ? ' - ' + activePage.name : ''}}
  </div>

  <div class="content">

    <div v-show="activePage.page === 'mode'">
      <div class="form-group">
        <div class="form-title">
          ゲームマスター <div class="help-box">？</div>
        </div>
        <div class="form-content">
          <div style="margin-bottom: 20px;">
            <label><input type="radio" name="mode" v-model="mode" :value="true" disabled>あり (comming soon)</label>
          </div>
          <div>
            <label><input type="radio" name="mode" v-model="mode" :value="false">なし</label>
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
            あなたの役職は{{getJobName(activePlayer.job)}}です。

            <div v-show="activePlayer.job === 'fortune'" class="margin-top">
              <div v-if="isShowFotunedPlayer">
                {{getPlayerName(fortunePlayerIndex)}}さんは{{getJobName(fortunedPlayer.job)}}です。
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
                </select>
                <br>
                <a href="#" @click.prevent.stop="fortunePlayer(fortunePlayerIndex)">占う</a>
              </div>
            </div>

            <div v-show="activePlayer.job === 'thief'" class="margin-top">
              <div v-if="isChanged">
                あなたは{{getJobName(activePlayer.finalJob)}}になりました。
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

            <div v-show="isShowJob && (activePlayer.job !== 'fortune' || isShowFotunedPlayer)">
              「次へ」を押してください
            </div>

          </div>

        <div>
          ------------------------------------
          <div v-for="(player,index) in playerList" :key="index">{{getPlayerName(index)}}, {{getJobName(player.job)}} {{player.finalJob ? ' => ' + getJobName(player.finalJob) : ''}}</div>
        </div>

        </div>
      </div>
    </div>

  </div>

  <div class="footer">
    <div class="next-button" @click="moveTopPage">トップ</div>
    <div class="next-button" @click="nextPage">{{buttonText}}</div>
  </div>
</template>

<script>
export default {
  data () {
    return {
      buttonText: '次へ',
      page: 1,
      mode: false,
      pageObj: {
        1: {page: 'mode', name: ''},
        2: {page: 'player', name: 'プレイヤー'},
        3: {page: 'job', name: '役職'},
        4: {page: 'night-start', name: '夜のターン'},
        5: {page: 'night-turn', name: '役職確認'},
      },
      playerList: [
        {name: '', job:'', finalJob:''},
        {name: '', job:'', finalJob:''},
        {name: '', job:'', finalJob:''},
        {name: '', job:'', finalJob:''},
      ],
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
  computed: {
    activePage () {
      return this.pageObj[this.page]
    },
    activePlayer () {
      return this.playerList[this.playerIndex]
    },
    fortunedPlayer () {
      return this.playerList[this.fortunePlayerIndex] || {}
    }
  },
  methods: {
    moveTopPage () {
      this.page = 1
      this.playerIndex = 0
      this.toggleIsShowJob(false)
      this.fortunePlayerIndex = false
      this.isShowFotunedPlayer = false
      this.isChanged = false
      this.emptyJobs = []
      this.resetJob()
    },
    nextPage () {
      if (this.activePage.page === 'night-start') {
        this.shuffleJob();
      } else if (this.activePage.page === 'night-turn') {
        if (!this.isShowJob) {
          alert('自分の役職を確認してください。')
          return
        }
        if (this.playerList[this.playerIndex].job === 'fortune' && !this.isShowFotunedPlayer) {
          alert('占う人を選択してください。')
          return
        }
        // 怪盗確認
        if (this.playerList[this.playerIndex+1]) {
          this.toggleIsShowJob(false)
          this.playerIndex++
          return
        }
      }

      if (this.pageObj[this.page+1]) {
        this.page++
      }
    },
    addPlayer () {
      this.playerList.push({name: ''})
    },
    deletePlayer (index) {
      this.playerList.splice(index, 1)
    },
    getPlayerName (index) {
      return this.playerList[index].name || ('プレイヤー' + (index+1))
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
      var shuffledIndexes = []
      var min = 0
      var max = this.playerList.length + 1
      while (shuffledIndexes.length < 6) {
        var ramdomIndex = Math.floor( Math.random() * (max + 1 - min) ) + min
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
      })
    },
    toggleIsShowJob (isShowJob) {
      this.isShowJob = isShowJob
    },
    fortunePlayer () {
      if (!this.playerList[this.fortunePlayerIndex]) {
        alert('プレイヤーを選択してください')
        return
      }
      this.isShowFotunedPlayer = true;
    },
    changePlayerJob () {
      if (!this.playerList[this.changePlayerIndex]) {
        alert('プレイヤーを選択してください')
        return
      }
      var thiefIndex = this.playerIndex
      this.playerList[this.changePlayerIndex].finalJob = 'thief'
      this.playerList[thiefIndex].finalJob = this.playerList[this.changePlayerIndex].job
      this.isChanged = true
    },
  }
}
</script>

<style>
html {
  height: 100%;
}
body {
  height: -webkit-fill-available;
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

.header {
  height: 100px;
  line-height: 100px;
  background: #455A64;
  color: #fff;
  font-size: 18px;
  font-weight: bold;
  text-align: left;
  padding: 0px 20px;
}

.content {
  background: #ECEFF1;
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

.footer {
  position: absolute;
  height: 80px;
  width: 100%;
  background: #CFD8DC;
  bottom: 0px;
  color: #fff;
  text-align: center;
}

.next-button {
  display: inline-block;
  min-width: 100px;
  height: 40px;
  margin: 20px;
  border-radius: 50px;
  line-height: 40px;
  font-weight: bold;
  background: #455A64;
}
</style>
