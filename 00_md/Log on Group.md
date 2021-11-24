# 1. ログオングループ
ログオングループとは負荷分散の方法の1つです。<br>
大規模なSAPシステムでは、複数のサーバーが存在します。その場合、どのようにしてロードバランスを取るかを検討する必要があります。<br>
ここでは SAPGUI ユーザーがいて、セントラルインスタンスとアプリケーションインスタンスがあり、データベースがあります。（下図の構成）<br>
シナリオ１つとして、１つログオングループを作成し、セントラルインスタンスやアプリケーションインスタンスを入れることです。

ある地域、例えばアジア太平洋のユーザー数がヨーロッパのユーザー数よりも多い可能性があります。<br>
そのような場合、特定の地域に特定のリソースを分割して割り当てたいとします。<br>
そこで、異なるログオングループを作成し、異なるアプリケーションインスタンスまたはセントラルインスタンスに割り当てます。<br>
セントラルインスタンスを複数のログオングループに割り当てることができます。<br>
同様に、アプリケーションインスタンスにも複数のログオングループを割り当てることができます。<br>
1つのアプリケーションインスタンスが1つのログオングループに割り当てられても、他のグループに割り当てられないということではありません。

![Log on Group](https://github.com/koishi755/SAP/blob/main/logon_group/logon_group.png)

# 2. ログオングループの作成
## 2.1. ログオングループの作成

Tools →　CCMS　→　Configuration　→　Logon Groupsをクリックします。<br>
またはトランザクションコード「SMLG」を実行します。<br>
![１ログオングループ](https://github.com/koishi755/SAP/blob/main/logon_group/1logon.png)

<br>

Create assignment をクリックします。<br>
![１ログオングループ](https://github.com/koishi755/SAP/blob/main/logon_group/2logon.png)

<br>

Logon Group の名前を入力します。<br>
![ログオングループ名入力](https://github.com/koishi755/SAP/blob/main/logon_group/3logon.png)

<br>

次にInstanceテキストボックスの横の検索ボタンをクリックします。<br>
![インスタンス検索](https://github.com/koishi755/SAP/blob/main/logon_group/search.png)

<br>

インスタンスが表示されるので選択します。<br>
![インスタンス選択](https://github.com/koishi755/SAP/blob/main/logon_group/4logon.png)

<br>

ログオングループ、インスタンスの入力が終わったら、Copyをクリックします。<br>
![インスタンス選択](https://github.com/koishi755/SAP/blob/main/logon_group/5logon.png)

<br>

インスタンスが追加されてことを確認してSaveをクリックします。<br>
![インスタンス選択](https://github.com/koishi755/SAP/blob/main/logon_group/6logon.png)

<br>

## 2.2. サービスとポート番号の登録

以下のフォルダのserviceファイルを編集します。

```
C:\Windows\System32\drivers\etc
```

![インスタンス選択](https://github.com/koishi755/SAP/blob/main/logon_group/7logon.png)

<br>

サービスとポート番号は次のような形式で追加します。<br>

```
sapms<SID>    メッセージサーバのポート/tcp
```

メッセージサーバのポートは基本的には、36\<Instance number>　という番号になります。

![インスタンス選択](https://github.com/koishi755/SAP/blob/main/logon_group/8logon.png)

## 2.3. ログオングループから接続

SAP GUI から New Itemをクリックします。<br>
![インスタンス選択](https://github.com/koishi755/SAP/blob/main/logon_group/9logon.png)

<br>

User Specified Systemを選択し、Nextをクリックします。<br>
![インスタンス選択](https://github.com/koishi755/SAP/blob/main/logon_group/10logon.png)

<br>

Connection Typeより、Group/Server Selectionを選択します。<br>
![インスタンス選択](https://github.com/koishi755/SAP/blob/main/logon_group/11logon.png)

<br>

System Configuration Parametersを以下のように入力し、Finishをクリックします。<br>

| 項目| 説明 |
-|-
|Description|任意|
|SystemID|対象のSIDを入力します|
|Message Server|メッセージサーバのIPアドレス、または名前解決したホスト名を入力します|
|SAProuter|今回は空白|
|Group/Server|SystemID、Message Serverに問題が無ければ自動で選択出来るようになります|

![インスタンス選択](https://github.com/koishi755/SAP/blob/main/logon_group/12logon.png)

<br>

作成したログオングループを検索して、Log Onを試してみます。<br>
![インスタンス選択](https://github.com/koishi755/SAP/blob/main/logon_group/13logon.png)

<br>

以下の画面が表示されれば問題ありません。
![インスタンス選択](https://github.com/koishi755/SAP/blob/main/logon_group/14logon.png)


