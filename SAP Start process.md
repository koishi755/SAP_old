# 1. 起動順序
SAPは以下の順に起動させる必要があります。<br>
SAPシステムにはデータベースが必要なので、まずはデータベースから起動させます。<br>
1. データベース
1. セントラルインスタンス
1. ダイアログインスタンス

![00_SAP_Start_process](https://github.com/koishi755/SAP/blob/main/SAP_Start_process/00_SAP_Start_process.png)

一般的にはエンキューサービスはセントラルインスタンスに含まれます。<br>
ですが、エンキューサービスしか存在しないメッセージサーバーはデータベースを起動させることなく、インスタンスを起動することができます。<br>
SAPを起動する際には、明示的にデータベースを起動する必要はありません。<br>
これは内部でコントロールされており、自動的にデータベースが起動します。<br>
SAPを停止する際は以下の手順で行います。<br>
1. ダイアログインスタンス
1. セントラルインスタンス
1. データベース

# 2. SAPへ接続
SSHを使って、SAPのNPLに接続しています。
使用しているユーザーIDがNPLADMであることがわかります。<br>
これは SIDADM というユーザー名で、SAP システムのインストール時に作成された標準的なユーザーです。<br>
![01_ssh](https://github.com/koishi755/SAP/blob/main/SAP_Start_process/01_ssh.png)

<br>

hostnameコマンドでこのシステムのホスト名を確認します。<br>
ホスト名がsaptestとなっていることを確認できました<br>
![02_login](https://github.com/koishi755/SAP/blob/main/SAP_Start_process/02_login.png)


次に「SAP Management Console」から状態を確認します。<br>
![03_SAP_management_console](https://github.com/koishi755/SAP/blob/main/SAP_Start_process/03_SAP_management_console.png)

<br>

NPLが緑になっています。緑は稼働していることを意味します。<br>
![04_SAP_management_console_status](https://github.com/koishi755/SAP/blob/main/SAP_Start_process/04_SAP_management_console_status.png)

<br>

Process Listをクリックするとこのインスタンスで起動しているワークプロセスが確認できます。<br>
disp+workが存在することからダイアログインスタンスだと確認できます。<br>
igswd_mtはゲートウェイサービスを意味します。<br>
![05_saptest_processes_list](https://github.com/koishi755/SAP/blob/main/SAP_Start_process/05_saptest_processes_list.png)

<br>

AS ABAP WP Tableをクリックするとこのシステムで実行されているワークプロセスを見ることができます。<br>
DIAと書かれているものはダイアログワークプロセスです。<br>
これはトランザクションコード SM50 を使用して SAP システム内で通常見られるものと同じものです。<br>
![06_as_abap_wp](https://github.com/koishi755/SAP/blob/main/SAP_Start_process/06_as_abap_wp.png)

<br>

同様にICMも確認してみます。ステータスがrunningとなっており、現在ICMが起動していることが分かります。
![07_ICM](https://github.com/koishi755/SAP/blob/main/SAP_Start_process/07_ICM.png)

<br>

# 3. ディレクトリ構造

起動プロセスについて見る前にファイルシステムやオペレーティングシステムのファイルがSAPシステムにどのように保存されているかを見てみます。<br>
SAPシステムを動作させるためには、オペレーティング・システムからいくつかのファイルにアクセスする必要があります。<br>
これらのファイルは、SAPシステムに直接保存されています。<br>
例えば、WindowsやUnixなどのOSでは、/usr/sapというディレクトリがあります。<br>

![08_sap_start_process](https://github.com/koishi755/SAP/blob/main/SAP_Start_process/08_sap_start_process.png)

ディレクトリ|説明
-|-
SAPSID|私の環境では、NPLというシステムを使っています。このシステムがABAPスタックを持つセントラルインスタンスの場合、/SAPNPL/というディレクトリが作成されます。インスタンスごとにサブディレクトリがあります。各インスタンスのディレクトリ名は、関連するインスタンスに対応しています。
DVEBMGS|「ABAP」 セントラルインスタンスのためのサブディレクトリです。各文字が1つのワークプロセスを表し、2桁の数字がインスタンス番号になります。<br>ここでは、00がインスタンス番号になります。（プライマリアプリケーションサーバインスタンスのディレクトリ）
DVEBMGS/work | DVEBMGSディレクトリの下に、workというサブディレクトリがあります。<br>一般的にはこのディレクトリは作業用ディレクトリと呼ばれ、この特定のインスタンスのすべてのログを見つけることができます。
DVEBMGS/J2EE|こJava関連のファイルが置かれ、すべてのJavaノードこのディレクトリ内の情報を利用しています。
DVEBMGS/exe|exeの実行ファイルがあります。
D<No.>|ABAP ダイアログインスタンスのためのサブディレクトリです。ABAP ダイアログインスタンスのインスタンス名は D<Instance_Number> となります。<br>この下には、work、J2EE、exeの実行ファイルがあります。（追加のアプリケーションサーバインスタンスのディレクトリ）
SCS|Java セントラルサービスインスタンス (SCS) のディレクトリです。ABAPの場合だと次のようになります。<br>ASCS<No.>
SYS|複数のインスタンスで使用されるデータを保存するために使用する、/sapmnt/SAPSID 内の該当するディレクトリへのシンボリックリンクが含まれています。SYS は論理的に共有され、SAP システムの各ホストで使用することができます。このサブディレクトリには、SAP グローバルホスト上にある /sapmnt/SAPSID に対応するサブディレクトリへのシンボリックリンクが含まれています。
SYS/global|ログファイルが含まれています。分散インスタンスを使用している SAP システムでは、このディレクトリを同じオペレーティングシステムのすべてのホストで共有する必要があります。
SYS/profile|すべてのインスタンスの起動プロファイルと操作プロファイルが含まれています。分散インスタンスを使用している SAP システムでは、このディレクトリを同じオペレーティングシステムのすべてのホストで共有する必要があります。
SYS/exe|また、exeというフォルダがありますがこれは実行可能なカーネルファイルです。これらのディレクトリは、あるシステムから別のシステムへ、共有名"$sapmnt "を使って共有されます。

## 実際のディレクトリ構造の確認

では、実際のOSにログインして実際のディレクトリ構造を確認してみます。<br>
まずは/(root) 配下にsapmntがあり、その下にSAPSID(NPL)ディレクトリがあります。<br>
![09_sapmnt](https://github.com/koishi755/SAP/blob/main/SAP_Start_process/09_sapmnt.png)

<br>

次に/(root)配下に/usr/sapというディレクトリがあることを確認してみます。<br>
そしてこのディレクトリ配下にもSAPSID(NPL)ディレクトリがあります。<br>
![10_usr_sap](https://github.com/koishi755/SAP/blob/main/SAP_Start_process/10_usr_sap.png)

<br>

SAPSID(NPL)ディレクトリに移動してみます。<br>
ここでは、最初にディレクトリ構造を確認した時のディレクトリが見られますが、私のテスト環境ではDVEBMGS00が存在しませんでした。<br>
その他のABAP用のASCS01とSYSはありますね。<br>
![11_usr_sap_sid](https://github.com/koishi755/SAP/blob/main/SAP_Start_process/11_usr_sap_sid.png)

<br>

SYSの中には「gen」「exe」「global」「profile」があります。<br>
![12_usr_sap_sid_sys](https://github.com/koishi755/SAP/blob/main/SAP_Start_process/12_usr_sap_sid_sys.png)

<br>

プロファイルには、さまざまな種類のプロファイル、デフォルトプロファイル、インスタンスプロファイル、そしてスタートアッププロファイルがあります。<br>
（一般的にはSTART_DVEBMGS00_という形式のようです。）<br>
![13_usr_sap_sid_sys_profile](https://github.com/koishi755/SAP/blob/main/SAP_Start_process/13_usr_sap_sid_sys_profile.png)

<br>

それはSAPシステムは、起動するためにいくつかの値を必要とします。例えば、いくつのワークプロセスを持つのか、どのくらいのメモリを占有するかということです。<br>
これらのパラメータはプロファイルのパラメータで管理されます。<br>
これらのプロファイルの意味は以下の通りです。<br>

プロファイル|説明
-|-
スタートプロファイル|最初に読み込まれるプロファイルです。スタートプロファイルはすべてのインスタンスに適用されます。<br>そのため、SAPシステムを起動するコマンドを実行すると、スタートプロファイルの値がすべてのインスタンスに適用されます。<br>スタートプロファイルには、SAPシステムを起動させるための共通の値であり、必要な情報や構成が定義されています。<br>
デフォルトプロファイル|2番目に読み込まれるプロファイルです。<br>デフォルトプロファイルも、すべてのインスタンスに適用されます。<br>例えば、パスワードの長さのようなセキュリティパラメータがあるとします。<br>このようなパラメータは、システム全体で定義する必要があるため、デフォルトプロファイルで定義します。そうすれば、すべてのシステムで利用できるようになります。
インスタンスプロファイル|特定のインスタンスに特化したプロファイルです。<br>特定のインスタンスにダイアログワークプロセスやその他のワークプロセスの数を定義します。
