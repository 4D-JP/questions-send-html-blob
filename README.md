コンテキストモードの代わりに自動セッション管理を使用する例題です。

__はじめに__

__自動セッション管理__を使用すれば，従来のコンテキストモードと同じように，Webアプリケーションの__コンテキスト__と4Dの__プロセス__』を関連付けることができます。ただし，プログラミングのスタイルや，使用するコマンドなどは違います。コンテキストモードから自動セッション管理に移行するためには，自動セッション管理に関する基本的な理解をしっかりと習得することが必要です。

__概要__

自動セッション管理は，__v13以降__，使用することができます。以前のバージョンでも，__HTTPクッキー__を自分で管理し，Webコンテキストを再現するために必要な__プロセス変数__や__セレクション__をレコードなどに保存すれば，Webセッションを管理することができました。自動セッション管理は，その名称が示すように，__HTTPクッキーの送受信__・Webコンテキストと4Dプロセスの関連付けなどを4Dが自動的に管理するというものです。さらに，パフォーマンスを向上させるために待機させておいた__プロセスを再利用__したり，そのような__プロセスが消滅する前__にコンテキストをデータベースに保存するイベントなども提供されています。自動セッション管理を活用すれば，とても簡単にWebセッションを管理することができます。

__セットアップ__

自動セッション管理は[データベース設定](http://doc.4d.com/4Dv13/4D/13.5/Web-Sessions-Management.300-1457382.ja.html)または[WEB SET OPTION](http://doc.4d.com/4Dv13/4D/13.5/WEB-SET-OPTION.301-1457388.ja.html)で有効にすることができます。

```
WEB SET OPTION(Web keep session;1)
```

__スコープ__

デフォルトの設定では，ランゲージで処理されるURLはすべてWebセッション管理が有効にされています。ランゲージで処理されるURLとは，つまりスタティックWebサーバー（Webフォルダー内に存在するファイルを自動的に配信すること）以外のURLであり，具体的には，[On Web Connection](http://doc.4d.com/4Dv13/4D/13.5/On-Web-Connection-Database-Method.300-1457408.ja.html)イベントが発生するようなURLのことです。

__注記__：2003以前は，そのようなURLには``4DCGI``という文字列を付けることが要求されていました。2004以降，``4DCGI``は省略できるようになりました。Webフォルダー内にファイルが存在しなければ，必然的に``On Web Connection``イベントが発生します。なお，ドキュメントでは，便宜上，Webフォルダー内に対応するファイルが存在しないことを『無効なリクエスト』と呼んでいます。

__セッションID__

自動セッション管理では，ブラウザから送信されるHTTPクッキーとIPアドレスの組み合わせでセッションを識別します。クッキーの名称はデフォルトで``4DSID``ですが，[WEB SET OPTION](http://doc.4d.com/4Dv13/4D/13.5/WEB-SET-OPTION.301-1457388.ja.html)で変更することもできます。すでに開かれたセッションであるとWebサーバーが判断すれば，停止中のプロセスが再開され，前回の処置で作成したセレクションやプロセス変数をそのまま使い続けることができます。新しいセッションであるとWebサーバーが判断すれば，新規プロセスが作成されます。

デベロッパーは，``On Web Connection``，あるいは``4DACTION``メソッドコールなど，``On Web Connection``を実行しないようなアクセスであれば，そのコールされたメソッドの冒頭で，[WEB Get Current Session ID](http://doc.4d.com/4Dv13/4D/13.5/WEB-Get-Current-Session-ID.301-1457386.ja.html)をコールし，__セッションIDをプロセス変数に代入__します。以降，メソッド開始時にプロセス変数を参照し，すでに値が代入されていれば，それは継続中のコンテキストであると判断することができます。

```
If (Length(Web_currentSessionId)=0)
  Web_currentSessionId:=WEB Get Current Session ID
End if 
```

__Web変数__

v12以前には，プロセス変数を``Compiler_Web``でプロセス変数を宣言しておけば，[HTTP POSTまたはGETで送信されたWebフォーム変数を4Dのプロセス変数に自動代入する](http://doc.4d.com/4Dv13/4D/13.5/Binding-4D-objects-with-HTML-objects.300-1457417.ja.html)，というメカニズムが存在しました。しかし，Webフォーム変数は[WEB GET VARIABLES](http://doc.4d.com/4Dv13/4D/13.5/WEB-GET-VARIABLES.301-1457406.ja.html)，マルチパートでポストされたファイルデータなどは[WEB GET BODY PART](http://doc.4d.com/4Dv13/4D/13.5/WEB-GET-BODY-PART.301-1457383.ja.html)でより効率的に受け取ることができ，そうすれば，Web変数名と同名のプロセス変数を用意する必要もないので，[『Webから送信された変数をプロセス配列に自動代入する』は，v13で廃止されました](http://doc.4d.com/4Dv13/4D/13.4/Compatibility-page.300-1226529.ja.html)。

``WEB GET VARIABLES``は，Webフォーム変数を名前と値の配列ペアで返すコマンドです。目的の変数は，``Find in array``でチェックした後，任意のローカル変数に代入することができます。Web変数と同名のプロセス変数である必要はありません。v14であれば，オブジェクト変数にまとめて管理することもできます。

```
C_OBJECT($variables)
ARRAY TEXT($names;0)
ARRAY TEXT($values;0)
WEB GET VARIABLES($names;$values)
For ($i;1;Size of array($names))
  OB SET($variables;$names{$i};$values{$i})
End for 
```

自動セッション管理が有効にされていれば，Webプロセスのコンテキストにおけるプロセス変数の値は保持されています。ただし，そのような変数は，``Compiler_WEB``で宣言されていなければなりません。たとえば，HTMLに下記のようなコードが記述されているとします。


```html
<form method="post" action="/4daction/Web_得意先履歴一覧/">
<input type="text" name="v得意先名検索" value="<!--#4dtext v得意先名検索-->" />
<input type="submit" value="検索" />
</form>
```

はじめてこのページからフォームを送信したとき，``Compiler_WEB``が実行され，``v得意先名検索``のようなプロセス変数を4D側で宣言する機会が与えられます。前述したように，Web変数の自動代入はもう実行されないので，``4DACTION``でコールされたメソッドの中で``WEB GET VARIABLES``を使用し，受け取った値を明示的に代入しなければなりません。しかし，そうすれば，次回のアクセスからは，[Compiler_WEBが実行されません](http://doc.4d.com/4Dv13/4D/13.5/Web-Sessions-Management.300-1457382.ja.html)し，変数には前回代入した値が残っています。


