<template>
  <div>
    <div class="mdl-grid" v-if="gameStarted">
      <div class="mdl-cell mdl-cell--6-col mdl-cell--4-col-phone">

        <md-card>
          <md-card-header>
            <div class="md-title">You</div>
          </md-card-header>

          <md-card-content>
            <span class="health-text">{{me.HP}} HP</span>
            <md-progress :md-theme="checkMyHP"  :md-progress="me.HP"></md-progress>
          </md-card-content>

          <md-card-actions>
            <md-button class="md-raised md-accent" @click.native="myStep('attack')">Attack</md-button>
            <md-button class="md-raised md-warn" :disabled="me.step < 4" @click.native="myStep('special')">Special Attack</md-button>
            <md-button md-theme="teal" class="md-raised md-primary" :disabled="me.HP == 100" @click.native="myStep('heal')">Heal</md-button>
            <md-button class="md-raised" @click.native="gameStarted = !gameStarted">Give Up</md-button>
          </md-card-actions>

        </md-card>

      </div>

      <div class="mdl-cell mdl-cell--6-col mdl-cell--4-col-phone">

        <md-card>
          <md-card-header>
            <h2 class="md-title">ENEMY</h2>
          </md-card-header>

          <md-card-content>
            <span class="health-text">{{enemy.HP}} HP</span>
            <md-progress :md-theme="checkEnemyHP" :md-progress="enemy.HP"></md-progress>
          </md-card-content>

          <md-card-actions>
            
          </md-card-actions>
        </md-card>

      </div>
      <md-tabs md-centered class="md-transparent">
        <md-tab id="Logs" md-label="Logs">
          <md-list >
            <md-list-item md-centered v-for="(log, i) in gameLog" :key="i">
              <span>{{log}}</span>
            </md-list-item>
          </md-list>
        </md-tab>
      </md-tabs>
    </div>


    <div class="mdl-grid" v-else>


      <div class="mdl-layout-spacer"></div>
      <div class="mdl-cell mdl-cell--6-col mdl-cell--2-col-phone">
        <md-card class="card-example">
          <md-card-area md-inset>

            <md-card-header>
              <h2 class="md-title">WELCOME</h2>
            </md-card-header>
          </md-card-area>

          <md-card-actions>
            <md-button class="md-raised md-primary start-game-button" @click.native="startGame"><h5>Start New Game</h5></md-button>
          </md-card-actions>
        </md-card>
      </div>
      <div class="mdl-layout-spacer"></div>

    </div>
    <md-dialog-alert
      md-title="The Game Ended"
      :md-content-html="gameResult"
      ref="dialog">
    </md-dialog-alert>
  </div>
</template>

<script>
export default {
  name: 'app',
  data () {
    return {
      gameStarted: false,
      me: {
          HP: 80,
          theme: 'green',
          step: 1
      },
      enemy: {
        HP: 50,
        theme: 'green',
        step: 1
      },
      gameResult: 'result',
      gameLog: []
    }
  },
  computed: {
    checkMyHP() {
      return this.parseHP(this.me.HP);
    },
    checkEnemyHP() {
      return this.parseHP(this.enemy.HP);
    }
  },
  methods: {
      startGame() {
          this.gameStarted = true;
          this.enemy.HP = 100;
          this.me.HP = 100;
          this.gameLog = [];
      },
      parseHP(HP) {
        if(HP < 50 && HP > 25) {
          return "warn";
        }
        else if(HP <= 25) {
          return "accent";
        }
        return "green";
      },
      myStep(type) {
        let log = "Player ";
        let rand = 0;
        switch(type) {
          case 'attack':
            rand = this.getRandom(1, 6);
            this.enemy.HP -= rand;
            this.me.step++;
            log += "HITS enemy for " + rand;
            break;
          case 'special':
            rand = this.getRandom(5, 11);
            this.enemy.HP -= rand;
            this.me.step -= 3;
            log += "HITS enemy WITH SPECIAL ATACK for " + rand;
            break;
          case 'heal':
            rand = this.getRandom(1, 11);
            this.me.HP += rand;
            if(this.me.HP > 100) {
              this.me.HP = 100;
            }
            this.me.step++;
            log += "HEALS for " + rand;
            break;
        }
        this.enemyStep();
        this.gameLog.unshift(log);
      },
      enemyStep() {
         let step = this.getRandom(1, 4);
         let log = "Enemy ";
         let rand = 0;

         if(step == 2 && this.enemy.step < 4) {
          this.enemyStep();
          return false;
         }

         if(step == 3 && this.enemy.HP == 100) {
          this.enemyStep();
          return false;
         }

         switch(step) {
          case 1:
            rand = this.getRandom(1, 6);
            this.me.HP -= rand;
            this.enemy.step++;
            log += "HITS player for " + rand;
            break;
          case 2:
            rand = this.getRandom(1, 11);
            this.me.HP -= rand;
            this.enemy.step -= 3;
            log += "HITS player WITH SPECIAL ATTACK for " + rand;
            break;
          case 3:
            rand = this.getRandom(1, 11);
            this.enemy.HP += rand;
            if(this.enemy.HP > 100) {
              this.enemy.HP = 100;
            }
            this.enemy.step++;
            log += "HEALS for " + rand;
            break;
        }
        this.gameLog.unshift(log);
        if(this.enemy.HP < 1) {
          this.gameStarted = false;
          this.gameResult = "YOU WIN";
          this.openDialog();
        }
        else if(this.me.HP < 1) {
          this.gameStarted = false;
          this.gameResult = "YOU LOOSE";
          this.openDialog();
        }
      },
      getRandom(min, max) {
        return Math.floor(Math.random() * (max - min)) + min;
      },
      openDialog() {
        this.$refs['dialog'].open();
      }
  }
}
</script>

<style>
.md-title {
  text-align: center;
}
.start-game-button {
  width: 200px;
  background-color: #1e88e5 !important;
  margin: 0 auto !important;
  display: block;
}
.md-card-actions {
  justify-content: center;
}
.md-progress-track, 
.md-progress {
  height: 50px;
}
.health-text {
  width: 100%;
  text-align: center;
  display: block;
  font-size: 25px;
}
.md-theme-green.md-progress .md-progress-track {
  background-color: #4caf50;
}
.md-theme-green.md-progress {
  background-color: rgba(76, 175, 80, .38);
  margin-top: 10px;
}
.md-theme-warn.md-progress .md-progress-track {
  background-color: #ff5722;
}
.md-theme-warn.md-progress {
  background-color: rgba(255, 87, 34, .38);
  margin-top: 10px;
}
.md-theme-accent.md-progress .md-progress-track {
  background-color: #e91e63;
}
.md-theme-accent.md-progress {
  background-color: rgba(233, 30, 99, .38);
  margin-top: 10px;
}
.md-button.md-raised:not([disabled]) {
  box-shadow: 0 1px 5px rgba(0,0,0,.2), 0 2px 2px rgba(0,0,0,.14), 0 3px 1px -2px rgba(0,0,0,.12);
}
.md-theme-teal.md-button:not([disabled]).md-primary.md-raised,
.md-theme-teal.md-button:not([disabled]).md-primary.md-fab {
  background-color: #009688;
  color: rgba(255, 255, 255, .87);
}
</style>