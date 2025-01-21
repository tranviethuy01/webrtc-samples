# Hướng dẫn của Việt Huy

1/ Caller + callee đều tham gia socket 
2/ Caller bấm nút gọi
=> makeCall
=> createPeerConnection : tạo peerConnection , trong này tạo RTCPeerConnection, đồng thời tạo listening cho 2 sự kiện là pc.onicecandidate , pc.ontrack, và tiến hành addTrack: localStream.getTracks().forEach(track => pc.addTrack(track, localStream));
=> tạo offer với   4   const offer = await pc.createOffer();
=> tiến hành gửi offer tới server WebSocket với signaling.postMessage({type: 'offer', sdp: offer.sdp});
=> Tiến hành setLocalDescription cho peerconnection với:  pc.setLocalDescription(offer)

3/ Callee nhận offer và xử lý
=> nhận được 1 message type là offer 
=> createPeerConnection : tạo peerConnection , trong này tạo RTCPeerConnection, đồng thời tạo listening cho 2 sự kiện là pc.onicecandidate , pc.ontrack, và tiến hành addTrack: localStream.getTracks().forEach(track => pc.addTrack(track, localStream));

=> tiến hành setRemoteDescription với offer nhận được với  await pc.setRemoteDescription(offer);
=> tiến hành tạo một answer với pc.createAnswer
=> gửi message answer tới WebSocket  với câu lệnh signaling.postMessage({type: 'answer', sdp: answer.sdp});
=> tiến hành setLocalDescription với thông tin answer vừa tạo được:  await pc.setLocalDescription(answer);


4/ Caller nhận answer và xử lý
=> nhận được 1 message với type là answer
=> tiến hành setRemoteDescription với thông tin answer nhận được :  await pc.setRemoteDescription(answer);
=> không gửi dữ liệu gì nữa cả, đến đây là kết nối được


Ghi chú:
Chúng ta nói về sự kiện onicecandidate và message `candidate`
message `candidate` mà chúng ta nhận được là do trong quá trình tạo peerConnection với createPeerConnection, tiến hành tạo một pc = new RTCPeerConnection() , sau đó tạo 2 sự kiện listening là onicecandidate và ontrack
Trong onicecandidate, thì mỗi lần có sự kiện này, chúng ta lại tiến hành gửi thông tin đi qua WebSocket với  signaling.postMessage(message);
=> đây là lý do chúng ta nhận được các message về candidate thông qua WebSocket


```

function createPeerConnection() {
  console.log("function createPeerConnection");
  pc = new RTCPeerConnection();
  pc.onicecandidate = e => {
    console.log("pc.onicecandidate with event e", e);
    console.log("e.candidate", e.candidate);
    const message = {
      type: 'candidate',
      candidate: null,
    };
    if (e.candidate) {
      message.candidate = e.candidate.candidate;
      message.sdpMid = e.candidate.sdpMid;
      message.sdpMLineIndex = e.candidate.sdpMLineIndex;
    }
    signaling.postMessage(message);
  };
  pc.ontrack = e => 
  {
    console.log("pc.ontrack with event e", e);
    remoteVideo.srcObject = e.streams[0];
  }
  localStream.getTracks().forEach(track => pc.addTrack(track, localStream));
}



```








Một đoạn log

Log của caller

