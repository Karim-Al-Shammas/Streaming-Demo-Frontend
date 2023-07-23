<template>
  <div>
    <div ref="output"></div>
  </div>
</template>

<script>

import axios from 'axios';

export default {
  mounted() {
    (async () => {

      let res = await fetch('https://stream-test-eus.azurewebsites.net/api/ClientTokenGenerator?');
      let data = await res.json();
      console.log(data)


      let ws = new WebSocket(data.url, 'json.webpubsub.azure.v1');
      let ackId = 0;
      ws.onopen = () => {
        console.log('connected');
        ws.send(
          JSON.stringify({
            type: 'joinGroup',
            group: data.user_id,
            ackId: ++ackId,
          })
        );
      };

      const body = {
        query: 'Can you give me a way to connect with Invest Qatar?',
        websocket_client_url: data.url,
        websocket_user_id: data.user_id
      }

      //axios.post('http://localhost:7071/api/Stream-Backend', JSON.stringify('tell me a story'), {
      axios.post('http://localhost:7071/api/BotQnAHTTPFunc', JSON.stringify(body), {
          headers: {'Content-Type': 'application/json'},
        })
        .then((response) => console.log('Response:' + response.data));
      console.log('Hello!');


      ws.onmessage = (event) => {
        let message = JSON.parse(event.data);
        console.log(message);
        if (message.type === 'message' && message.group === data.user_id) {
          let d = document.createElement('span');
          d.innerText = message.data;
          this.$refs.output.appendChild(d);
          window.scrollTo(0, document.body.scrollHeight);
        }
      };


      ws.onclose = (event) => {
        console.log('connection closed');
      };


    })();
  },
};
</script>
