# 1.プロジェクト及びコンテナの作成と動作の確認

## 1-1.プロジェクトの作成
 OpenShiftは、”プロジェクト”　単位でアプリケーションや権限などを管理しています。  
 アプリケーションを作成するにはまずプロジェクトを作成します。  
 
 　　![project-new](./2-1-1.jpg)
   
### ***OpenShift のWebコンソールにログインし、"New Project" をクリックします。***  
 　※Webコンソールのアドレスについては別途ご確認ください。


　![project-name](./2-1-2.jpg)

### ***「Name」の欄に、"nexus-your-name"を入力し、「Create」をクリックします。***

※your-name にはご自身のお名前をローマ字で入力ください。今回利用する環境はDemo環境で、利用者共有のシステムとなっています。OpenShiftでは、プロジェクト名は一意でなければなりません。プロジェクト名にご自身の名前を入れるのは、プロジェクト名の重複を避けるためです。



## 1-2.コンテナのデプロイメント
プロジェクト内にアプリケーションを作成してみます。  
　![project-Deploy1](./2-2-1-2.jpg)
### ***「Deploy Image」をクリック。「Image Name」 欄に、”sonatype/nexus3”を入力の上、右端の*** 🔍アイコンをクリックします。下方に画面をスクロールし、「Create」をクリックします。

※Sonatype が提供するDockerレポジトリからのコンテナ取得を行っています。また、詳細パラメータでは、コンテナに独自の構成情報を付加する「環境変数」と、リソースをグループ化し、アクセス制限など細かな制御を実現する「ラベル」の設定が可能であることを確認します（今回は設定しません）。

　![project-Deploy1](./2-2-3.jpg)
### ***「Continue to overview」をクリックします。***


　![project-Deploy1](./2-2-4.jpg)
 
「Project」の「Overview」ページが開きます。「Nexus3」というサービスが作成され、Podが1つデプロイされていることが分かります。  
上下の矢印を
### ***クリック***
し、Podが増減することを確認します。 
コンテナをデプロイするだけでは外からの接続が行えません。そこで、
### ***「Create Route」***
をクリックし、「Nexus3」サービスに対するルートを作成します。


 
 ルート作成では、以下のようなオプションが設定可能です。（今回は指定しません）

HostName：
アプリケーションのDNS名を指定
（指定しない場合は自動で作成）
Path：
　サービスへの追加URL（サブディレクトリ）
Target Port：
　ネットワークポートの指定
Security：
　TLSターミネーションタイプの変更

 
 　![project-Deploy1](./2-2-5.jpg)
  
  サービスへのルートが作成されました。クリックしてみます。
  
 　![project-Deploy1](./2-2-6.jpg)

2-2-7.
Nexusリポジトリマネージャーが起動し、接続できることを確認します。サインインに関しては以下のアカウントが利用可能です。

admin/admin123

※サービスが完全に立ち上がるまでに少し時間がかかります。もし接続できないときは、少し時間をおいて確認してみましょう。


![project-Deploy1](./2-2-7.jpg)

2-2-8.
動作が確認できたら、、ホームページに戻り、プロジェクトを削除します。

## 2.使用リソースの確認
　新しくアプリケーションをデプロイし、そのアプリケーションが使用しているCPUやメモリリソースについて確認してみましょう。

### 2-1.テンプレートの利用とリソースの確認
　まずはプロジェクトを作成します。

![project-Deploy1](./3-1-1.jpg)
名前を入力し、「Create」をクリックします。

※名前はご自由に入力ください。但し、OpenShif内で一意である必要があります。

![project-Deploy1](./3-1-2.jpg)

今度はテンプレートからデプロイしてみます。「Browse Catalog」をクリックし、"cakephp" を入力。
CakePHP +MySQL　の「Select」をクリックします。

![project-Deploy1](./3-2-1.jpg)

サービス名、PHPとMySQLコンテナがそれぞれ利用可能な、最大メモリの値、アプリのソースコードが含まれるGitリポジトリが指定できることを確認します。


![project-Deploy1](./3-2-2.jpg)

さらに、データベースのユーザーやパスワード等の設定やラベルをここで定義できることを確認します。指定しない場合は、自動的に作成されます。これらの値を作成して、フロントエンドのアプリケーションとバックエンドのデータベースのデプロイを行います。
一番下の「Create」をクリックし、デプロイを開始します。

![project-Deploy1](./3-2-3.jpg)

