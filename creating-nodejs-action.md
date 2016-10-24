# NodejsのActionの作成と実行


ActionはOpenWhisk上で動くステートレスなコードスニペットです。
アクションはJavaScript関数、Swift関数、またはDockerの形式で提供されるカスタム実行可能プログラムを動かす事が出来ます。たとえば、Actionは、画像中の顔を検出API呼び出しを実行したりツイートを投稿するために使用することができます。

Actionは明示的に起動(invoked)、またはイベントに応答して実行(Run)することができます。いずれの場合も、Actionの実行は、固有のActivation IDによってかんりされます。Actionへの入力(Input)とアクションの結果(Result)は、キーが文字列と値のJSON型であるキーと値のペアの辞書で行われます。

Actionは、他のActionまたはActionの定義された配列(sequence)からの呼び出しで構成することができます。


## JavaScript Actionの作成と実行

このセクションでは、JavaScriptでの操作で作業を紹介します。
単純なアクションの作成と呼び出しで始まります。そして、アクションにパラメータを追加し、パラメータを指定して、そのアクションを呼び出す事をします。
次に、デフォルトパラメータを設定し、アクションを起動します。非同期アクションを作成して、最終的には、アクションのシーケンスを扱います。


## 単純なJavaScript actionの作成と実行

最初のJavaScriptのActionの作成を例に習ってみていきます。

1. 'hello.js'という名前で以下の内容のファイルを作成します。

  ```
  function main() {
      return {payload: 'Hello world'};
  }
  ```

  このJavaScriptは、追加の"functions"があるかもしれません、しかし慣例として main()関数はActionのエントリポイントを提供するために存在している必要があります。


2. 次のJavaScript関数からActionを作成します。この例ではActionは"hello"とします。

  ```
  $ wsk action create hello hello.js
  ```
  ```
  ok: created action hello
  ```

3. あなたの作成したActionをリストします:

  ```
  $ wsk action list
  ```
  ```
  actions
  hello       private
  ```

  あなた見ている"hello"が作成したものです。



4. アクションを作成したら、「起動(Invoke)」コマンドでOpenWhiskにおけるクラウドでそれを呼び
   出すことができます。コマンドにフラグを指定することにより、*ブロッキング呼び出し*（すなわち、
   要求/応答スタイル）または *ノンブロッキング呼び出し* とアクションを実行することができます。
   ブロッキング呼び出し要求が利用できるように活性化した結果を待ちます。待機時間は60秒または
   アクションの設定された制限時間の少ない方です。それは待機期間内に利用可能である場合に活性化の
   結果が返されます。そうでなければ、活性化は、システム内で処理を続行し、*ノンブロッキング要求*
   と同様に、後で結果を確認できるように、アクティベーションIDが返されます。

  この例は、ブロッキングの引数である `--blocking`を使用します。

  ```
  $ wsk action invoke --blocking hello
  ```
  ```
  ok: invoked hello with id 44794bd6aab74415b4e42a308d880e5b
  {
      "result": {
          "payload": "Hello world"
      },
      "status": "success",
      "success": true
  }
  ```d

  このコマンドの結果は、２つの重要な情報が含まれます。
  * activation ID は (`44794bd6aab74415b4e42a308d880e5b`)
  * 待機時間内に完了した場合にはその結果の内容

  この場合の結果は、JavaScript関数によって返される文字列 `Hello world`です。
  Activation IDは、将来の時点での呼び出しのログや結果を取得するために使用することができます。


5. If you don't need the action result right away, you can omit the `--blocking`
   flag to make a non-blocking invocation. You can get the result later by using
   the activation ID. See the following example:

  ```
  $ wsk action invoke hello
  ```
  ```
  ok: invoked hello with id 6bf1f670ee614a7eb5af3c9fde813043
  ```

  ```
  $ wsk activation result 6bf1f670ee614a7eb5af3c9fde813043
  ```
  ```
  {
      "payload": "Hello world"
  }
  ```