```
signaling got message ready
main.js?v=1.0.3:24 data {type: 'ready'}
main.js?v=1.0.3:111 function makeCall
main.js?v=1.0.3:86 function createPeerConnection
main.js?v=1.0.3:89 pc.onicecandidate with event e RTCPeerConnectionIceEvent {isTrusted: true, candidate: RTCIceCandidate, type: 'icecandidate', target: RTCPeerConnection, currentTarget: RTCPeerConnection, …}
main.js?v=1.0.3:90 e.candidate RTCIceCandidate {candidate: 'candidate:1526993604 1 udp 2122194687 192.168.1.2 …eration 0 ufrag Z286 network-id 1 network-cost 10', sdpMid: '0', sdpMLineIndex: 0, foundation: '1526993604', component: 'rtp', …}
main.js?v=1.0.3:89 pc.onicecandidate with event e RTCPeerConnectionIceEvent {isTrusted: true, candidate: RTCIceCandidate, type: 'icecandidate', target: RTCPeerConnection, currentTarget: RTCPeerConnection, …}
main.js?v=1.0.3:90 e.candidate RTCIceCandidate {candidate: 'candidate:3412597694 1 udp 2122262783 2402:800:629…eration 0 ufrag Z286 network-id 2 network-cost 10', sdpMid: '0', sdpMLineIndex: 0, foundation: '3412597694', component: 'rtp', …}
main.js?v=1.0.3:89 pc.onicecandidate with event e RTCPeerConnectionIceEvent {isTrusted: true, candidate: RTCIceCandidate, type: 'icecandidate', target: RTCPeerConnection, currentTarget: RTCPeerConnection, …}
main.js?v=1.0.3:90 e.candidate RTCIceCandidate {candidate: 'candidate:1526993604 1 udp 2122194687 192.168.1.2 …eration 0 ufrag Z286 network-id 1 network-cost 10', sdpMid: '1', sdpMLineIndex: 1, foundation: '1526993604', component: 'rtp', …}
main.js?v=1.0.3:89 pc.onicecandidate with event e RTCPeerConnectionIceEvent {isTrusted: true, candidate: RTCIceCandidate, type: 'icecandidate', target: RTCPeerConnection, currentTarget: RTCPeerConnection, …}
main.js?v=1.0.3:90 e.candidate RTCIceCandidate {candidate: 'candidate:3412597694 1 udp 2122262783 2402:800:629…eration 0 ufrag Z286 network-id 2 network-cost 10', sdpMid: '1', sdpMLineIndex: 1, foundation: '3412597694', component: 'rtp', …}
main.js?v=1.0.3:23 signaling got message answer
main.js?v=1.0.3:24 data {type: 'answer', sdp: 'v=0\r\no=- 8039228528536995690 2 IN IP4 127.0.0.1\r\ns…c4SYi\r\na=ssrc:1386196955 cname:Fkr7UrlDdvxc4SYi\r\n'}
main.js?v=1.0.3:134 function handleAnswer {type: 'answer', sdp: 'v=0\r\no=- 8039228528536995690 2 IN IP4 127.0.0.1\r\ns…c4SYi\r\na=ssrc:1386196955 cname:Fkr7UrlDdvxc4SYi\r\n'}
main.js?v=1.0.3:104 pc.ontrack with event e RTCTrackEvent {isTrusted: true, receiver: RTCRtpReceiver, track: MediaStreamTrack, streams: Array(1), transceiver: RTCRtpTransceiver, …}
main.js?v=1.0.3:104 pc.ontrack with event e RTCTrackEvent {isTrusted: true, receiver: RTCRtpReceiver, track: MediaStreamTrack, streams: Array(1), transceiver: RTCRtpTransceiver, …}
main.js?v=1.0.3:23 signaling got message candidate
main.js?v=1.0.3:24 data {type: 'candidate', candidate: 'candidate:1544714748 1 udp 2122194687 192.168.1.2 …eration 0 ufrag u1pd network-id 1 network-cost 10', sdpMid: '0', sdpMLineIndex: 0}
main.js?v=1.0.3:143 function handleCandidate {type: 'candidate', candidate: 'candidate:1544714748 1 udp 2122194687 192.168.1.2 …eration 0 ufrag u1pd network-id 1 network-cost 10', sdpMid: '0', sdpMLineIndex: 0}
main.js?v=1.0.3:23 signaling got message candidate
main.js?v=1.0.3:24 data {type: 'candidate', candidate: 'candidate:4028547144 1 udp 2122262783 2402:800:629…eration 0 ufrag u1pd network-id 2 network-cost 10', sdpMid: '0', sdpMLineIndex: 0}
main.js?v=1.0.3:143 function handleCandidate {type: 'candidate', candidate: 'candidate:4028547144 1 udp 2122262783 2402:800:629…eration 0 ufrag u1pd network-id 2 network-cost 10', sdpMid: '0', sdpMLineIndex: 0}
main.js?v=1.0.3:23 signaling got message candidate
main.js?v=1.0.3:24 data {type: 'candidate', candidate: null}
main.js?v=1.0.3:143 function handleCandidate {type: 'candidate', candidate: null}
main.js?v=1.0.3:89 pc.onicecandidate with event e RTCPeerConnectionIceEvent {isTrusted: true, candidate: null, type: 'icecandidate', target: RTCPeerConnection, currentTarget: RTCPeerConnection, …}
main.js?v=1.0.3:90 e.candidate null



```


