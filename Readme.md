# Chat 
TypeScript 言語でチャット開発
開発環境、フロントエンド開発環境は、TypeScript+Next.js
バックエンド開発には、TypeScript+Express+websocket.io
DBは使用せず簡単なチャット動作を学習目的になります。

## プロジェクトディレクトリを作成
```bash
mkdir chat_typescript && cd chat_typescript
```
## Next.js (client)ディレクトリ　Express+Socket.io (Server)ディレクトリを作成
```bash
mkdir client server
```
## Next.js プロジェクトを作成
```bash
cd client
npx create-next-app@latest ./ --ts
cd ../
```
## Express+Socket.io プロジェクトを作成
```bash
cd server
npm init -y
npm install -D typescript @types/node ts-node
npx tsc --init
npm install express
npm install -D @types/express
```
## http サーバーを起動する
```bash
vi index.ts
```
```typescript
import Express from "express"
import http from 'http'

const app = Express()
const httpServer = http.createServer(app)
const PORT = process.env.PORT || 8080

httpServer.listen(PORT, () => {
	console.log(`Server listening on port ${PORT}`)
})
```
```bash
npx ts-node index.ts
```
無事に起動すると以下の様にコンソールに表示されます
```bash
Server listening on port 8080
```
## socket 通信をする
本学習の目的である、チャットアプリを作成していきます。
```bash
npm install socket.io
```
```bash
vi index.ts
```
```typescript
import Express from "express"
import http from 'http'
import { Server } from "socket.io"


const app = Express()
const httpServer = http.createServer(app)
const io = new Server(httpServer, {
	cors: {
		origin: ['http://localhost:3000']
	}
})

// client と通信
io.on('connection', (socket) => {
	console.log('client と接続しました')

	//client から受信
	socket.on('send_message', (data) => {
		console.log(data)
		//client へ送信
		io.emit('received_message', data)
	})

	socket.on('disconnect', () => {
		console.log('client と接続が切れました')
	})
})

const PORT = process.env.PORT || 8080

httpServer.listen(PORT, () => {
	console.log(`Server listening on port ${PORT}`)
})
```
バックエンドサーバーとして起動させるので、以下のコマンドを実行します。
```bash
npx ts-node index.ts
```
次にフロントエンド開発用に、ターミナルウィンドウ別で開き
フロントエンド開発ディレクトリに移動します。
```bash
cd ../client
```
```bash
npm install socket.io-client
```
```bash
vi pages/index.tsx
```
```typescript
import type { NextPage } from 'next'
import Head from 'next/head'
import io from 'socket.io-client'
import { useEffect, useState } from 'react'

type TMessage = {
  user: string
  message: string
}
const socket = io('http://localhost:8080')

const Home: NextPage = () => {
  const [message, setMessage] = useState<string>("")
  const [list, setList] = useState<TMessage[]>([])
  const [user, setUser] = useState<string>("")
  const getUserName = (): string => {
    return Math.floor(1000 * Math.random()).toString(16)
  }

  const headleSendMessage = () => {
    // server へ送信
    socket.emit('send_message', { user: user, message: message })
    setMessage('')
  }
  // server から受信
  socket.on('received_message', (data: TMessage) => {
    console.log(data)
    setList([data, ...list])
  })
  // Page init getUserName
  useEffect(() => {
    setUser(getUserName())
  }, [])
  return (
    <div className='container'>
      <Head>
        <title>Chat Next.js Typescript</title>
        <meta name="description" content="Chat Next.js Typescript" />
        <meta name="viewport" content="width=device-width, initial-scale=1" />
        <link rel="icon" href="/favicon.ico" />
        <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-EVSTQN3/azprG1Anm3QDgpJLIm9Nao0Yz1ztcQTwFspd3yD65VohhpuuCOmLASjC" crossOrigin="anonymous" />
      </Head>
      <div className='container'>
        <h1 className='p-4 text-center fw-bold'>チャットアプリ</h1>
        <div className='input-group'>
          <input className='form-control form-control-lg' onChange={(e) => setMessage(e.target.value)} value={message} type="text" placeholder='テキスト入力してみて' />
          <button className='btn btn-primary' onClick={headleSendMessage}>チャット送信</button>
        </div>
        {list.map((chat, key) => {
          if (chat.user == user) {
            return (
              <div className='row justify-content-start' key={key}>
                <div className='col-8 shadow-lg p-2 m-2 mt-4 rounded bg-primary bg-gradient text-white'>
                  <p className='lh-base'>{chat.user}<br />
                    {chat.message}</p>
                </div>
              </div>
            )
          }
          return (
            <div className='row justify-content-end' key={key}>
              <div className='col-8 shadow-lg p-2 m-2 mt-4 rounded border border-primary text-primary'>
                <p className='lh-base'>{chat.user}<br />
                  {chat.message}</p>
              </div>
            </div>
          )
        })}
      </div>
    </div>
  )
}

export default Home
```
フロントエンドサーバーとして起動させるので、以下のコマンドを実行します。
```bash
npm run dev
```
以上で、チャットアプリのプログラムが完成しましたので、ブラウザーで **http://localhost:3000/** にアクセスしてチャットが出来るか確認してみてください。