6. If you forget to record the activation ID, you can get a list of activations
   ordered from the most recent to the oldest. Run the following command to get
   a list of your activations:

  ```
  $ wsk activation list
  ```
  ```
  activations
  44794bd6aab74415b4e42a308d880e5b         hello
  6bf1f670ee614a7eb5af3c9fde813043         hello
  ```
### Creating asynchronous actions

JavaScript functions that run asynchronously may need to return the activation
result after the `main` function has returned. You can accomplish this by
returning a Promise in your action.

1. Save the following content in a file called `asyncAction.js`.

  ```
  function main(args) {
    return new Promise(function(resolve, reject) {
        setTimeout(function() {
            resolve({ done: true });
            }, 2000);
        })
  }
```

Notice that the `main` function returns a Promise, which indicates that the
activation hasn't completed yet, but is expected to in the future.

The `setTimeout()` JavaScript function in this case waits for twenty seconds
before calling the callback function.  This represents the asynchronous code and
goes inside the Promise's callback function.

The Promise's callback takes two arguments, resolve and reject, which are both
functions.  The call to `resolve()` fulfills the Promise and indicates that the
activation has completed normally.

A call to `reject()` can be used to reject the Promise and signal that the
activation has completed abnormally.

2. Run the following commands to create the action and invoke it:

  ```
  $ wsk action create asyncAction asyncAction.js
  ```
  ```
  $ wsk action invoke --blocking --result asyncAction
  ```
  ```
  {
      "done": true
  }
  ```

  Notice that you performed a blocking invocation of an asynchronous action.

3. Fetch the activation log to see how long the activation took to complete:

  ```
  $ wsk activation list --limit 1 asyncAction
  ```
  ```
  activations
  b066ca51e68c4d3382df2d8033265db0             asyncAction
  ```


  ```
  $ wsk activation get b066ca51e68c4d3382df2d8033265db0
  ```
 ```
  {
      "start": 1455881628103,
      "end":   1455881648126,
      ...
  }
  ```

  Comparing the `start` and `end` time stamps in the activation record, you can
  see that this activation took slightly over two seconds to complete.

### Using actions to call an external API

The examples so far have been self-contained JavaScript functions. You can also
create an action that calls an external API.

This example invokes a Yahoo Weather service to get the current conditions at a
specific location.

1. Save the following content in a file called `weather.js`.

  ```
  var request = require('request');

  function main(params) {
      var location = 'Vermont';
      var url = 'https://query.yahooapis.com/v1/public/yql?q=select item.condition from weather.forecast where woeid in (select woeid from geo.places(1) where text="' + location + '")&format=json';

      return new Promise(function(resolve, reject) {
          request.get(url, function(error, response, body) {
              if (error) {
                  reject(error);
              }
              else {
                  var condition = JSON.parse(body).query.results.channel.item.condition;
                  var text = condition.text;
                  var temperature = condition.temp;
                  var output = 'It is ' + temperature + ' degrees in ' + location + ' and ' + text;
                  resolve({msg: output});
              }
          });
      });
  }
  ```

Note that the action in the example uses the JavaScript `request` library to
make an HTTP request to the Yahoo Weather API, and extracts fields from the JSON
result. The [References](./reference.md#javascript-runtime-environments) detail
the Node.js packages that you can use in your actions.

This example also shows the need for asynchronous actions. The action returns a
Promise to indicate that the result of this action is not available yet when the
function returns. Instead, the result is available in the `request` callback
after the HTTP call completes, and is passed as an argument to the `resolve()`
function.

2. Run the following commands to create the action and invoke it:

  ```
  $ wsk action create weather weather.js
  ```
  ```
  $ wsk action invoke --blocking --result weather
  ```
  ```
  {
      "msg": "It is 28 degrees in Vermont and Cloudy"
  }
  ```