Log của callee

```
signaling got message ready
main.js?v=1.0.3:24 data {type: 'ready'}
main.js?v=1.0.3:26 not ready yet
main.js?v=1.0.3:23 signaling got message offer
main.js?v=1.0.3:24 data {type: 'offer', sdp: 'v=0\r\no=- 6835999877517266799 2 IN IP4 127.0.0.1\r\ns…16352f42c9 0a4dbe85-f2a0-46bc-b2a4-225726604836\r\n'}
main.js?v=1.0.3:120 function handleOffer {type: 'offer', sdp: 'v=0\r\no=- 6835999877517266799 2 IN IP4 127.0.0.1\r\ns…16352f42c9 0a4dbe85-f2a0-46bc-b2a4-225726604836\r\n'}
main.js?v=1.0.3:86 function createPeerConnection
main.js?v=1.0.3:23 signaling got message candidate
main.js?v=1.0.3:24 data {type: 'candidate', candidate: 'candidate:1526993604 1 udp 2122194687 192.168.1.2 …eration 0 ufrag Z286 network-id 1 network-cost 10', sdpMid: '0', sdpMLineIndex: 0}
main.js?v=1.0.3:143 function handleCandidate {type: 'candidate', candidate: 'candidate:1526993604 1 udp 2122194687 192.168.1.2 …eration 0 ufrag Z286 network-id 1 network-cost 10', sdpMid: '0', sdpMLineIndex: 0}
main.js?v=1.0.3:23 signaling got message candidate
main.js?v=1.0.3:24 data {type: 'candidate', candidate: 'candidate:3412597694 1 udp 2122262783 2402:800:629…eration 0 ufrag Z286 network-id 2 network-cost 10', sdpMid: '0', sdpMLineIndex: 0}
main.js?v=1.0.3:143 function handleCandidate {type: 'candidate', candidate: 'candidate:3412597694 1 udp 2122262783 2402:800:629…eration 0 ufrag Z286 network-id 2 network-cost 10', sdpMid: '0', sdpMLineIndex: 0}
main.js?v=1.0.3:23 signaling got message candidate
main.js?v=1.0.3:24 data {type: 'candidate', candidate: 'candidate:1526993604 1 udp 2122194687 192.168.1.2 …eration 0 ufrag Z286 network-id 1 network-cost 10', sdpMid: '1', sdpMLineIndex: 1}
main.js?v=1.0.3:143 function handleCandidate {type: 'candidate', candidate: 'candidate:1526993604 1 udp 2122194687 192.168.1.2 …eration 0 ufrag Z286 network-id 1 network-cost 10', sdpMid: '1', sdpMLineIndex: 1}
main.js?v=1.0.3:23 signaling got message candidate
main.js?v=1.0.3:24 data {type: 'candidate', candidate: 'candidate:3412597694 1 udp 2122262783 2402:800:629…eration 0 ufrag Z286 network-id 2 network-cost 10', sdpMid: '1', sdpMLineIndex: 1}
main.js?v=1.0.3:143 function handleCandidate {type: 'candidate', candidate: 'candidate:3412597694 1 udp 2122262783 2402:800:629…eration 0 ufrag Z286 network-id 2 network-cost 10', sdpMid: '1', sdpMLineIndex: 1}
main.js?v=1.0.3:104 pc.ontrack with event e RTCTrackEvent {isTrusted: true, receiver: RTCRtpReceiver, track: MediaStreamTrack, streams: Array(1), transceiver: RTCRtpTransceiver, …}
main.js?v=1.0.3:104 pc.ontrack with event e RTCTrackEvent {isTrusted: true, receiver: RTCRtpReceiver, track: MediaStreamTrack, streams: Array(1), transceiver: RTCRtpTransceiver, …}
main.js?v=1.0.3:89 pc.onicecandidate with event e RTCPeerConnectionIceEvent {isTrusted: true, candidate: RTCIceCandidate, type: 'icecandidate', target: RTCPeerConnection, currentTarget: RTCPeerConnection, …}
main.js?v=1.0.3:90 e.candidate RTCIceCandidate {candidate: 'candidate:1544714748 1 udp 2122194687 192.168.1.2 …eration 0 ufrag u1pd network-id 1 network-cost 10', sdpMid: '0', sdpMLineIndex: 0, foundation: '1544714748', component: 'rtp', …}
main.js?v=1.0.3:89 pc.onicecandidate with event e RTCPeerConnectionIceEvent {isTrusted: true, candidate: RTCIceCandidate, type: 'icecandidate', target: RTCPeerConnection, currentTarget: RTCPeerConnection, …}
main.js?v=1.0.3:90 e.candidate RTCIceCandidate {candidate: 'candidate:4028547144 1 udp 2122262783 2402:800:629…eration 0 ufrag u1pd network-id 2 network-cost 10', sdpMid: '0', sdpMLineIndex: 0, foundation: '4028547144', component: 'rtp', …}
main.js?v=1.0.3:89 pc.onicecandidate with event e RTCPeerConnectionIceEvent {isTrusted: true, candidate: null, type: 'icecandidate', target: RTCPeerConnection, currentTarget: RTCPeerConnection, …}
main.js?v=1.0.3:90 e.candidate null
main.js?v=1.0.3:23 signaling got message candidate
main.js?v=1.0.3:24 data {type: 'candidate', candidate: null}
main.js?v=1.0.3:143 function handleCandidate {type: 'candidate', candidate: null}


```


