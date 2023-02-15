<template>
  <div id="app">
    매장번호:
    <input v-model="strodId" type="text" />
    <button type="button" @click="connect">접속</button>
    <br />
    전송매장번호:
    <input v-model="targetStrodId" type="text" />
    <button type="button" @click="send">보내기</button>
    <div v-for="(item, idx) in recvList" :key="idx">
      <h3>유저이름: {{ item.orderId }}</h3>
      <h3>내용: {{ item.message }}</h3>
    </div>
  </div>
</template>

<script>
import Stomp from 'webstomp-client'
import SockJS from 'sockjs-client'

export default {
  name: 'App',
  auth: false, // login안된 상태에서도 사용가능 페이지
  data() {
    return {
      strodId: '1',
      targetStrodId: '',
      recvList: [],
    }
  },
  created() {
    // App.vue가 생성되면 소켓 연결을 시도합니다.
  },
  methods: {
    send() {
      console.log('Send message:' + this.message)
      const msg = {
        orderId: '222',
        content: 'Direct message',
      }
      this.stompClient.send(
        `/users/${this.targetStrodId}/queue/messages`,
        JSON.stringify(msg),
        {}
      )
    },
    connect() {
      const serverURL = 'http://localhost:3002/webSocket'
      const socket = new SockJS(serverURL)
      this.stompClient = Stomp.over(socket)
      // console.log(`소켓 연결을 시도합니다. 서버 주소: ${serverURL}`)
      const headers = { username: this.strodId, test: 'zzzz' }
      this.stompClient.connect(
        headers,
        (frame) => {
          // 소켓 연결 성공
          this.connected = true
          console.log('소켓 연결 성공', frame)
          // 서버의 메시지 전송 endpoint를 구독합니다.
          // 이런형태를 pub sub 구조라고 합니다.
          this.stompClient.subscribe('/customer/estory_uuid', (res) => {
            console.log('구독으로 받은 메시지 입니다.', res.body)

            // 받은 데이터를 json으로 파싱하고 리스트에 넣어줍니다.
            this.recvList.push(JSON.parse(res.body))
          })

          this.stompClient.subscribe('/topic', (res) => {
            console.log('구독으로 받은 메시지 입니다.', res.body)

            // 받은 데이터를 json으로 파싱하고 리스트에 넣어줍니다.
            this.recvList.push(JSON.parse(res.body))
          })
        },
        (error) => {
          // 소켓 연결 실패
          console.log('소켓 연결 실패', error)
          this.connected = false
        }
      )
    },
  },
}
</script>
