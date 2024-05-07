# nest-chat-backend

This repository is use nestjs with socket.io as chat service backend


## pre-install

```shell
pnpm i -S @nestjs/websockets @nestjs/platform-socket.io 
```

## create chat module

```shell
nest g module chat
```

## setup logic for listen onEvent from client

```typescript
import {
  MessageBody,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';
@WebSocketGateway(3002, {
  cors: {
    origin: '*',
  },
})
export class ChatGateway {
  @SubscribeMessage('newMessage')
  handleNewMessage(@MessageBody() message: string) {
    // read message from client
    console.log(message);
  }
}
```

## setup logic for handle socket connection status

```typescript
import {
  MessageBody,
  OnGatewayConnection,
  OnGatewayDisconnect,
  SubscribeMessage,
  WebSocketGateway,
  WebSocketServer,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';
@WebSocketGateway(3002, {
  cors: {
    origin: '*',
  },
})
export class ChatGateway implements OnGatewayConnection, OnGatewayDisconnect {
  handleDisconnect(client: Socket) {
    console.log('user disconnected', client.id);
    this.server.emit('user-left', {
      message: `User Left the chat: ${client.id}`,
    });
  }
  handleConnection(client: Socket) {
    console.log('New user connected..', client.id);
    // broadcast other client socket
    client.broadcast.emit('user-joined', {
      message: `New User Joined the chat: ${client.id}`,
    });
  }
  // setup server socket
  @WebSocketServer() server: Server;
  @SubscribeMessage('newMessage')
  handleNewMessage(@MessageBody() message: string) {
    // broadcast from server to client socket
    this.server.emit('message', message);
  }
}
````