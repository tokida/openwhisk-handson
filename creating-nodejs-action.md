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


5. すぐにアクションの結果を必要としない場合は、ノンブロッキング呼び出しを行うために`--blocking`
   フラグを省略することができます。アクティベーションIDを使用して後で結果を得ることができます。
   次の例を参照してください

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

6. もしアクティベーションIDがを忘れてしまった場合、新しいものから順に順序付けられた
   アクティベーションIDのリストを取得することができます。あなたのアクティベーションのリストを
   取得するには、次のコマンドを実行します。

  ```
  $ wsk activation list
  ```
  ```
  activations
  44794bd6aab74415b4e42a308d880e5b         hello
  6bf1f670ee614a7eb5af3c9fde813043         hello
  ```

### 非同期アクションの作成

非同期に実行するJavaScript関数は、main関数が戻った後に動いた結果を返す必要があります。あなた
のアクションでプロミス(Promise)を返すことによって、これを達成することができます。

1. `asyncAction.js`として次の内容を保管します。

  ```
  function main(args) {
    return new Promise(function(resolve, reject) {
        setTimeout(function() {
            resolve({ done: true });
            }, 2000);
        })
  }
```

主な機能は活性化がまだ完了していないことを示していますが、将来的にすることが期待され、プロミスを
返すことに注意してください。

Notice that the `main` function returns a Promise, which indicates that the
activation hasn't completed yet, but is expected to in the future.

この場合のsetTimeout（）JavaScript関数は、コールバック関数を呼び出す前に20秒間待ちます。
これは、非同期コードを表し、プロミスのコールバック関数内部に入ります。

The `setTimeout()` JavaScript function in this case waits for twenty seconds
before calling the callback function.  This represents the asynchronous code and
goes inside the Promise's callback function.

プロミスのコールバックは、両方のfunctionであるresolveとreject、2つの引数をとります。
`resolve()`の呼び出しはプロミスを実現し、アクティベーションが正常に完了したことを示します。

The Promise's callback takes two arguments, resolve and reject, which are both
functions.  The call to `resolve()` fulfills the Promise and indicates that the
activation has completed normally.

`reject()`の呼び出しはプロミスを拒否し、アクティベーションが異常終了したことを通知するため
に使用することができます。

A call to `reject()` can be used to reject the Promise and signal that the
activation has completed abnormally.

1. Actioを作成し実行するコマンドは以下のとおりです。

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

  非同期アクションのブロッキング呼び出しを行っていることに注意して下さい。

3. アクティベーションが完了するまでにかかった時間を確認するためにアクティベーションログを取得します。

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

  起動レコードで`start`と`end`のタイムスタンプを比較すると、あなたは、このアクティベーションが
  完了するまでに2秒以上かかったことがわかります。

### 外部APIの呼び出しをアクションから使う

例としては、これまでの自己完結型のJavaScript関数となっています。また、外部APIを呼び出すアク
ションを作成することができます。

この例では、特定の場所に現在の状態を取得するにはYahoo天気サービスを起動します。


1. `weather.js`として次の内容を保管します。

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



この例では、アクションは JavaScript request ライブラリーを使用して Yahoo Weather API への HTTP 要求を行い、JSON 結果からフィールドを抽出することに注意してください。リファレンスに、アクションで使用できる Node.js パッケージについての詳しい説明が記載されています。

この例は、非同期アクションの必要性も示しています。アクションは Promise を戻して、関数が戻ったときにこのアクションの結果はまだ使用可能になっていないことを示します。代わりに、結果は、HTTP 呼び出しが完了した後に request コールバックで使用可能になり、引数として resolve() 関数に渡されます。


2. 以下のコマンドを実行して、アクションを作成して起動します。

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