oc コマンドツールのダウンロートサイトへのリンクや、webhookトリガーに簡単にアクセスできることを確認します。

![project-Deploy1](./3-2-4.jpg)

Overviewページで以下を確認します。
・Podが二つデプロイされていること
・ルートが既に作成されていること
・Build状態を確認するためのリンク
・サービスへのリンク
さらに、レプリカの数の増減が出来ること、リソースの消費量が確認できます。

![project-Deploy1](./3-2-5.jpg)

※リソースが表示されない場合は以下を確認しブラウザのアクセス制限を解除下さい。

「Applications」→「Pods」を選択。
「cakephp-xxxx...」をクリックし、「Metrics」をクリック
ブラウザのセキュリティを解除してください

![project-Deploy1](./3-2-6.jpg)

### 2-2.サービスとビルドの確認
　今作成したアプリケーションに関するサービスとビルドについて詳細を確認してみましょう。
 「Application」→「Services」をクリック。
![project-Deploy1](./3-3-1.jpg)

「mysql」サービスをクリック。
![project-Deploy1](./3-3-2.jpg)

ルート、ポッドなどを確認することができます。

![project-Deploy1](./3-3-3.jpg)

次に、ビルド情報を見てみます。
「Builds」→「Builds」を選択。ここにはプロジェクト内のすべてのビルド、及びそのステータス、ソースへのリンク、デプロイがいつ行われたかなどが表示されます。一通り確認が終了したら、「cakephp-xxxx...」をクリック。

![project-Deploy1](./3-3-4.jpg)

右上の「Start Build」をクリックしてみましょう。
#2が追加されることが分かります。#1、#2をクリックして内容を確認してみます。

![project-Deploy1](./3-3-5.jpg)

こちらは#1の表示例です。以下を確認します
・「Rebuild」のクリックで再ビルド実行可能
・ビルドのステータスとトリガー理由
・ソース情報
　など
「Logs」をクリックします。

![project-Deploy1](./3-3-6.jpg)

ソースコードリポジトリがコンテナとしてダウンロードされていることが分かります。一番下の方を確認すると、完了したイメージがプロジェクト名を追加されてレジストリに追加されていることが分かります。

![project-Deploy1](./3-3-7.jpg)

ソースコードリポジトリがコンテナとしてダウンロードされていることが分かります。一番下の方を確認すると、完了したイメージがプロジェクト名を追加されてレジストリに追加されていることが分かります。

![project-Deploy1](./3-3-8.jpg)

ソースコードリポジトリがコンテナとしてダウンロードされていることが分かります。一番下の方を確認すると、完了したイメージがプロジェクト名を追加されてレジストリに追加されていることが分かります。

![project-Deploy1](./3-3-9.jpg)

・デプロイに利用されたテンプレート情報
・ポート情報
・メモリ制限
・永続ボリュームが追加可能
などが確認できます。

また、パフォーマンスがしきい値をオーバーした際のオートスケール設定や、リソース使用量の制限も設定可能です。

![project-Deploy1](./3-3-10.jpg)

「Environment」タブをクリックします。
デプロイ時に設定された環境変数が確認できます。環境変数はPod内で、ユーザー名やデータベース名など様々なパラメーターを設定することが可能で、また、ユーザー自身での追加も可能です。
確認が終了したら、「Events」タブをクリックします。

![project-Deploy1](./3-3-11.jpg)

デプロイに関する様々なイベントを確認できます。一連の作業内容の確認や、デプロイの際のトラブルの確認などに利用可能です。

![project-Deploy1](./3-3-12.jpg)

### 2-3.Podの確認
次にPodの確認を行ってみましょう。
「Applications」→「Pods」をクリックします。

![project-Deploy1](./3-4-1.jpg)


プロジェクト内で有効なPodが表示されることを確認します。「Completed」、「Running」などのステータスが確認できます。Podの一つをクリックします。

![project-Deploy1](./3-4-2.jpg)

「Details」タブで以下が確認できます。
・Podの状態
・ホストするOpenShiftノード
・イメージ
・ビルド（#含む）
・CPU、メモリの利用範囲
・永続ストレージの追加
　など。
確認が終わったら、「Metrics」タブをクリックします。

![project-Deploy1](./3-4-3.jpg)

メトリックスタブにはPodで使用されるリソースが表示されます。
・利用可能な最大メモリと、実使用量
・利用可能な最大CPUと実使用量
・ネットワーク使用量も表示されます

一通り確認したら、「Logs」タブをクリックします。

