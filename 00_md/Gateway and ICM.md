# Gateway

ゲートウェイの機能は以下の通りです。<br>
* あるシステムから別のSAPシステムへの通信。
* あるインスタンスから別のインスタンスへの通信。
* CPIC、RFCの通信。
* 一般的にはインバウンドコールに使用され、インバウンドコールがあるとゲートウェイを経由でディスパッチャーに送られます。


![00_gateway](https://github.com/koishi755/SAP/blob/main/gateway_ICM/00_gateway.png)

<br>

非常に小さなインストールであれば、ゲートウェイを設定しません。<br>
そのため、ゲートウェイがどのように使用されているかを意識しません。<br>
デフォルトでは、SAPインスタンスにインスタント番号で終わる名前が存在します。<br>

<br>

# ICM

ICMはInternet Communication Manager(Protocol)と呼ばれ、Webブラウザからの情報を受信します。<br>
ICMは、受け取ったリクエストがJava関連のリクエストなのか、それともABAPのリクエストなのかを判断します。<br>
もしそれがJavaのリクエストであれば、J2EEディスパッチャを経由してJavaエンジンに送られます。<br>
ABAPエンジンとJAVAエンジンはJCoコネクションで接続されています。JcoもRFCコネクションのようなもので、JavaとABAPの間で内部通信を行います。<br>

![01_ICM](https://github.com/koishi755/SAP/blob/main/gateway_ICM/01_ICM.png)

<br>
