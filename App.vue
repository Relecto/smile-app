<template>
  <view v-if="connected" class="container">
    <view class="alyss">
      <TextInput 
        :containerStyle="{ backgroundColor: '#eeeeee' }" 
        :style="{ width: '70%' }" 
        :onFocus="() => showAutocomplete = true"
        :onEndEditing="() => showAutocomplete = false"
        placeholder="Enter your message" 
        class="text-input mrgn" 
        v-model="message" 
      />
      <button type="clear" :icon="{ name: 'comment', type: 'font-awesome', size: 30 }" class="row-item mrgn" :on-press="sound"  />  

    </view>

    <view v-if="showAutocomplete" class="autocomplete mrgn">
      <list-item 
        :titleStyle="{ fontSize: 16 }" 
        :containerStyle="{ backgroundColor: '#eeeeee' }" 
        v-for="(s, i) in filteredPhrases" 
        :key="i" :title="s" 
        :onPress="() => message = s"
        
        />
    </view>

    <view class="spacer">

    </view> 

    <view class="row">
      <button type="clear" :icon="{ name: 'arrow-up', type: 'font-awesome', size: 80 }" class="row-item mrgn" :on-press="up" />
    </view>
    <view class="row">
      <button type="clear" :icon="{ name: 'arrow-left', type: 'font-awesome', size: 80 }" class="row-item mrgn" :on-press="left"  />
      <button type="clear" :icon="{ name: 'stop', type: 'font-awesome', size: 80 }" class="row-item mrgn" :on-press="stop"></button>
      <button type="clear" :icon="{ name: 'arrow-right', type: 'font-awesome', size: 80 }" class="row-item mrgn" :on-press="right"></button>
    </view>
    <view class="row">
      <button type="clear" :icon="{ name: 'arrow-down', type: 'font-awesome', size: 80 }" class="row-item mrgn" :on-press="down"></button>
    </view>
    
    <view class="spacer">

    </view>

    <view class="row">
      <button :type="emotion == 'sad' ? 'solid' : 'outline'" :titleStyle="{ fontSize: 40 }" class="row-item mrgn" :on-press="sad" title="😢"/>
      <button :type="emotion == 'angry' ? 'solid' : 'outline'" :titleStyle="{ fontSize: 40 }" class="row-item mrgn" :on-press="angry" title="😠" />
      <button :type="emotion == 'neutral' ? 'solid' : 'outline'" :titleStyle="{ fontSize: 40 }" class="row-item mrgn" :on-press="neutral" title="😑"></button>
      <button :type="emotion == 'happy' ? 'solid' : 'outline'" :titleStyle="{ fontSize: 40 }" class="row-item mrgn" :on-press="happy" title="😄"></button>
    </view>
    
    <text>{{test}}</text>
  </view>

  

  <view v-else class="center">
    <activity-indicator :animating="connecting" size="large" color="#0000ff" />
    <text v-if="connecting" class="heading">Connecting...</text>
    <button v-if="!connecting" :on-press="connect" title="Try again"></button>
  </view>
</template>
 

<script>
// import { Player, Recorder, MediaStates } from '@react-native-community/audio-toolkit'
import { Button } from 'react-native-elements'
import Icon from 'react-native-vector-icons/FontAwesome'
import { Input, ListItem } from 'react-native-elements'

import BluetoothSerial from 'react-native-bluetooth-serial'

import Tts from 'react-native-tts';

Tts.setDefaultLanguage('ru-RU');


// import EasyBluetooth from 'easy-bluetooth-classic'


var Sound = require("react-native-sound")

Sound.setCategory("Playback")

function toString(buffer) {
  var byteArray = new Uint8Array(buffer)
  var byteString = ""
  for (var i = 0; i < byteArray.byteLength; i++) {
    byteString += String.fromCodePoint(byteArray[i])
  }
  return byteString
}



