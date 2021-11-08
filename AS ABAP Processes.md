# 1. 3層アーキテクチャ
プレゼンテーションレイヤー、アプリケーションレイヤー、データベースレイヤーがあります。<br>
3層アーキテクチャについては[こちら](https://github.com/koishi755/SAP/wiki/SAP%E3%81%AE%E3%82%A2%E3%83%BC%E3%82%AD%E3%83%86%E3%82%AF%E3%83%81%E3%83%A3#1-3%E5%B1%A4%E3%82%A2%E3%83%BC%E3%82%AD%E3%83%86%E3%82%AF%E3%83%81%E3%83%A3)を参照してください。

<br>

![](https://github.com/koishi755/SAP/blob/main/AS_ABAP_Processes/ABAP_three_Tier_Acrchitecture.png)

<br>

# 2. Work Process Architecture
ワークプロセスは、タスクハンドラ、ダイアログまたはdynproインタプリタ、ABAPプロセッサ、データベースインタフェースから構成される。

<br>

![ワークプロセス](https://github.com/koishi755/SAP/blob/main/AS_ABAP_Processes/work_processes.png)

<br>


## 2.1. Task handler
ワークプロセス内のタスクはタスクハンドラが調整します。<br>
タスクハンドラは、各ダイアログステップの開始時と終了時に、ユーザーセッションコンテキストのロードとアンロードを管理します。<br>
また、ディスパッチャと通信し、タスクを実行するために必要に応じてdynproインタプリタやABAPプロセッサを起動します。<br>


## 2.2. ABAPプロセッサ
アプリケーションプログラムの実際の制御ロジックはSAP 独自のプログラミング言語である ABAP で書かれます。ABAP プロセッサによりアプリケーションプログラムの制御ロジックが実行され、データベースインタフェースとの通信が行われます。

<br>


## 2.3. データベースインタフェース
データベースインタフェースは以下のサービスを提供します。

* ワークプロセスとデータベースの間の接続の確立と終了
* データベーステーブルへのアクセス
* リポジトリオブジェクト (ABAP プログラム、Dynpro など) へのアクセス
* カタログ情報へのアクセス (ABAP ディクショナリ)
* トランザクションの制御 (コミットとロールバックの処理)
* ABAP アプリケーションサーバ上でのテーブルバッファの管理

 
![Database Interface](https://github.com/koishi755/SAP/blob/main/AS_ABAP_Processes/Database%20Interface.png)

上図では、データベースへのアクセス方法に、Open SQL と Native SQL の 2 種類があることを示しています。<br><br>

### Open SQL
Open SQL はABAP と完全に統合された Standard SQL のサブセットです。<br>
Open SQL を使用すると、ユーザのシステムで使用しているデータベースシステムとは無関係にデータにアクセスすることができます。<br>
Open SQL は Standard SQL のデータ操作言語 (DML) 部分から構成されます。<br>
すなわち、これを使用してデータの読込 (SELECT) と変更 (INSERT、UPDATE、DELETE) を行うことができます。<br>
NetWeaver AS ABAP では、Standard SQL のデータ定義言語 (DDL) とデータ制御言語 (DCL) 部分の役割は、ABAP ディクショナリおよび権限システムによって実行されます。<br>
これらはデータベースとは無関係に、統一された一連の機能を提供します。<br>
各種のデータベースシステムから提供されるものを超えた機能も含まれます。<br><br>

### Native SQL

Native SQL は緩くABAP と統合されているのみなので、各データベースシステムのプログラミングインタフェースに含まれる機能のすべてに対してアクセスすることができます。<br>
Native SQL では、主としてデータベース固有の SQL 文を使用することができます。<br>
それらの SQL 文は Native SQL インタフェースから実行場所であるデータベースシステムへとそのまま送信されます。各データベースのすべての SQL 言語領域を使用することができます。<br>
したがって、Native SQL を使用するすべてのプログラムは、インストールされたデータベースシステムに固有のものとなります。<br><br>


# 3. ワークプロセスのタイプ
SAP システムを起動すると、いくつかの異なるカテゴリーのワークプロセスが開始されます。
* SAP dialog work processes
* SAP background (batch) work processes
* SAP update work processes
* SAP enqueue (lock) work processes
* SAP spool work processes



## 3.1. Dialog Work Processes

ダイアログワークプロセスは、システム内のすべての対話型処理に使用されます。<br>
ダイアログワークプロセスの数は、インスタンスプロファイルのrdisp / wp_no_dia プロファイルパラメータを変更することによって構成されます。 <br>
トランザクションコードSM51 は、SAP システムのすべてのアプリケーションサーバー（インスタンス）のリストを提供します。 <br>

<br>

![SM51](https://github.com/koishi755/SAP/blob/main/AS_ABAP_Processes/sm51.png)

<br>


アプリケーションサーバーをダブルクリックすると、特定のワークプロセスの概要を確認できます。<br>
特定のアプリケーションサーバーで実行されるトランザクションコード SM50 は、インスタンスに構成されているすべてのワークプロセスの概要を提供します。 下図では、ダイアログワークプロセスはタイプ DIA として表示されています。

<br>

![sm50](https://github.com/koishi755/SAP/blob/main/AS_ABAP_Processes/sm50.png)

<br>

複数のアプリケーションサーバーがある場合、トランザクションコード SM66 を使用して、グローバルワークプロセスの概要を取得できます。 <br>
このトランザクションコードを使用して、特定の時点でのSAP システムのさまざまなアプリケーションサーバーからのワークプロセスのステータスを確認できます。
このトランザクションは、オンライントランザクションまたはバックグラウンドジョブの実行をリアルタイムでチェックする場合に特に役立ちます。

<br>

![sm66](https://github.com/koishi755/SAP/blob/main/AS_ABAP_Processes/sm66.png)

<br>

要約すると、次の 3 つのトランザクションは、SAP ワークプロセスの監視と追跡に役立ちます。

| T-CODE | 意味 |
-|-
|sm50|個々のアプリケーションサーバーのワークプロセスを一覧表示します。|
|sm51|1つのSAPシステムに属するすべてのアプリケーションサーバーを一覧表示します。|
|sm66|1つのSAPシステムに属するすべてのアプリケーションサーバーのアクティブなワークプロセスを一覧表示します。|


### リクエストの流れ

SAPABAPリクエストのプロセスフローを見てみましょう。<br>
ここではユーザーがSAPGUIを使用していると仮定しましょう。<br>
ユーザーはSAPGUIを使用して、SAPシステムにログインし、情報を要求したり、タスクを実行したりします。<br>


1. リクエストはABAPディスパッチャーに送られ、ABAPディスパッチャキューにキューイングされます。
1. ディスパッチャはリクエストを処理するために利用可能なワークプロセスがあるかチェックします。<br>
もしなければリクエストはリクエストキューに入れられ、利用可能になるまで待ちます。
1. ダイアログワークプロセスが利用可能になると、ディスパッチャはデータをワークプロセスに送ります。
1. リクエストに必要な特定の情報があるかどうか、共有メモリをチェックします。
1. リクエストに必要な情報が共有メモリーにない場合は、データベースにアクセスします。<br>
データベースから情報を引き出し、メモリに取り込み、特定のデータを処理して、ユーザーに出力します
<br>

![dialog_steps](https://github.com/koishi755/SAP/blob/main/AS_ABAP_Processes/dialog_steps.png)

<br>

## 3.2. Enqueue Work Processes

エンキューワークプロセスは、データの一貫性を保つため、誰かがデータを使用または編集している時、他のユーザーがデータを使用・編集出来ないようにロックします。

![Enqueue](https://github.com/koishi755/SAP/blob/main/AS_ABAP_Processes/Enqueue_work_process.png)

### ロックの流れ（エンキューワークプロセスが内部にある場合）
1. ユーザがデータへの変更アクセスを希望する場合、実行中のダイアログワークプロセスがロックを要求します。
2. エンキューサーバでダイアログリクエストが処理されると、ダイアログワークプロセスはロックテーブルに直接アクセスできるようになります。
3. ここで、新しいロックが生成できるかどうか、つまり、すでに設定されているロックとの衝突がないかどうかをチェックする。
4. ロックが設定できる場合、ダイアログワークプロセスはそれを作成し、ユーザ(ロック・オーナー)にロックキーを与える。ロックキーは、共有メモリ上のユーザコンテキストに保持される。

<br>

### ロックの流れ（エンキューワークプロセスが内部にない場合）
1.  ユーザーリクエストを処理するダイアログワークプロセスとエンキューワークプロセスが同一インスタンス上で動作していない場合、これら2つのワークプロセスはメッセージサーバーを介して通信します。
2. ロックリクエストは、ダイアログワークプロセスからディスパッチャとメッセージ・サーバを経由して、エンキューワークプロセスに転送されます。
3. ここでエンキューワークプロセスは、ロックが設定できるかどうかをチェックする。
4. ロックが可能であれば、エンキューワークプロセスによってロックが設定され、ロックキーがディスパッチャとメッセージサーバを介して要求元のダイアログワークプロセスに転送される。
5. ロックがリクエストされると、システムはリクエストされたロックがロックテーブルの既存のエントリと競合するかどうかをチェックします。ロックテーブルに既に対応するエントリがある場合、ロックリクエストは拒否されます。アプリケーション・プログラムは、リクエストされた操作が現在実行できないことをユーザーに通知します。

<br>

### エンキューワークプロセスの確認

トランザクションコードSM12を実行します。<br>
または以下のように[SAP Menu]-[Tools]-[Administration]-[Monitor]-[Lock Entries]から確認できます。

![Lock_entry](https://github.com/koishi755/SAP/blob/main/AS_ABAP_Processes/Lock_entry.png)



ここではユーザー名などを指定してロックエントリーを検索できます。<br>
今回はユーザー名を「*」としてEnterをクリックしました。
![sm12](https://github.com/koishi755/SAP/blob/main/AS_ABAP_Processes/sm12.png)

<br>

今の状態ではロックエントリーは存在しないようです。
![Lock_entry_list](https://github.com/koishi755/SAP/blob/main/AS_ABAP_Processes/Lock_entry_list.png)

<br>

次にユーザーセッションの状態を確認します。<br>
トランザクションコードSM04を実行します。<br>
下図のようにシステム内のユーザーセッションが確認できます。

![sm04](https://github.com/koishi755/SAP/blob/main/AS_ABAP_Processes/sm04.png)

<br>

ユーザーをダブルクリックすると特定のユーザーのセッション情報を確認できます。<br>
またこの画面からセッションを削除することもできます。

![sm04_detail](https://github.com/koishi755/SAP/blob/main/AS_ABAP_Processes/sm04_detail.png)

<br>

次に実際にロックエントリーの動きを確認します。<br>
SU01を実行します。SU01ではユーザー情報を編集できます。<br>
UserにSAP*と入力し、左上の鉛筆ボタンをクリックします。

![su01](https://github.com/koishi755/SAP/blob/main/AS_ABAP_Processes/su01.png)

<br>

以下のようにSAP*ユーザーの編集モードに入りました。

![su01_edit](https://github.com/koishi755/SAP/blob/main/AS_ABAP_Processes/su01_edit.png)

次に新しいセッションを開き、SM12を実行します。<br>
ロックエントリーが追加されたことを確認できます。

![sm12_lock](https://github.com/koishi755/SAP/blob/main/AS_ABAP_Processes/sm12_lock.png)

<br>


ロックエントリーを削除する場合は、対象のエントリー行を選択して、Deleteボタンをクリックします。
![sm12_delete](https://github.com/koishi755/SAP/blob/main/AS_ABAP_Processes/sm12_delete.png)

<br>

## 3.3 Spool Work Process

スプールワークプロセスは順次データセットをプリンタまたはイメージストアへと渡します。<br>
ダイアログステップを使用して印刷用のデータを作成し、インターフェイスを通してプリンタにすることができます。<br>
このインターフェースは、プリンタ、ファックス、また電子メールなどがあります。<br>


### 印刷の流れ

1. 印刷したい画面がある場合は、ダイアログワークプロセスから印刷のリクエストを作成します。<br>
このリクエストには固有の番号と必要なデータがすべて含まれます。 <br>
2. スプールリクエストはTemSeに送られます。<br>
TemSeとは、一時的なデータが保存されるテーブルのことです。<br>
3. データを明示的に出力機器に送る場合は、スプールリクエストから出力要求を生成します。これはスプールワークプロセス（S-WP）に転送されます。
4. スプールワークプロセスは、出力要求データをフォーマットします。
これは、SAPシステムの内部データストリームを、出力デバイスが理解できるデータストリームに変換します。
5. フォーマット後、スプールワークプロセスはプリント要求をホストスプールシステム(OSのスプーラ)に転送します。


![spool](https://github.com/koishi755/SAP/blob/main/AS_ABAP_Processes/spool.png)

<br>

### スプールリクエストの確認

トランザクションコードSM50を実行します。（今回は例としてSM50を実行しています。）<br>
画面上部の印刷ボタンをクリックします。
![sm50_spool](https://github.com/koishi755/SAP/blob/main/AS_ABAP_Processes/sm50_spool.png)

<br>

Output Deviceを入力します。（今回は「LP01」としています。）<br>
画面下部のContinuボタンをクリックします。
![print_list](https://github.com/koishi755/SAP/blob/main/AS_ABAP_Processes/print_list.png)

<br>

Informationが表示されるのでContinuボタンをクリックします。
![spool_informatin](https://github.com/koishi755/SAP/blob/main/AS_ABAP_Processes/print2.png)

<br>

画面下部にSpool requestが表示されます。こちらのnumberは覚えてください。
![spool_request](https://github.com/koishi755/SAP/blob/main/AS_ABAP_Processes/spool_request.png)

<br>

[System]-[Own Spool Requests]をクリックします。
![goto_check_spool](https://github.com/koishi755/SAP/blob/main/AS_ABAP_Processes/check_spool_request.png)

以下の通り上記で確認したnumberのリクエストが作成されています。
![spool_request2](https://github.com/koishi755/SAP/blob/main/AS_ABAP_Processes/spool_request2.png)

<br>

## 3.4. Background Work Process

ここではバックグラウンドワークプロセスがどのように使用されているのかを見ていきます。<br>

### 何故バックグラウンドワークプロセスが必要か？

そもそも何故バックグラウンドワークプロセスが必要なのでしょうか。<br>
ダイアログワークプロセスは状況によって複数のユーザーを生み出す形になります。
また特定のリクエストに長い時間を要することがあります。
するとこのダイアログワークプロセスはある特定のユーザーに占有されてしまいます。
このユーザーのリクエストにリソースが集中する可能性があるので、通常はこれを避けるべきです。<br>
そのためには、バックグラウンドワークプロセスを利用することで、リソースを必要とするプロセスのリクエストをダイアログワークプロセスのバックグラウンドで実行することができます。<br>
そしてバックグラウンドワークプロセスはバックグラウンドで実行されるプロセスを制御します。

![long_running_ABAP](https://github.com/koishi755/SAP/blob/main/AS_ABAP_Processes/long_running_ABAP.png)

<br>

### バックグラウンドワークプロセスの流れ

バックグラウンドワークプロセスのリクエストはユーザーによりスケジュールすることができます。<br>
またバックグラウンドジョブを同時に設定した場合、これは順番に実行されます。<br>
さらに、バックグラウンドワークプロセスで利用可能なオプションもあり、分類したり、優先順位をつけたりすることができます。<br>
ジョブは一般的に、ダイアログワークプロセスでは実行できないリクエストに使用されます。<br>

![background_work_process](https://github.com/koishi755/SAP/blob/main/AS_ABAP_Processes/background_work_process.png)

<br>

### バックグラウンドジョブの登録

#### 現在のパラメータの確認（RZ11）
トランザクションコードRZ11を実行します。<br>
Parameter Nameに「rdisp/max_wprun_time」と入力し、Displayをクリックします。<br>

![rz11](https://github.com/koishi755/SAP/blob/main/AS_ABAP_Processes/rz11.png)

<br>

現在の値を確認すると600となっています。これは10分間を意味しています。<br>
これは10分を過ぎると自動的にユーザーとのセッションを停止します。

![rz11_parameter](https://github.com/koishi755/SAP/blob/main/AS_ABAP_Processes/rz11_parameter.png)

<br>

#### 現在のパラメータの確認（SE38）
トランザクションコードSE38を実行します。<br>
Programに「RSPFPAR」と入力しF8を押します。<br>
(RSPFPARは、SAPのプロファイルパラメータの表示に使用されるトランザクションコードです。)<br>
![SE38](https://github.com/koishi755/SAP/blob/main/AS_ABAP_Processes/SE38.png)

<br>

Profile Parametersに「rdisp*」と入力し、F8を押します。<br>
![SE38_display_profile_parameter](https://github.com/koishi755/SAP/blob/main/AS_ABAP_Processes/SE38_display_profile_parameter.png)

<br>

ここではパラメータのデフォルトの値や関連する値を確認することができます。<br>
![SE38_display_profile_parameter_2](https://github.com/koishi755/SAP/blob/main/AS_ABAP_Processes/SE38_display_profile_parameter_2.png)

<br>

#### ジョブの登録
トランザクションコードSM36を実行します。<br>
![sm36](https://github.com/koishi755/SAP/blob/main/AS_ABAP_Processes/sm36.png)

<br>

ジョブクラスは以下のABCから選択します。<br>
![sm36_job_class](https://github.com/koishi755/SAP/blob/main/AS_ABAP_Processes/sm36_job_class.png)

<br>

次にStepをクリックします。<br>
![sm36_click_step](https://github.com/koishi755/SAP/blob/main/AS_ABAP_Processes/sm36_click_step.png)

<br>


赤枠をクリックします。<br>
![sm36_step1](https://github.com/koishi755/SAP/blob/main/AS_ABAP_Processes/sm36_step1.png)

<br>

「SAP&AUTH」を選択します。<br>
![sm36_Variant](https://github.com/koishi755/SAP/blob/main/AS_ABAP_Processes/sm36_Variant.png)

<br>


[Check]をクリックし、問題が無ければSaveをクリックします。<br>
![sm36_step1_check_and_save](https://github.com/koishi755/SAP/blob/main/AS_ABAP_Processes/sm36_step1_check_and_save.png)

<br>

Backをクリックします。<br>
![sm36_job](https://github.com/koishi755/SAP/blob/main/AS_ABAP_Processes/sm36_job.png)

<br>


「Start condition」をクリックします。<br>
![sm36_clickstart_condition](https://github.com/koishi755/SAP/blob/main/AS_ABAP_Processes/sm36_clickstart_condition.png)

<br>

ジョブは毎日、毎週、または1回だけ行うなど定義することもできます。<br>
今回は「Immediate」（即時実行）をクリックします。<br>
![sm36_start_time](https://github.com/koishi755/SAP/blob/main/AS_ABAP_Processes/sm36_start_time.png)

<br>


Date/Tiemに「Immediate Start」がチェックされるます。<br>
次に画面下部の「Check」をクリックし、問題が無ければ「Save」をクリックします。<br>
![sm36_start_time_entied_Date_time](https://github.com/koishi755/SAP/blob/main/AS_ABAP_Processes/sm36_start_time_entied_Date_time.png)

<br>

Define Job画面に戻りSaveをクリックします。<br>
すると画面下部にジョブがリリースされた旨のメッセージが表示されます。<br>
![sm36_save](https://github.com/koishi755/SAP/blob/main/AS_ABAP_Processes/sm36_save.png)

<br>

#### ジョブの終了確認
トランザクションコードSM37を実行します。<br>
今回はこのまま「Execute」をクリックします。
![sm37](https://github.com/koishi755/SAP/blob/main/AS_ABAP_Processes/SM37.png)

<br>

ジョブが既に終了していることを確認出来ます。<br>
次にこのジョブを選択し、「Spool」をクリックします。<br>
![sm37_job_finished](https://github.com/koishi755/SAP/blob/main/AS_ABAP_Processes/sm37_job_finished.png)

<br>

以下のようにスプールリストが作成されています。<br>
typeをクリックします。<br>
![sm37_type](https://github.com/koishi755/SAP/blob/main/AS_ABAP_Processes/sm37_type.png)

<br>

ここではジョブに実行時に選択した全てのパラメータを確認できます。<br>
このようにジョブの登録から終了まで確認できます。<br>
![sm37_spool_type](https://github.com/koishi755/SAP/blob/main/AS_ABAP_Processes/sm37_spool_type.png)













