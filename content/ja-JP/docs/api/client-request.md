## クラス: ClientRequest

> HTTP/HTTPリクエストを行います。

プロセス: [Main](../glossary.md#main-process)

`ClientRequest` は [Writable Stream](https://nodejs.org/api/stream.html#stream_writable_streams) インターフェースを実装しているため、 [EventEmitter](https://nodejs.org/api/events.html#events_class_eventemitter) です。

### `new ClientRequest(options)`

* `options` (Object | String) - もし `options` が String の場合、リクエストURLとして解釈されます。もし Object の場合、以下のプロパティによるHTTPリクエストとして完全に指定されていることが期待されます。 
  * `method` String (任意) - HTTPリクエストメソッド。省略値は、GETメソッドです。
  * `url` String (任意) - リクエストURL。httpまたはhttpsとして指定されているプロトコルスキームを伴う完全な形式で指定しなければなりません。
  * `session` Object (任意) - リクエストが関連付けられている [`Session`](session.md) のインスタンス。
  * `partition` String (任意) - リクエストが関連付けられている [`partition`](session.md) の名前。 省略値は、空の文字列です。 `session` オプションは、`partition` よりも優先されます。 そのため、`session` が明示的に指定されている場合、`partition` は無視されます。
  * `protocol` String (任意) - 'scheme:' という形式のプロトコルスキーム。 現在サポートされている値は、'http:' または 'https:' です。省略値は、'http:' です。
  * `host` String (任意) - ホスト名とポート番号を連結した 'hostname:port' として指定されたサーバーホスト。
  * `hostname` String (任意) - サーバーホスト名。
  * `port` Integer (任意) - サーバーのリスニングポート番号。
  * `path` String (任意) - リクエストURLのパスの部分。
  * `redirect` String (任意) - このリクエストのリダイレクトモード。 `follow`、`error` または `manual` のいずれかにする必要があります。 省略値は、`follow` です。 モードが `error` のとき、リダイレクトは中止されます。 モードが `manual` のとき、[`request.followRedirect`](#requestfollowRedirect) が呼び出されるまで、リダイレクトは遅延されます。 リダイレクトリクエストの詳細を得るため、このモードでは、[`redirect`](#event-redirect) イベントを待ち受けるようにしてください。

`protocol`、`host`、`hostname`、`port` や `path` といった `options` プロパティは、[URL](https://nodejs.org/api/url.html) モジュールで説明されている Node.js モデルに厳密に従うようにしてください。

例えば、以下のようにすると、'github.com' に対してリクエストをしているのと同じです。

```JavaScript
const request = net.request({
  method: 'GET',
  protocol: 'https:',
  hostname: 'github.com',
  port: 443,
  path: '/'
})
```

### インスタンスイベント

#### イベント: 'response'

戻り値:

* `response` IncomingMessage - HTTPレスポンスメッセージを表すオブジェクト。

#### イベント: 'login'

戻り値:

* `authInfo` Object 
  * `isProxy` Boolean
  * `scheme` String
  * `host` String
  * `port` Integer
  * `realm` String
* `callback` Function 
  * `username` String
  * `password` String

認証プロキシがユーザの資格情報を要求しているときに発生します。

`callback` ファンクションは、ユーザの資格情報と共にコールバックされます。

* `username` String
* `password` String

```JavaScript
request.on('login', (authInfo, callback) => {
  callback('username', 'password')
})
```

空の資格情報を指定すると、リクエストがキャンセルされ、レスポンスオブジェクトで認証エラーが返ります。

```JavaScript
request.on('response', (response) => {
  console.log(`STATUS: ${response.statusCode}`);
  response.on('error', (error) => {
    console.log(`ERROR: ${JSON.stringify(error)}`)
  })
})
request.on('login', (authInfo, callback) => {
  callback()
})
```

#### イベント: 'finish'

`request` のデータの最後のチャンクが `request` オブジェクトに書き込まれた直後に発生します。

#### イベント: 'abort'

`request` が中止されたときに発生します。`request` が既に終了している場合、`abort` イベントは発生しません。

#### イベント: 'error'

戻り値:

* `error` Error - 失敗に関するいくつかの情報を提供するエラーオブジェクト。

`net` モジュールがネットワークリクエストを行うのに失敗するときに発生します。 通常、`request` オブジェクトが `error` イベントを発生させるとき、続いて `close` イベントが発生し、レスポンスオブジェクトが返ることはありません。

#### イベント: 'close'

HTTPのリクエストからレスポンスまでのやり取りの最後のイベントして発生します。 `close` イベントは、`request` または `response` オブジェクトのいずれでもこれ以上のイベントが発生しないことを示します。

#### イベント: 'redirect'

戻り値:

* `statusCode` Integer
* `method` String
* `redirectUrl` String
* `responseHeaders` Object

リダイレクトがあり、モードが `manual` のときに発生します。[`request.followRedirect`](#requestfollowRedirect) を呼び出すことでリダイレクトが続行されます。

### インスタンスプロパティ

#### `request.chunkedEncoding`

A `Boolean` specifying whether the request will use HTTP chunked transfer encoding or not. Defaults to false. The property is readable and writable, however it can be set only before the first write operation as the HTTP headers are not yet put on the wire. Trying to set the `chunkedEncoding` property after the first write will throw an error.

Using chunked encoding is strongly recommended if you need to send a large request body as data will be streamed in small chunks instead of being internally buffered inside Electron process memory.

### インスタンスメソッド

#### `request.setHeader(name, value)`

* `name` String - An extra HTTP header name.
* `value` Object - An extra HTTP header value.

Adds an extra HTTP header. The header name will issued as it is without lowercasing. It can be called only before first write. Calling this method after the first write will throw an error. If the passed value is not a `String`, its `toString()` method will be called to obtain the final value.

#### `request.getHeader(name)`

* `name` String - Specify an extra header name.

Returns `Object` - The value of a previously set extra header name.

#### `request.removeHeader(name)`

* `name` String - Specify an extra header name.

Removes a previously set extra header name. This method can be called only before first write. Trying to call it after the first write will throw an error.

#### `request.write(chunk[, encoding][, callback])`

* `chunk` (String | Buffer) - A chunk of the request body's data. If it is a string, it is converted into a Buffer using the specified encoding.
* `encoding` String (optional) - Used to convert string chunks into Buffer objects. Defaults to 'utf-8'.
* `callback` Function (optional) - Called after the write operation ends.

`callback` is essentially a dummy function introduced in the purpose of keeping similarity with the Node.js API. It is called asynchronously in the next tick after `chunk` content have been delivered to the Chromium networking layer. Contrary to the Node.js implementation, it is not guaranteed that `chunk` content have been flushed on the wire before `callback` is called.

Adds a chunk of data to the request body. The first write operation may cause the request headers to be issued on the wire. After the first write operation, it is not allowed to add or remove a custom header.

#### `request.end([chunk][, encoding][, callback])`

* `chunk` (String | Buffer) (任意)
* `encoding` String (任意)
* `callback` Function (任意)

Sends the last chunk of the request data. Subsequent write or end operations will not be allowed. The `finish` event is emitted just after the end operation.

#### `request.abort()`

Cancels an ongoing HTTP transaction. If the request has already emitted the `close` event, the abort operation will have no effect. Otherwise an ongoing event will emit `abort` and `close` events. Additionally, if there is an ongoing response object,it will emit the `aborted` event.

#### `request.followRedirect()`

Continues any deferred redirection request when the redirection mode is `manual`.