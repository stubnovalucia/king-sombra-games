<template>
    <div id="app">
      <v-app>
        <div v-if="!gameSelected">
          <v-btn color="deep-purple" class="ma-2" dark @click="selectGame('snake')">SNAKE</v-btn>
          <v-btn color="red darken-4" dark @click="selectGame('poker')">POKER</v-btn>
        </div>
        <div  v-if="gameSelected && !gameStarted">
          <v-btn color="indigo" dark class="ma-2" @click="startGame()">START</v-btn>
          <v-btn color="teal" dark @click="">JOIN</v-btn>
        </div>
        <div  v-if="gameStarted">
          Available rooms:
        </div>
        <div v-if="gameStarted">
          <canvas width="700" height="500" style="border:1px solid #000000;"></canvas>
          <img alt="Vue logo" :src="qrCodeUrl">
          <br>
          PEER ID: {{ peerId }}
        </div>
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
    selectedGame: ''
  }),
  created() {
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
    }
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
      this.peer.on('connection', function(c) {
        console.log("CONNECTIONNNNNN")
        if(this.connection) {
          c.close()
          return
        }
        this.connection = c
        //turn = true
        //$('#game .alert p').text('Your move!')
        //begin()
      })
    },

    joinGame() {
        //this.initializePeer()
        this.peer = new Peer()
        console.log('opponenttt',this.opponentPeerId)
        this.peer.on('open', id => {
          this.peerId = id
          console.log('opponent',this.opponentPeerId)
          this.connection = this.peer.connect(this.opponentPeerId, {
            reliable: true
          })
          this.connection.on('open', function() {
            //this.opponentPeerId = this.$route.query.peerid
            // $('#game .alert p').text("Waiting for opponent's move")
            // $('#game').show().siblings('section').hide()
            // turn = false
            // begin()
          })
        })
      },

      selectGame(game) {
        this.selectedGame = game
        this.gameSelected = true
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
  margin-top: 60px;
}
</style>