# Một hướng dẫn khác

Setting up WebRTC involves multiple steps to establish a peer-to-peer (P2P) connection between two clients for real-time audio, video, or data communication. Below is the full flow to implement WebRTC:

---

## **1. Set Up the Signaling Server**
WebRTC requires a signaling server to exchange connection information between peers. This is not part of the WebRTC protocol but is essential to share data like SDP (Session Description Protocol) and ICE candidates.

- Use a technology like **WebSocket**, **Socket.IO**, or HTTP polling to create a signaling server.
- The server facilitates communication but does not process media streams.

### Example (Node.js + WebSocket):
```javascript
const WebSocket = require('ws');
const server = new WebSocket.Server({ port: 8080 });

server.on('connection', (socket) => {
  socket.on('message', (message) => {
    // Broadcast the message to all connected clients
    server.clients.forEach((client) => {
      if (client !== socket && client.readyState === WebSocket.OPEN) {
        client.send(message);
      }
    });
  });
});
```

---

## **2. Create Peer Connections**
Each client must:
1. Create an `RTCPeerConnection` instance.
2. Configure it with appropriate STUN/TURN servers to handle NAT traversal.

### Example:
```javascript
const configuration = {
  iceServers: [
    { urls: 'stun:stun.l.google.com:19302' }, // Free public STUN server
    {
      urls: 'turn:your-turn-server.com:3478',
      username: 'user',
      credential: 'password',
    },
  ],
};
const peerConnection = new RTCPeerConnection(configuration);
```

---