export default {
  components: {
    Button, Icon, Input, ListItem
  },
  data() {
    return {
      message: "",
      test: "",
      connected: false,
      connecting: true,
      host: "128.199.47.222",
      port: 2580,
      emotion: 'happy',
      showAutocomplete: false,
      phrases: [
        "Привет",
        "Приветствую",
        "Моё почтение",
        "Отлично, ты  как?",
        "И тебе не стыдно?",
        "С тобой общаюсь, как слышишь",
        "Пойдём?"
      ]
    }
  },
  computed: {
    voiceEmotion() {
      switch (this.emotion) {
        case 'happy':
          return 'good'
          break
        case 'neutral':
          return 'neutral'
          break
        case 'sad':
          return 'neutral'
          break
        case 'angry':
          return 'evil'
          break
      }
    },
    filteredPhrases() {
      return this.phrases.filter(val => val.toLowerCase().search(this.message.toLowerCase()) != -1)
    }
  },
  methods: {
    up() {
      this.$conn.send("A")
    },
    down() {
      this.$conn.send("D")
    },
    left() {
      this.$conn.send("B")
    },
    right() {
      this.$conn.send("C")
    },
    stop() {
      this.$conn.send("E")
    },
    sad() {
      this.emotion = 'sad'
      this.$conn.send("K")
    },
    neutral() {
      this.emotion = 'neutral'
      this.$conn.send("G")
    },
    happy() {
      this.emotion = 'happy'
      this.$conn.send("F")
    },
    angry() {
      this.emotion = 'angry'
      this.$conn.send("H")
    },
    sound() {
      if (!this.message) {
        return
      }
      Tts.speak(this.message)
      // let audio = new Sound(
      //   `http://${this.host}:1234/mp3?text=${encodeURIComponent(this.message)}&emotion=${this.voiceEmotion}`,
      //   "",
      //   err => {
      //     if (err) {
      //       alert(err.message)
      //       return
      //     }
      //     audio.play()
      //   }
      // )
    },
    async connect() {
      // const ws = new WebSocket(`ws://${this.host}:8181`)

      // ws.onopen = () => {
      //   this.connected = true
      // }
      // ws.onclose = e => {
      //   this.connected = false
      //   this.$conn = null
      // }
      // ws.onerror = e => {
      //   alert(e.message)
      // }
      // ws.onmessage = e => {
      //   let data = toString(e.data)
      //   alert(data)
      //   this.test = data
      // }

      // this.$conn = ws

      let config = {
      "uuid": "00001101-0000-1000-8000-00805f9b34fb",
      "deviceName": "Bluetooth Example  Project",
      "bufferSize": 1024,
      "characterDelimiter": "\n"
      }

      this.connecting = true

      let devices = await BluetoothSerial.list()

      // devices = [...devices, ...await BluetoothSerial.discoverUnpairedDevices()]

      let robot = devices.find(device => device.name == 'HC-06')
      if (!robot) {
        alert('SMILE not found!\n' + JSON.stringify(devices))
        this.connecting = false
        return 
      }

      BluetoothSerial.connect(robot.id)

      // try {
      //   let res = await EasyBluetooth.init(config)

      //   // EasyBluetooth.startScan()
      //   // EasyBluetooth.addOnDeviceFoundListener(device => alert(device))

      //   let devices = await EasyBluetooth.startScan()
        
      //   let robot = devices.find(device => device.name == 'HC-06')
      //   if (!robot) {
      //     alert('SMILE not found!\n' + JSON.stringify(devices))
      //     this.connecting = false
      //     return
      //   }

      //   await EasyBluetooth.connect(robot)

      //   EasyBluetooth.addOnStatusChangeListener(status => {
      //     if (status == 'NONE') {
      //       this.connecting = false
      //       this.connected = false
      //     }
      //     alert(JSON.stringify(status))
      //   })
      // } catch (e) {
      //   alert(JSON.stringify(e))
      //   this.connecting = false

      //   return
      // }

      alert('Connected')
      this.connecting = false

      this.connected = true
      
      this.$conn = {
        // send(data) {
        //   return EasyBluetooth.writeln(data)
        // }

        send(data) {
          BluetoothSerial.write(data)
            // .then(alert)
            .catch(alert)
        }
      }
    }
  },
  mounted() {
    this.connect()
  }
};
</script>


<style>



.autocomplete {
  position: absolute;
  left: 0; top: 40px; right: 0; bottom: 0;
  z-index: 3;
}

.mrgn {
  margin: 10px 10px;
}

.container {
  background-color: white;
  align-items: center;
  align-content: flex-start;
  flex-direction: column;
  height: 100%;
  margin-top: 55px;
  padding-bottom: 100px;
}
.center {
  flex: 1;
  align-items: center;
  justify-content: center;
}

.list-item {
  background-color: #eeeeee;
  /* font-size: 18px; */
}


.alyss {
  display: flex;
  /* align-self: flex-start; */
  /* justify-self: flex-start; */
  flex-direction: row;
  align-items: center;

}

.alyss-item {
  display: flex;
  /* flex-grow: 1; */
}


.row {
  display: flex;
  flex-direction: row;
  align-items: center;
}

.row-item {
  display: flex;
  flex-grow: 1;
}

.spacer {
  flex-grow: 1;
  flex: 1;
}

.heading {
  font-size: 18px;
  margin: 8px;
}

.text-color-primary {
  color: blue;
}
</style>