![project-Deploy1](./3-4-4.jpg)

View Archive をクリックすると、Kibanaログシステムに接続できます。ここでは、過去から現在まで、全てのPodのログを表示できます。
「Terminal」タブをクリックします。

![project-Deploy1](./3-4-5.jpg)

ターミナルでは、Podの中のいずれかのコンテナ内のターミナルを利用できます。

「Event」をクリックします。

![project-Deploy1](./3-4-6.jpg)

「Event」タブでは、Podに関するイベントが表示されます。

![project-Deploy1](./3-4-7.jpg)

### 2-4.ルートの確認
次にルートの確認を行ってみましょう。
「Applications」→「Services」をクリック。
プロジェクトに関連する全サービスが表示されます。
・クラスタIP
　サービスに接続するためのIPアドレス
・ポート
　サービスが待機しているポート番号

![project-Deploy1](./3-5-1.jpg)
「Applications」→「Routes」をクリックし、ルートが表示されることを確認します。
![project-Deploy1](./3-5-2.jpg)

## 3.クォータの設定と確認
一つのプロジェクトがリソースを食いつぶしてしまわないよう、プロジェクトにはクォータの設定が可能です。プロジェクトの管理者・利用者はあらかじめ定められたリソース内でリソースを利用することが可能です。

![project-Deploy1](./4-1-1.jpg)

クォータでは、以下が確認できます。
・このプロジェクトと全プロジェクトの利用CPU/メモリ
・このプロジェクトと全プロジェクトに設定されているクォータ
・Pod、サービスの数など

![project-Deploy1](./4-1-2.jpg)

クォータの効果を確認するため、「Overview」をクリックし、Podを7に増やします。
![project-Deploy1](./4-1-3.jpg)

クォータに戻ります。Pod数が増えていること、クォータ制限にかかりそうなリソースが一目瞭然であることを確認します。クラスタ管理者が設定したポリシーに従って、プロジェクトごとにクォータ設定が与えられ、ユーザーはそのクォーター内でデプロイが可能です。
![project-Deploy1](./4-1-4.jpg)

終了したら、、ホームページに戻り、プロジェクトを削除します。
![project-Deploy1](./4-1-5.jpg)

## 4.ログとメトリックスの収集
アプリケーションをデプロイし、メトリックスの収集と表示、ログ表示について学びます。

### 4-1.テキストログの確認
新規にプロジェクトを作成します。
![project-Deploy1](./5-1-1.jpg)

"eap64" を入力し、「jboss-eap-openshift」を選択します。
![project-Deploy1](./5-1-2.jpg)

以下を入力して作成します。
S2Iであることも確認しておきます。
Name:
 openshift-tasks
Git Repo:  https://github.com/openshiftdemos/openshift-tasks

![project-Deploy1](./5-1-3.jpg)

Continue to overviewをクリックし、Overview画面を表示させます。
![project-Deploy1](./5-1-4.jpg)

自動的にルートが作成されていることを確認し、「View Log」をクリックします。
![project-Deploy1](./5-1-5.jpg)

最終行を追いたい時には、「Follow」、特定の位置で止めたい場合は、「Stop Following」を選択
![project-Deploy1](./5-1-6.jpg)

ソースコードがダウンロードされ、イメージが作成されていることを確認します。
後のデプロイメントに利用するためのデータもコピーされていることが分かります。これは、例えば、Jenkins Server上のMavenビルドなど、他の場所で実行された場合でも作成されます。確認終了後、「Overview」を表示します。

![project-Deploy1](./5-1-7.jpg)

### 4-2.メトリックスの確認とKibanaを利用したLogの閲覧
ロードジェネレーターを利用して、CPUに負荷をかけ、その様子を確認してみましょう。
![project-Deploy1](./5-2-1.jpg)

値（負荷発生時間）に300を入力し、「Load」をクリックします。

![project-Deploy1](./5-2-2.jpg)

「Overview」をクリックし、CPUに負荷がかかっていることを確認します。
![project-Deploy1](./5-2-3.jpg)

アプリケーション実行中のPodをクリックします。
![project-Deploy1](./5-2-4.jpg)

「Metrics」タブをクリックします。
![project-Deploy1](./5-2-5.jpg)

Podで使用されているMemory、CPU、Networkのリソース利用量が表示されることを確認します。ロードジェネレーターを起動させたので、CPUの負荷が一時的に上がっていることが分かります。
![project-Deploy1](./5-2-6.jpg)

