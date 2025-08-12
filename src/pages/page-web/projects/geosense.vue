<template>
    <div class="flex-column flex-justify-start flex-align-center">
      <router-link
        style="width: 90%; margin: auto;"
        v-for="item in options"
        :key="item.key"
        :to="item.path"
        :class="{
          'menu-item': true,
        }"
      >
      <a-button
        class="mt10"
        style="width:100%;"
        type="primary"
        @click="selectLivestream(item.routeName)"
        >{{ item.label }}</a-button
      >
      </router-link>
    </div>
    <div class="live" v-if="showLive" v-drag-window>
      <div style="height: 40px; width: 100%" class="drag-title"></div>
      <a style="position: absolute; right: 10px; top: 10px; font-size: 16px; color: white;" @click="() => root.$router.push('/' + ERouterName.LIVESTREAM)"><CloseOutlined /></a>
      <router-view :name="routeName" />
    </div>
    <div class="mqtt-container">
        <h2>📡 MQTT Subscriber</h2>
        <p>Listening to: <strong>{{ topic }}</strong></p>

        <div class="message-box">
        <h3>📥 Received Messages</h3>
        <ul>
            <li v-for="(msg, index) in receivedMessages" :key="index">
            {{ msg }}
            </li>
        </ul>
        </div>
    </div>
  </template>

<script setup lang="ts">
import { onMounted, onUnmounted, ref } from 'vue'
import mqtt, { MqttClient } from 'mqtt'

// MQTT Broker URL (WebSocket required for Vue in browsers)
const brokerUrl: string = 'localhost:1883'
const topic: string = 'thing/product/1ZNPM56001E014/osd'

// Reactive state variables
const receivedMessages = ref<string[]>([]) // Stores received messages
let client: MqttClient | null = null

// Function to connect to MQTT and subscribe to the topic
const connectToMqtt = () => {
  client = mqtt.connect(brokerUrl)

  client.on('connect', () => {
    receivedMessages.value.push('✅ Connected to MQTT broker')
    client?.subscribe(topic, (err) => {
      if (!err) {
        receivedMessages.value.push(`✅ Subscribed to topic: ${topic}`)
      } else {
        receivedMessages.value.push('❌ Subscription failed: ' + err.message)
      }
    })
  })

  client.on('message', (receivedTopic, message) => {
    if (receivedTopic === topic) {
      const decodedMessage = message.toString()
      receivedMessages.value.push(`📩 ${decodedMessage}`)
    }
  })

  client.on('error', (error) => {
    receivedMessages.value.push('❌ MQTT Error: ' + error.message)
  })
}

// Function to disconnect when the component is destroyed
const disconnectMqtt = () => {
  if (client) {
    client.end()
    receivedMessages.value.push('⚠️ Disconnected from MQTT broker')
  }
}

// Connect when the component is mounted and disconnect when unmounted
onMounted(connectToMqtt)
onUnmounted(disconnectMqtt)
</script>

<style scoped>
.mqtt-container {
  border: 2px solid #4CAF50;
  padding: 16px;
  border-radius: 8px;
  background-color: #f4fff4;
  width: 400px;
  font-family: Arial, sans-serif;
}

h2 {
  color: #333;
}

.message-box {
  background: #fff;
  border-radius: 6px;
  padding: 10px;
  box-shadow: 0px 2px 5px rgba(0, 0, 0, 0.1);
  margin-top: 10px;
}

ul {
  list-style: none;
  padding: 0;
}

li {
  padding: 6px;
  border-bottom: 1px solid #ddd;
}

li:last-child {
  border-bottom: none;
}
</style>
