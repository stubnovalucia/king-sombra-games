<template>
    <div id="app">
      <v-app class="ma-0">

        <v-container fluid class="main-container text-center">

          <template v-if="!gameSelected">
            <h1 class="mt-2">KING SOMBRA GAMES</h1>
            <v-row>
              <v-col cols="4" offset="2">
                <v-card color="deep-purple darken-1" class="choose-game-btn" dark @click="selectGame('snake')">SNAKE</v-card>
              </v-col>
              <v-col cols="4">
                <v-card color="deep-purple darken-1" class="choose-game-btn" dark @click="selectGame('poker')">POKER</v-card>
              </v-col>
            </v-row>
          </template>
          <template  v-if="gameSelected && !gameStarted && !controls">
            <v-btn color="indigo" dark class="ma-2" @click="startGame()">START</v-btn>
            <v-btn color="teal" dark @click="">JOIN</v-btn>
          </template>
          <!-- <template  v-if="joinGame && !gameStarted">
            Available rooms:
          </template> -->

          <template v-if="gameStarted">
            <v-row class="main-row">
              <v-col cols="12">
                <v-card class="main-card">
                  <v-row>
                    <v-col cols="8" class="justify-center align-center text-center">
                      <!--<canvas width="700" height="700" style="border:1px solid #000000;"></canvas>-->
                      <table v-if="!gameOver" class="align-center justify-center text-center" cellspacing="0" cellpadding="0">
                        <tr v-for="y in gridSize" :key="y">
                          <td v-for="x in gridSize" :key="`${x}${y}`" class="grid-cell" :class="{
                            snake1: snakes[0] && snakes[0].body.includes(`${x} ${y}`), 
                            snake2: snakes[1] && snakes[1].body.includes(`${x} ${y}`),
                            snake3: snakes[2] && snakes[2].body.includes(`${x} ${y}`),
                            snake4: snakes[3] && snakes[3].body.includes(`${x} ${y}`),
                            food: food.includes(`${x} ${y}`)

                          }">
                          </td>
                        </tr>
                      </table>
                    </v-col>
                    <v-col cols="4" class="align-end">
                      <v-row>
                        <h3>Players ({{ numPlayers }})</h3>
                      </v-row>
                      <v-row>
                        <v-list>
                          <v-list-item
                            v-for="playerId in numPlayers"
                            :key="playerId"
                          >
                            <v-list-item-icon>
                              <v-icon :color="playerColors[playerId]">person</v-icon>
                            </v-list-item-icon>

                            <v-list-item-content>
                              <v-list-item-title>Player {{playerId}}</v-list-item-title>
                            </v-list-item-content>

                          </v-list-item>
                        </v-list>
                      </v-row>
                      <v-row>
                        <p>Scan the QR code to join the game:</p>
                      </v-row>
                      <v-row>
                        <img alt="QR code" :src="qrCodeUrl">
                      </v-row>
                      <v-row>
                        PEER ID: {{ peerId }}
                      </v-row>
                    </v-col>
                  </v-row>
                </v-card>
              </v-col>
            </v-row>
          </template>

          <template v-if="controls">
            <v-row class="main-row">
              <v-col cols="12">
                <!-- <v-card class="main-card"> -->
                  <v-row class="row-100" justify="center">
                    <v-col cols="4" justify="center">
                      <v-card class="control-card side-control" @click="move('L')">
                        <v-icon>
                          keyboard_arrow_left
                        </v-icon>
                      </v-card>
                    </v-col>
                    <v-col cols="4">
                      <v-row class="row-50">
                        <v-card class="control-card middle-control" @click="move('U')">
                            <v-icon>
                              keyboard_arrow_up
                            </v-icon>
                        </v-card>
                      </v-row>
                      <v-row class="row-50 mt-2">
                        <v-card class="control-card middle-control" @click="move('D')">  
                          <v-icon>
                            keyboard_arrow_down
                          </v-icon>
                        </v-card>
                      </v-row>
                    </v-col>
                    <v-col cols="4">
                      <v-card class="control-card side-control" @click="move('R')">  
                        <v-icon>
                          keyboard_arrow_right
                        </v-icon>
                      </v-card>
                    </v-col>
                  </v-row>
                <!-- </v-card> -->
              </v-col>
            </v-row>
          </template>

        </v-container>
      </v-app>
    </div>
</template>

<script>
import Peer from 'peerjs'

import Snake from './components/Snake.vue'