アプリケーションページに戻り、「Logger」にある各Logを数回クリックし、メッセージを生成します。

![project-Deploy1](./5-2-7.jpg)


「Overview」で、Podを3Podsに増やします。
![project-Deploy1](./5-2-8.jpg)


3つのPodのうち、一つをクリックします。
![project-Deploy1](./5-2-9.jpg)

「Logs」タブをクリックします。Kibanaを使ってLog内容が表示されています。Kibanaでは、実行中・停止中にかかわらず全てのPodのLog情報が閲覧可能です。「View Archive」をクリックします。

![project-Deploy1](./5-2-10.jpg)

上部フィールドには、特定のPodを示すフィルターが入力されていることを確認します。また、下部には、メッセージが表示されていることを確認します。

※環境依存でうまく動かないケースもある様です。うまくいかない場合は、6.に進んでください。


![project-Deploy1](./5-2-11.jpg)

フィルターの最初の部分を削除して、プロジェクト内すべてのPodを表示します。

![project-Deploy1](./5-2-12.jpg)

さらに、*error*を最後に追加して、全エラーメッセージが表示される様にします。

![project-Deploy1](./5-2-13.jpg)

確認が終了したら、、ホームページに戻り、プロジェクトを削除します

![project-Deploy1](./5-2-14.jpg)

## 5.環境変数の設定と確認
アプリケーション作成の際に環境変数を設定し、コンテナに設定が反映されることを確認してみましょう。
　新しくアプリケーションをデプロイし、そのアプリケーションが使用しているCPUやメモリリソースについて確認してみましょう。

### 5-1.環境変数の設定と動作の確認
　まずはプロジェクトを作成します。

![project-Deploy1](./6-1-1.jpg)
### ***「New Project」をクリックします。***

![project-Deploy1](./6-1-2.jpg)
### ***名前を入力し、「Create」をクリックします。***

※名前はご自由に入力ください。但し、OpenShif内で一意である必要があります。

![project-Deploy1](./6-1-3.jpg)
### ***テンプレートの選択画面で、"Node"と入力し、Node.js + MongoDB (Ephemeral)を選択します。***

![project-Deploy1](./6-1-4.jpg)

(確認のみ)このテンプレートをデプロイするために利用されるイメージが表示されています。

![project-Deploy1](./6-1-5.jpg)
### ***下にスクロールし、環境変数として「Administrator Password」欄に適当なパスワードを入力してみましょう。入力がすんだら、「Create」をクリックします。***

![project-Deploy1](./6-1-6.jpg)
### ***「Applications」→「Pods」をクリックします。***

![project-Deploy1](./6-1-7.jpg)
### ***nodejs-mongodb-example-xxxをクリックします。***

![project-Deploy1](./6-1-8.jpg)
### ***Terminalタブを開き、echo $MONGODB_ADMIN_PASSWORDを入力し、デプロイ時に設定したパスワードが表示されることを確認します。
さらに、以下のコマンドで、OpenShiftが自動生成したパスワードも確認します。  
### ***echo $MONGODB_PASSWORD***

![project-Deploy1](./6-1-9.jpg)
以下のコマンドを入力してみます。
### ***ls***  
### ***ps -ef***
　コンテナ内のプロセス確認
### ***env |grep -i mongo***
　環境変数の確認
### ***cat /etc/resolv.conf***
　DNS設定の確認

![project-Deploy1](./6-1-10.jpg)

確認が終了したら、ホームページに戻りプロジェクトを削除します。  


## 6.環境変数によるアプリケーションの動作と再デプロイ
異なるアプリケーションの動作を環境変数によって実現してみましょう。また、構成変更した後にアプリケーションの再デプロイを行い、その動きについて確認してみます。

### 6-1.環境変数の設定と動作の確認
　まずはプロジェクトを作成します。

![project-Deploy1](./7-1-1.jpg)
### ***「New Project」をクリックします。***

![project-Deploy1](./7-1-2.jpg)
### ***名前を入力し、「Create」をクリックします。***

![project-Deploy1](./7-1-3.jpg)
### ***カタログ選択画面で、"php"を入力し、PHPを選択します。***


![project-Deploy1](./7-1-4.jpg)
### Name欄に、"cotd"、GitURLに、https://github.com/StefanoPicozzi/cotd を入力。さらに、「advanced options」をクリックし、下の方へスクロールします。***

![project-Deploy1](./7-1-5.jpg)
### ***Deployment Configurationsセクションで、環境変数として、"SELECTOR cities" を入力します***