## **3. Capture Media Streams (Optional)**
Use the `getUserMedia` API to capture audio and video streams.

### Example:
```javascript
navigator.mediaDevices.getUserMedia({ video: true, audio: true })
  .then((stream) => {
    // Display the local video
    const videoElement = document.querySelector('#localVideo');
    videoElement.srcObject = stream;

    // Add tracks to the peer connection
    stream.getTracks().forEach((track) => {
      peerConnection.addTrack(track, stream);
    });
  })
  .catch((error) => console.error('Error accessing media devices:', error));
```

---

## **4. Exchange SDP Offer/Answer**
- **Caller (Initiator):**
  1. Create an SDP offer using `peerConnection.createOffer()`.
  2. Set the offer as the local description using `peerConnection.setLocalDescription()`.
  3. Send the offer to the signaling server.

- **Callee (Receiver):**
  1. Receive the offer via the signaling server.
  2. Set the offer as the remote description using `peerConnection.setRemoteDescription()`.
  3. Create an SDP answer using `peerConnection.createAnswer()`.
  4. Set the answer as the local description and send it to the signaling server.

### Example (Caller):
```javascript
peerConnection.createOffer()
  .then((offer) => {
    return peerConnection.setLocalDescription(offer);
  })
  .then(() => {
    signalingServer.send(JSON.stringify({ type: 'offer', sdp: peerConnection.localDescription }));
  });
```

### Example (Callee):
```javascript
signalingServer.onmessage = (message) => {
  const data = JSON.parse(message);

  if (data.type === 'offer') {
    peerConnection.setRemoteDescription(new RTCSessionDescription(data.sdp))
      .then(() => peerConnection.createAnswer())
      .then((answer) => peerConnection.setLocalDescription(answer))
      .then(() => {
        signalingServer.send(JSON.stringify({ type: 'answer', sdp: peerConnection.localDescription }));
      });
  }
};
```

---

## **5. Exchange ICE Candidates**
- Both peers exchange ICE candidates via the signaling server.
- These candidates help establish the most efficient route for P2P communication.

### Example:
```javascript
peerConnection.onicecandidate = (event) => {
  if (event.candidate) {
    signalingServer.send(JSON.stringify({ type: 'candidate', candidate: event.candidate }));
  }
};

signalingServer.onmessage = (message) => {
  const data = JSON.parse(message);

  if (data.type === 'candidate') {
    peerConnection.addIceCandidate(new RTCIceCandidate(data.candidate));
  }
};
```

---

## **6. Handle Media Streams**
- Once the connection is established, handle the incoming media stream.

### Example:
```javascript
peerConnection.ontrack = (event) => {
  const remoteVideo = document.querySelector('#remoteVideo');
  remoteVideo.srcObject = event.streams[0];
};
```

---

## **7. Handle Disconnections (Optional)**
Implement error handling and reconnection logic for network interruptions.

### Example:
```javascript
peerConnection.onconnectionstatechange = () => {
  if (peerConnection.connectionState === 'disconnected') {
    console.log('Peer disconnected.');
  }
};
```

---

## **8. Deploy a TURN Server (Optional)**
If both peers are behind restrictive NATs, a TURN server relays the media.

- Use **coturn** or a hosted TURN service.
- Example configuration (`/etc/turnserver.conf`):
  ```
  listening-port=3478
  fingerprint
  use-auth-secret
  static-auth-secret=your-secret
  realm=example.com
  ```
---

## **9. Final Checklist**
- **Signaling Server:** Facilitates the exchange of SDP and ICE candidates.
- **STUN/TURN Servers:** Handle NAT traversal and media relay.
- **SDP Exchange:** Share session descriptions to establish a connection.
- **ICE Candidates:** Find the best P2P route.
- **Media Handling:** Capture and display local and remote streams.

---

By following this flow, you can establish a WebRTC connection for real-time communication. If you encounter specific issues, share more details, and I’ll help troubleshoot!