export default {
  name: 'app',
  components: {
    Snake
  },
  data: () => ({
    connection: null,
    connecting: true,
    message: '',
    messages: [],
    peer: null,
    peerId: '',
    opponentPeerId: '',
    //qrCodeUrl: '',
    gameSelected: false,
    gameStarted: false,
    gameOver: false,
    controls: false,
    selectedGame: '',
    grid: null, 
    intervals: [],

    numPlayers: 0,
    playerColors: ["#4DB6AC","#E57373","#64B5F6","#FFEE58"],
    snakes: [
      {
        body: ['3 5','3 4','3 3','3 2'],
        direction: 'D'
      },
      {
        body: ['38 38','38 37','38 36','38 35'],
        direction: 'U'
      },
      {
        body: ['3 38','4 38','5 38','6 38'],
        direction: 'R'
      },
      {
        body: ['38 3','37 3','36 3','35 3'],
        direction: 'L'
      },
    ],
    food: ['5 5', '15 4'],
    gridSize: 40
  }),
  created() {

    window.addEventListener('keydown', e => {
      switch (e.keyCode) {
        case 37: this.snakes[0].direction = 'L'
                 break
        case 38: this.snakes[0].direction = 'U'
                 break
        case 39: this.snakes[0].direction = 'R'
                 break
        case 40: this.snakes[0].direction = 'D'
                 break
      }
    });

    if (this.$route.query.peerid) {
      console.log('peerid', this.$route.query.peerid)
      this.opponentPeerId = this.$route.query.peerid
      this.joinGame()
    }
    // this.myPeer = new Peer();
    // this.myPeer.on('open', id => {
    //     console.log('My peer ID is: ' + id);
    //     //info('Connected at PeerJS server with success')
    //     this.myPeerId = id
    //     this.qrCodeUrl = "https://api.qrserver.com/v1/create-qr-code/?data=" + 'http://localhost:8080/' /*+ this.myPeerId*/ + "&size=200x200"
    //     this.connecting = false
    // })
    /* this.myPeer.on('call', remoteCall => {
        const getUserMedia = navigator.getUserMedia || navigator.webkitGetUserMedia || navigator.mozGetUserMedia
        getUserMedia({ audio: true, video: true }, stream => {
            remoteCall.answer(stream)
            remoteCall.on('stream', remoteStream => {
                const myVideo = document.querySelector('.js-callStream')
                if (myVideo) {
                    myVideo.srcObject = remoteStream
                    info('Connected with local Video and Audio')
                }
            })
        }, error => error(error))
    }) */
    // this.myPeer.on('connection', con => {
    //     this.connection = con
    //     con.on('open', () => {
    //         con.on('data', newMessage => this.messages.push(newMessage))
    //     })
    // })
  },
  computed: {
    qrCodeUrl: function() {
      return "https://api.qrserver.com/v1/create-qr-code/?data=" + location.host + '?peerid=' + this.peerId + "&size=200x200"
    },
  },
  watch: {
    peerId: function(old, neww) {
      console.log('change ', old, neww )
    }
  },
  methods: {
    initializePeer() {
      this.peer = new Peer()
      this.peer.on('open', id => {
        console.log('My peer ID is: ' + id);
        this.peerId = id
        console.log('My peer ID is: ' + this.peerId);
      })
      this.peer.on('error', function(err) {
        alert(''+err)
      })
    },

    startGame() {
      this.initializePeer()
      this.gameStarted = true
      console.log('dMy peer ID is: ' + this.peerId);
      this.peer.on('connection', c => {
        this.numPlayers += 1
        console.log("CONNECTIONNNNNN")
        if(this.connection) {
          c.close()
          return
        }
        this.connection = c

        this.connection.on('data', data => {
          console.log('something')
          switch(data[0]) {
            case 'moveLeft':
              this.snakes[0].direction = 'L'
              break
            case 'moveRight':
              this.snakes[0].direction = 'R'
              break
            case 'moveUp':
              this.snakes[0].direction = 'U'
              break
            case 'moveDown':
              this.snakes[0].direction = 'D'
              break
              //this.moveSnake(0,'D')
              //console.log('moveLeft')
          }
        })
        //turn = true
        //$('#game .alert p').text('Your move!')
        //begin()
      })
      //this.newGame();
      //this.grid = 0;

      this.snakes.forEach((snake,id) => {
        this.intervals[id] = setInterval(() => {
          this.moveSnake(id,snake.direction)
        }, 1000)    
      })

    },

    joinGame() {
        //this.initializePeer()
        this.controls = true
        this.peer = new Peer()
        console.log('opponenttt',this.opponentPeerId)
        this.peer.on('open', id => {
          this.peerId = id
          console.log('opponent',this.opponentPeerId)
          this.connection = this.peer.connect(this.$route.query.peerid, {
            reliable: true
          })
          this.connection.on('open', function() {
            //this.opponentPeerId = this.$route.query.peerid
            // $('#game .alert p').text("Waiting for opponent's move")
            // $('#game').show().siblings('section').hide()
            // turn = false
            // begin()
          })

          this.connection.on('data', data => {
            console.log('something')
            console.log(this.connection)
            switch(data[0]) {
              case 'moveLeft':
                console.log('moveLeft')
            }
          })
        })
        this.controls = true;
      },

      selectGame(game) {
        this.selectedGame = game
        this.gameSelected = true
      },

      //Moved the whole body of the snake
      moveSnake(snakeId, direction) {
        //console.log(snakeId, direction, this.snakes[snakeId].head, this.snakes[snakeId].body)
        const snakeBody = [...this.snakes[snakeId].body]

        if (Number(snakeBody[0].split(' ')[0]) === 1 && this.snakes[snakeId].direction === 'L' || 
            Number(snakeBody[0].split(' ')[0]) === this.gridSize && this.snakes[snakeId].direction === 'R' ||
            Number(snakeBody[0].split(' ')[1]) === 1 && this.snakes[snakeId].direction === 'U' || 
            Number(snakeBody[0].split(' ')[1]) === this.gridSize && this.snakes[snakeId].direction === 'D') {
              clearInterval(this.intervals[snakeId])
              this.blink(snakeId)
              return
        }
        this.snakes[snakeId].body[0] = this.moveHead(this.snakes[snakeId].body[0], direction)
        this.snakes[snakeId].body = this.snakes[snakeId].body.map((x,i,arr) => (i === 0) ? arr[0] : snakeBody[i - 1])
        this.snakes[snakeId].direction = direction

        if (this.food.includes(snakeBody[0])) {
          this.snakes[snakeId].body.push(snakeBody[snakeBody.length - 1])
          this.placeFood(this.food.indexOf(snakeBody[0]))
        }
        //console.log(snakeId, direction, this.snakes[snakeId].head, this.snakes[snakeId].body)
      },

      //Moves the head of the snake based on the direction
      moveHead(cell, direction) {
        const cellArr = cell.split(' ')
        switch (direction) {
          case 'D': return `${cellArr[0]} ${(Number(cellArr[1]) + 1)}`
          case 'U': return `${cellArr[0]} ${(Number(cellArr[1]) - 1)}`
          case 'L': return `${(Number(cellArr[0]) - 1)} ${cellArr[1]}`
          case 'R': return `${(Number(cellArr[0]) + 1)} ${cellArr[1]}`
        }
      },

      //Blinking after a snake hit a wall or another snake
      blink(snakeId) {
        //console.log('blinking',`.snake${snakeId + 1}`)
        const snake = document.querySelectorAll(`.snake${snakeId + 1}`);
        //console.log(snake)
        snake.forEach(cell => {
          //console.log('cell', cell)
          let blinks = 0;
          let intervalId = setInterval(() => {
            if (cell.style.visibility === 'hidden') {
                cell.style.visibility = 'visible';
                if (blinks++ === 2) {
                    clearInterval(intervalId);
                }
            } else {
                cell.style.visibility = 'hidden';
            }    
          }, 300);
        })
      },

      //Places food on a new position
      placeFood(foodInd) {
        let foodPlaced = false
        let x = 0
        let y = 0
        while (!foodPlaced) {
          x = Math.floor(Math.random() * this.gridSize) + 1
          y = Math.floor(Math.random() * this.gridSize) + 1
          foodPlaced = true
          this.snakes.forEach(snake => {
            if (snake.body.includes(`${x} ${y}`)){
              foodPlaced = false
            }
          })
        }
        this.food[foodInd] = `${x} ${y}`
      },

      move(direction) {
        console.log('mooove left')
        switch (direction) {
          case 'L': this.connection.send(['moveLeft']); break;
          case 'R': this.connection.send(['moveRight']); break;
          case 'U': this.connection.send(['moveUp']); break;
          case 'D': this.connection.send(['moveDown']); break;
        }
      }


  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  /*margin-top: 60px;*/
}

.main-card, .main-row {
  height: 100%
}

.main-container {
  height: 100vh;
}

table {
  background-color: #263238;
}

td {
  width: 15px;
  height: 15px;
  background-color: #263238;
}

.snake1 {
  background-color: #4DB6AC !important;
}

.snake2 {
  background-color: #E57373 !important;
}

.snake3 {
  background-color: #64B5F6 !important;
}

.snake4 {
  background-color: #FFEE58 !important;
}

.food {
  background-color: #EC407A;
}

table {
  margin: auto;
}

.control-card {
  height: 100%;
}

.row-100 {
  height: 100%;
}

.row-50 {
  height: 50%;
}

.control-card i {
  font-size: 20vw !important;
}

.middle-control {
  width: 100%;
}

.side-control {
  height: 50%;
  margin-top: 37.5%;
}

.choose-game-btn {
  height:200px;
  padding:5%;
  font-size:5vw;
}


</style>