![project-Deploy1](./7-1-6.jpg)
### さらに、Labelsセクションで "ab cotd" を設定します。入力が終了したら、「Create」をクリックします。***

![project-Deploy1](./7-1-7.jpg)
（確認のみ）アプリケーションの環境変数が期待通りに設定されていることを確認します。この環境変数により、Webアプリケーションで表示される写真が変わります。今回設定したのは、「街」の写真です。


![project-Deploy1](./7-1-8.jpg)
### ***「Overview」で、アプリケーションのルートをクリックします。***  
アプリケーションがデプロイされていること設定した環境変数により、「街」の写真が表示されることを確認します。
### ***さらに、Podアイコン右側の上矢印を2度クリックし、3Podにスケールアップします。***

### 6-2.アプリケーションの再デプロイ
作成したアプリケーションを再デプロイしてみます。

![project-Deploy1](./7-2-1.jpg)
### ***「Applications」→「Deployments」をクリックします。***


![project-Deploy1](./7-2-2-1.jpg)
![project-Deploy1](./7-2-2-2.jpg)

### ***「Deploy」をクリックしすぐに「Overview」をクリックします。***
新しいPodが必要なレプリカ数（今回の場合3個）になるまで増え、逆に、旧Pod新Podが作成される度に1つずつ削除されることを確認します。
この様に、Podはローリングアップデートされます。


![project-Deploy1](./7-2-3.jpg)
### ***「Overview」から、アプリケーションへのショートカット、「cotd」をクリックします。***

![project-Deploy1](./7-2-4.jpg)
### ***「Environment」タブで、「deployment configuration」をクリック。環境変数の値を、”cats”に変更し、Saveします。***
### ***すぐに、「Overview」に戻り、Podが再デプロイされていることを確認します。***
### ***さらに、アプリケーションへのリンクをクリックし、先ほどの「街」ではなく、「ネコ」が表示されることを確認します。***

![project-Deploy1](./7-2-5.jpg)
### ***再度アプリケーションへのショートカットをクリックします。***


![project-Deploy1](./7-2-6.jpg)
### ***「Configuration」タブを表示させ、StrategyがRollingであることを確認。その後、「Actions」から、「Edit YAML」をクリックします。
※オートスケーラーやストレージの追加、リソース制限等もこちらから設定可能であることを確認します。

![project-Deploy1](./7-2-7.jpg)
### ***上記の例に従って、Strategy の中を編集し、保存します。***


![project-Deploy1](./7-2-8.jpg)
### ***「Deploy」をクリックし、「Overview」を表示します。

![project-Deploy1](./7-2-9.jpg)
（確認のみ）Recreateに変更したことにより、レプリカのすべてが0にスケールダウンされた後、新しいPodがデプロイされることを確認します。表示上の時間が短く分かりにくい点もありますが、上記のように推移します。

## 7.アプリケーションのライフサイクル管理
Jenkinsを使ったアプリケーションライフサイクルの管理を行います。まず、プロジェクトを3つ作成しJenkinsのデプロイを行います。  
このラボは、コマンドラインを多用しますので、SSHで接続できることをまず確認してみます。

![project-Deploy1](./8-1-1.jpg)
OPENTLC アカウントを利用し、sshで接続します。  

##### ***```$ ssh -i ~/.ssh/yourprivatekey.key opentlc-user@ocplab-guid.oslab.opentlc.com```***  
続いて、OpenShift環境へログインです。ご自身のID　Passwordを使って、OpenShiftマスターにログインします。  
##### ***```$ oc login https://master.na1.openshift.opentlc.com```***

３つの新しいプロジェクト(dev/test/ prod)を作成します。コマンドを4個実行します。 随時、OpenShiftのGUIでも作成の様子を確認してみましょう。

    $ GUID=yourname
    $ oc new-project pipeline-${GUID}-dev --description="Cat of the Day Development Environment" --display-name="Cat Of The Day - Dev"
    $ oc new-project pipeline-${GUID}-test --description="Cat of the Day Testing Environment" --display-name="Cat Of The Day - Test"
    $ oc new-project pipeline-${GUID}-prod --description="Cat of the Day Production Environment" --display-name="Cat Of The Day - Prod"
    
    




    [vagrant@master ~]# ssh-keygen
    Generating public/private rsa key pair.
    Enter file in which to save the key (/home/vagrant/.ssh/id_rsa):  ← リターンを入力
   




