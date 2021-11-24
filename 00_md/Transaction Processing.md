# DB transaction and SAP transaction

データベースのトランザクションがどのように実行されているかを説明します。<br>
DBのトランザクションが実行されている間はDBにアクセスすることができません。<br>
また、特定のトランザクションが途中で終了した場合は、ロールバックされます。<br>
SAPは自動的に特定のレコードの前のデータにロールバックします。<br>
![01_SAP_transaction](https://github.com/koishi755/SAP/blob/main/transactional_processing/01_SAP_transaction.png)

## ワークプロセスの確認

SAPにログインし、どのようなワークプロセスが動いているか確認します。<br>
トランザクションコードSM50を実行します<br>
SM50はインスタンスに構成されているすべてのワークプロセスの概要を提供します。<br>
下図では、ダイアログワークプロセスはタイプ DIA として表示されており、３つあります。<br>
![02_sm50](https://github.com/koishi755/SAP/blob/main/transactional_processing/02_sm50.png)

<br>

### PID
PIDはワークプロセス自身のプロセスIDです。PIDはOSレベルでも参照されています。<br>
PIDを使用してワークプロセスを識別します。またシステム上で実行されているプログラムを特定するために使用します。

### Status
次にステータスを確認します。一つのワークプロセスを除いて全て「Waiting」となっています。<br>
「Waiting」にはRFCを待っていたり、ロックエントリーから解放されるのを待っていたりと様々な理由が存在します。<br>
「Waiting」の理由は「Reason」に記載されます。<br>
「Reason」がブランクになっている場合は新しいリクエストを受け入れる状態を意味します。

### Restart
「Restart」がYesとなっている場合、プロセスの再起動が可能なことを意味します。

### Error/Dumps
以下のようにDumpがそれぞれひとつ存在しています。<br>
特定のワークプロセスが原因でエラーが発生した場合、それをここで見ることができるということです。
![03_sm50_error](https://github.com/koishi755/SAP/blob/main/transactional_processing/03_sm50_error.png)

### Report
以下の場合、SAP*ユーザーがクライアント100にログインしていることを意味しています。<br>
![04_sm50_report](https://github.com/koishi755/SAP/blob/main/transactional_processing/04_sm50_report.png)

<br>

## データベースのロック

トランザクションコードSU3を実行し、ユーザーの編集画面に遷移します。
![05_su3](https://github.com/koishi755/SAP/blob/main/transactional_processing/05_su3.png)

<br>

新しい画面を開いてSU01を実行します。<br>
UserにSAP*と入力し鉛筆をクリックします。<br>
すると画面下部に「Maintenance of user SAP* by user SAP*」と表示されました。<br>
誰かがSAP*レコードを編集中だということが分かります。<br>
![06_su01](https://github.com/koishi755/SAP/blob/main/transactional_processing/06_su01.png)

<br>

次にトランザクションコードSM12を実行し、ロックエントリーを確認します。<br>
Select Lock Entries画面には以下の項目があります。

| 項目| 意味 |
-|-
|Table name|特定のテーブルから検索したい場合に使用します|
|Lock argument|どのフィールドがロックされているのか検索する場合に使用します|
|Client|クライアントを指定して検索します|
|User name|ユーザーを指定して検索します|

<br>

今回はUser nameに「*」を入れて検索します。
![07_sm12](https://github.com/koishi755/SAP/blob/main/transactional_processing/07_sm12.png)

<br>

ロックエントリーが表示されます。<br>
クライアント001のSAP*ユーザーがロックしていることが分かります。<br>
![08_sm12](https://github.com/koishi755/SAP/blob/main/transactional_processing/08_sm12.png)

<br>

主要な項目は以下の通りです。
| 項目| 意味 |
-|-
|Client|ロックしているユーザーが所属するクライアント|
|User name|ロックしているユーザー|
|Date/Time|ロック開始時刻|
|Lock Mode|ロックモード|
|Table|ロックされたテーブル|
|Argument|ロックされたフィールド|

ロックモードのEはexclusive Lock(排他的)を意味しています。<br>
ロックモードを選択し、「F1」を押すとヘルプを表示することができます。<br>
![09_sm12_select_Lock_Mode](https://github.com/koishi755/SAP/blob/main/transactional_processing/09_sm12_select_Lock_Mode.png)

![10_sm12_help](https://github.com/koishi755/SAP/blob/main/transactional_processing/10_sm12_help.png)

<br>

## Update work process

SAPシステムでどのように更新が行われるのか、アップデートワークプロセスについて見てみます。<br>

1. SAPGUI からの更新のリクエストはABAPディスパッチャーに送られます。
1. ディスパッチャーは更新リクエストをダイアログワークプロセスに送信します。
1. 更新する前に、編集モードに移行する必要があるので、特定のレコードをロックするため、エンキューを要求します。
1. メッセージサーバーにエンキューリクエストを送信
1. メッセージサーバーからABAPディスパッチャーを経由してエンキューワークプロセスへリクエストが送られる。
1. エンキューワークプロセスがロックテーブルにロックエントリを配置します。
1. ロックエントリが完了すると、エンキューワークプロセスはフィードバックを行い、フィールドの更新を要求します。（V1アップデート）
1. 再びディスパッチに行き、特定の更新を行うためにアップデートワークプロセスを起動します。


![11_update](https://github.com/koishi755/SAP/blob/main/transactional_processing/11_update.png)

<br>

コミット後の変更は、必ずしも同期的な変更ではないので注意してください。<br>
SAPは多くのユーザーが利用するので、非同期的に変更されます。<br>
またアップデートワークプロセスには2種類あります。<br>
V1アップデートは非常に重要なアップデートです。V2アップデートはそれほど重要でないアップデートです。<br>
<br>


### モニタリングアップデートワークプロセス

SAPにログインし、アップデートの様子を確認してみます。<br>
まず、SM50を実行し、アップデートワークプロセスが存在することを確認します。<br>
UPDがアップデートワークプロセスです。（バージョンによってはUP1、UP2と表示されます。）
V1 と V2 で、それぞれ異なるワークプロセスです。<br>
![12_sm50_check_up-p](https://github.com/koishi755/SAP/blob/main/transactional_processing/12_sm50_check_up-p.png)

<br>

次にアップデートワークプロセスをモニターします。<br>
[Tools]-[Administration]-[Monitor]-[Update]をクリックするか、SM13を実行します。<br>
![13_sm13](https://github.com/koishi755/SAP/blob/main/transactional_processing/13_sm13.png)

<br>

一般的には、キャンセルされた更新を監視します。<br>
なぜなら、途中でキャンセルされた更新はチェックしてクローズする必要があります。<br>
これが適切に完了していない場合、業務に影響が出る可能性があります。<br>
今回はAllを選択し、画面左上のExecuteをクリックします。<br>
![14_sm13_all](https://github.com/koishi755/SAP/blob/main/transactional_processing/14_sm13_all.png)

<br>

現在更新中ものやキャンセルされたリクエストはありませんでした。<br>
レコードを削除することもできますが、一般的には行いません。<br>
![15_sm13_uodate_record](https://github.com/koishi755/SAP/blob/main/transactional_processing/15_sm13_uodate_record.png)
