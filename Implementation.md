# 1.はじめに（本コンテンツの前提等）  
　本コンテンツは、Red Hatがパートナー様向けに提供しているオンラインLAB環境（OPENTLC）を利用したF2Fでのハンズオントレーニングを想定しています。資料としては、一般的にも利用できる流れにはなっていますが、OPENTLCならではの表記もございますのでご了承ください。また、インプリメンテーションの無い、OpenShiftの使い方に関するLABコンテンツはこちらをご利用ください。
 
 https://github.com/hiro52/ocp-fastrax/blob/master/LAB.md
 
# 1-1.環境の説明  
　今回OpenShiftのインストールを行う環境は以下を想定しています。環境はOPENTLC前提です。
 
 ・Workstation（SSH接続を行うための踏み台ホスト）  
 ・NFS  
 ・Load Balancer  
   　外部：loadbalancer.$GUID.example.opentlc.com  
   　内部：loadbalancer1.$GUID.internal  
 ・OpenShift マスターサーバー x 3 台  
   　外部：master{1,2,3}.$GUID.example.opentlc.com  
   　内部：master{1,2,3}.$GUID.internal  
 ・OpenShift インフラノード x 2 台  
   　外部：infranode{1,2}.$GUID.example.opentlc.com  
   　内部：infranode{1,2}.$GUID.internal  
 ・OpenShift ワーカーノード x 2 台  
  　 外部： node{1,2}.$GUID.example.opentlc.com  
  　 内部：node{1,2}.$GUID.internal  

 
# 1-2.前準備とOpenShift3.7のインストール  
 では、上記環境にOpenShift3.7をインストールしてみましょう♪  
 まずは、WorkstationにSSH接続し、以下進めます。  
  ※接続先情報等は別途ご確認ください。  

　rootユーザーになって、LAB環境で利用するGUIDを変数に入れておきましょう。  

    # sudo -i
    # echo ${GUID}
    # export GUID=$(hostname | cut -d. -f2)
    # echo ${GUID}; echo "export GUID=${GUID}" >> /root/.bashrc

　OpenShift マスターノードにssh接続し、同じことを実施します。  
 
    # ssh master1.$GUID.internal
    # sudo -i
    # export GUID=$(hostname | cut -d. -f2)
    # echo $GUID; echo "export GUID=${GUID}" >> /root/.bashrc

  exit2回で、Workstation端末に戻りWorkstation上で以下継続します。  

    # exit 
    # exit 

  今回のLAB環境には、Ansibleを使った環境チェックのためのHostsファイルが作成されています。中身を見てみましょう。  

    # cat /etc/ansible/hosts 

  （確認のみ）Ansibleのコマンドを叩いて、ホストの内容を確認し、Ansible ping で応答も見てみましょう♪

    # ansible masters --list-hosts
    # ansible nodes --list-hosts
    # ansible all --list-hosts
    # ansible all -m ping

   Playbook 実行時のログを有効にします。

    # sed -i 's/#log_/log_/' /etc/ansible/ansible.cfg

   今回の環境には、マスターサーバー、インフラノード、ワーカーノードに既にDockerがインストールされ、デーモンが起動しています。以下のコマンドで確認してみましょう。  

    # ansible -f 10 nodes -m shell -a "systemctl status docker | grep Active"
  
  ![project-Deploy1](./1-1n.jpg)

   いよいよインストールです。OpenShiftのインストールにはAnsibleを利用します。  
   インストールのためのインベントリファイルは、/var/preserve/hosts にありますので、リネームしコピーします。

    # cp /var/preserve/hosts /root/my_ocp_inventory
    
  OpenShiftインストールにはこの構成ファイルを利用します。中身をご説明します。
  
    # vi /root/my_ocp_inventory

  1. Ansibleがコマンド実行の際、root権限を利用することの許可と、接続時にはec2-userを利用することの指定です。  
  2. インストールタイプとして、Red Hat のOpenShiftを選択しています。  
  3. コンテナでのインストールも可能ですが、ここでは、Falseを選択し、RPMインストールを行います。  
  4. デプロイメントの際、リソースが少ないとエラーとなります。今回の環境は潤沢なリソースがないので、チェックを回避するオプションを入れています。  
  5. ポッドが、env=appラベルの付いたノードで実行される様指定しています。  

 
  ![project-Deploy1](./1-2n2.jpg)
  
    
　さらに下の方を確認しましょう。
 
  1. cockpit のインストールを行う指定とそのパラメータの設定です。  
  2. OpenShiftのインストールと共に追加されるプロジェクトの指定を行います。  

  ![project-Deploy1](./1-3n2.jpg)

  ### OpenShiftのマスターに対する設定を行います。  

  1. OpenShift のAPIで利用するポート番号をしています。デフォルトは、8443です。  
  2. OpenShiftクラスタで利用するHAに関する設定を行います。現状サポートされるのは"native"のみです。  
  3. ロードバランサーのプライベート側のアドレスを指定します。  
  4. ロードバランサーのパブリック側のアドレス、および、デフォルトのサブドメインを指定します。  
  
  ![project-Deploy1](./1-4n.jpg)
  

  ### OpenShiftのネットワークに対する設定を行います。  

  1. ポッドが利用するIPアドレスレンジを指定します。デフォルトは10.1.0.0/16です。  
  2. サービスが利用するIPアドレスレンジを指定します。デフォルトは、172.30.0.0/16です。  
  3. SDNの指定です。ここでは標準のSDNを指定しています。マルチテナントプラグインを利用する場合は、redhat/openshift-ovs-multitenantを指定します。  
  
  ![project-Deploy1](./1-5n.jpg)


  ### OpenShiftの認証に関する設定を行います。   

  1. OpenShift マスターサーバでhtpasswdの仕組みを利用すること、及び構成ファイルの場所を指定しています。  
  2. 3台のマスターサーバに対して配布するWorkstation 状の構成ファイルの場所を指定しています。  

  ![project-Deploy1](./1-6n.jpg)

  後ほど、ユーザー名：karla, andrewでOpenShift環境に接続します。既にこのユーザーが作成されていることを確認しておきます。
  コンソール画面に戻って、以下のコマンドでユーザーを確認sてみましょう。
  
    # cat /root/htpasswd.openshift

  ### OpenShiftのLogに関する設定を行います。   

　クラスターのロギングはデフォルトでは設定されませんので、こちらで有効化します。  
 
  1. ログとメトリックスに関するインストールの可否を指定します。  
  2. ストレージコンポーネントに関する設定を行います。  
  3. cassndra/hawkular/heapster の稼働するホストをノードセレクターで指定しています。  

  ![project-Deploy1](./1-7n.jpg)

  ### Routerとレジストリに関する設定を行います。  
  
  1. Routerに対するノードセレクターはオプションです。指定を行うとマッチしたホストが存在する場合のみRouterが作成されます。  
  2. Routerの数を指定します。指定されない場合は、セレクターで指定されたホストの数となります。  
  3. Router同様、レジストリに対するノードセレクターはオプションです。指定を行うとマッチしたホストが存在する場合にのみ作成されます。  
  4. Router同様、レジストリの数を指定します。指定されない場合は、セレクターで指定されたホストの数となります。  
  5. レジストリをサポートするストレージ関連の設定です。  
  
  ![project-Deploy1](./1-8n.jpg)

  ### サービスカタログに関する設定を行います。  

  1. サービスカタログを有効化します。  
  2. エラー回避のため指定しているオプションです。  
  3. テンプレートサービスブローカーを有効化します。（1の有効化が必須）  
  4. テンプレートサービスブローカーのホストの指定です。  
  5. バグに関するワークアラウンドです。  
  6. テンプレートサービスブローカーによってサービスされる1つ以上の名前空間です。  


  ![project-Deploy1](./1-9n.jpg)
  
  
  ### 以下は、Ansible Hosts ファイルで、OpenShiftを構成するホストに関する設定です。  
  
  ![project-Deploy1](./1-10n.jpg)
  
 
 ### OpenShiftインストーラーの起動  
 
 以下のコマンドで、OpenShiftインストーラーを起動します。
 　※インストール完了までに20分ほどかかかります。適当に休憩を取りましょう♪
 
    # ansible-playbook -f 20 -i /root/my_ocp_inventory /usr/share/ansible/openshift-ansible/playbooks/byo/config.yml
 
 ### OpenShiftアドミン権限の設定  
 
 既に設定されているユーザー、andrewとkarlaの内、andrewにクラスターアドミの権限を与えてみます。

    # ssh master1.$GUID.internal
    # oc login -u system:admin 
    # oadm policy add-cluster-role-to-user cluster-admin andrew
    
    
## 1-1.プロジェクトの作成
 OpenShiftは、”プロジェクト”　単位でアプリケーションや権限などを管理しています。  
 アプリケーションを作成するにはまずプロジェクトを作成します。  
   
### ***1-1-1. OpenShift のWebコンソールにログインし、"New Project" をクリックします。***  

アドレスは、ロードバランサーの外部IPアドレスです。  
https://loadbalancer.$GUID.example.opentlc.com  
 user:karla  
 password:r3dh4t1!  
 
   <img src="2-1-1n.jpg" alt="attach:cat" title="attach:cat" width="700">  


### ***1-1-2. Name欄に、"nexus-your-name"を入力し、Createをクリックします。***  
 ※プロジェクト名は一意ある必要がありますのでご注意ください。
<img src="2-1-2.jpg" alt="attach:cat" title="attach:cat" width="700">  
   
  
## 1-2.コンテナのデプロイメント
プロジェクト内にアプリケーションを作成してみます。  

### ***1-2-1. Deploy Imageをクリック。***
![project-Deploy1](./2-2-1-2n.jpg)
### ***Image Name 欄に、”sonatype/nexus3”を入力の上、右端の*** 🔍 ***アイコンをクリックし、次に、Deployをクリックします。***
![project-Deploy1](./2-2-1-2n2.jpg)
※Sonatype が提供するDockerレポジトリからのコンテナ取得を行っています。また、詳細パラメータでは、コンテナに独自の構成情報を付加する「環境変数」と、リソースをグループ化し、アクセス制限など細かな制御を実現する「ラベル」の設定が可能であることを確認します（今回は設定しません）。
  　
  　
  　
### ***1-2-2. プロジェクト名をクリックします。***
　![project-Deploy1](./2-2-3n.jpg)


### ***1-2-3. 上下の矢印をクリックし、Podが増減することを確認します。***
「Project」の「Overview」ページが開きます。「Nexus3」というサービスが作成され、Podが1つデプロイされていることが分かります。  
  
　![project-Deploy1](./2-2-4n.jpg)
  

### ***1-2-4. Create Routeをクリック(上図参照)し、「Nexus3」サービスに対するルートを作成します。***

 　![project-Deploy1](./2-2-5.jpg)
※コンテナをデプロイするだけでは外からの接続が行えません。
　ルート作成では、以下のようなオプションが設定可能です。（今回は指定しません）

HostName：アプリケーションのDNS名を指定（指定しない場合は自動で作成）  
Path：サービスへの追加URL（サブディレクトリ）  
Target Port：ネットワークポートの指定  
Security：  TLSターミネーションタイプの変更
  
### ***1-2-5. サービスへのルートが作成されました。クリックしてみます。***
  
 　![project-Deploy1](./2-2-6n.jpg)

Nexusリポジトリマネージャーが起動し、接続できることを確認します。サインインに関しては以下のアカウントが利用可能です。

admin/admin123

※サービスが完全に立ち上がるまでに少し時間がかかります。もし接続できないときは、少し時間をおいて確認してみましょう。


![project-Deploy1](./2-2-7.jpg)

### ***1-2-6. 動作が確認できたら、、ホームページに戻り、プロジェクトを削除します。***

![project-Deploy1](./2-2-8n.jpg)

## 2.使用リソースの確認
　新しくアプリケーションをデプロイし、そのアプリケーションが使用しているCPUやメモリリソースについて確認してみましょう。

### 2-1.テンプレートの利用とリソースの確認
　### ***2-1-1. まずはプロジェクトを作成します。***

![project-Deploy1](./2-1-1n.jpg)
### ***2-1-2. 名前を入力し、「Create」をクリックします。***
※名前はご自由に入力ください。但し、OpenShif内で一意である必要があります。

![project-Deploy1](./3-1-2n.jpg)

### ***2-1-3. 今度はテンプレートからデプロイしてみます。検索画面で、"cakephp" を入力。CakePHP +MySQL　の「Ephemeral」を選択します。***

![project-Deploy1](./3-2-1n.jpg)

### ***2-1-4. 一番上のAdd to Projectで、作成したプロジェクトを選択し、さらに続く設定で以下が可能なことを確認の上、一番下の「Create」をクリックし、デプロイを開始します。***

サービス名、PHPとMySQLコンテナがそれぞれ利用可能な、最大メモリの値、アプリのソースコードが含まれるGitリポジトリが指定できることを確認します。

![project-Deploy1](./3-2-2n.jpg)

さらに、データベースのユーザーやパスワード等の設定やラベルをここで定義できることを確認します。指定しない場合は、自動的に作成されます。これらの値を利用して、フロントエンドのアプリケーションとバックエンドのデータベースのデプロイを行います。

プロジェクトをクリックし（図なし）、Overview で、以下を確認します。  
- Podが二つデプロイされていること  
- ルートが既に作成されていること  
- Build状態を確認するためのリンク  
- サービスへのリンク  
※作成には少し時間がかかります。Pod作成の完了は、Pod数 1 になっていることを確認しましょう。  
　さらに、レプリカの数の増減が出来ること、リソースの消費量が確認できます。

![project-Deploy1](./3-2-5n2.jpg)

※リソースが表示されない場合は以下を確認しブラウザのアクセス制限を解除下さい。

「Applications」→「Pods」を選択。
「cakephp-xxxx...」をクリックし、「Metrics」をクリック
ブラウザのセキュリティを解除してください

![project-Deploy1](./3-2-6-1.jpg) ![project-Deploy1](./3-2-6-2.jpg)
![project-Deploy1](./3-2-6-3.jpg)
![project-Deploy1](./3-2-6-4.jpg) ![project-Deploy1](./3-2-6-5.jpg)  

「Metrics」がそもそも表示されないケースもあるようです。その際は、まず、以下にアクセスし、
MetricsServiceがStartになっていることを確認の上、master 1/2/3をリブートしてみてください。

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

![project-Deploy1](./3-3-4-2.jpg)

右上の「Start Build」をクリックしてみましょう。
#2が追加されることが分かります。#1、#2をクリックして内容を確認してみます。

![project-Deploy1](./3-3-5-1.jpg)
![project-Deploy1](./3-3-5-2.jpg)

こちらは#1の表示例です。以下を確認します  
・「Rebuild」のクリックで再ビルド実行可能  
・ビルドのステータスとトリガー理由  
・ソース情報  
　など  
「Logs」をクリックします。

![project-Deploy1](./3-3-6.jpg)

ソースコードリポジトリがコンテナ用にダウンロードされていることが分かります。一番下の方を確認すると、完了したイメージがプロジェクト名を追加されてレジストリに追加されていることが分かります。

![project-Deploy1](./3-3-7.jpg)

「アプリケーション」→「Deployments」で何が見えるか確認してみます。

![project-Deploy1](./3-3-8-2n.jpg)

cakephp-mysql-exampleの方は、先ほどデプロイしましたので、LastVersionが＃2となっています。また以下の情報が取得できることを確認します。  
  
・それぞれのPodのレプリカ数  
・いつ作成されたか  
・作成されたきっかけ  
  
![project-Deploy1](./3-3-9n.jpg)

一通り確認ができたら、cakephp-mysql-example　行の「#2」をクリックします。テンプレートで設定された内容が確認できます。また、右上の「Actions」をクリックし、以下の確認・設定等が行えることを確認します。  

・オートスケール  
・リソースの制限  
・永続ボリュームの追加　など  

![project-Deploy1](./3-3-10.jpg)

「Environment」タブをクリックします。
デプロイ時に設定された環境変数が確認できます。環境変数はPod内で、ユーザー名やデータベース名など様々なパラメーターを設定することが可能で、また、ユーザー自身での追加も可能です。  
　※追加の場合は、ページ上部にある、以下のリンクをクリックし編集モードにします。  
　Environment variables can be edited on deployment config cakephp-mysql-example  
確認が終了したら、「Events」タブをクリックします。  

![project-Deploy1](./3-3-11.jpg)

デプロイに関する様々なイベントを確認できます。一連の作業内容の確認や、デプロイの際のトラブルの確認などに利用可能です。

![project-Deploy1](./3-3-12.jpg)

### 2-3.Podの確認
次にPodの確認を行ってみましょう。
「Applications」→「Pods」をクリックします。

![project-Deploy1](./3-4-1.jpg)


プロジェクト内で有効なPodが表示されることを確認します。「Completed」、「Running」などのステータスが確認できます。Podの一つをクリックします。

![project-Deploy1](./3-4-2n.jpg)

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

![project-Deploy1](./3-5-1-2.jpg)
「Applications」→「Routes」をクリックし、ルートが表示されることを確認します。  
![project-Deploy1](./3-5-2-2.jpg)

## 3.クォータの設定と確認 (03 02)
一つのプロジェクトがリソースを食いつぶしてしまわないよう、プロジェクトにはクォータの設定が可能です。プロジェクトの管理者・利用者はあらかじめ定められたリソース内でリソースを利用することが可能です。  

![project-Deploy1](./4-1-1.jpg)

まだクォータが設定されていないので、何も表示されません。このプロジェクトに対し、クォータを設定してみます。  

![project-Deploy1](./4-1-1-2n.jpg)

クォータの設定はコマンドラインから行います。sshクライアントで、Workstation 環境に接続します。  
　※接続先等は別途ご確認ください。  

    $ sudo -i
    # oc login -u andrew https://loadbalancer1.${GUID}.example.opentlc.com/
    　login ID:andrew
      Password:r3dh4t1! 
    # cat << EOF > /root/compute-resources.yaml
    apiVersion: v1
    kind: ResourceQuota
    metadata:
      name: compute-resources
    spec:
      hard:
        pods: 5
        limits.cpu: 2
        limits.memory: 8Gi
        services: 5
    EOF
    
    # cat << EOF > /root/core-resource-limits.yaml
    apiVersion: v1
    kind: "LimitRange"
    metadata:
      name: "core-resource-limits"
    spec:
      limits:
        - type: "Pod"
          max:
            cpu: "2"
            memory: "1Gi"
          min:
            cpu: "200m"
            memory: "6Mi"
        - type: "Container"
          max:
            cpu: "2"
            memory: "1Gi"
          min:
            cpu: "100m"
            memory: "4Mi"
          default:
            cpu: "300m"
            memory: "200Mi"
          defaultRequest:
            cpu: "200m"
            memory: "100Mi"
          maxLimitRequestRatio:
            cpu: "10"
    EOF

    # oc create -f compute-resources.yaml -n <Project Name>
    # oc create -f core-resource-limits.yaml -n <Project Name>
      ※最後のパラメータは自身のプロジェクト名に変更ください。
    # oc project <Project Name>
    # oc describe quota
    Name:           compute-resources
    Namespace:      new-apps
    Resource        Used    Hard
    --------        ----    ----
    limits.cpu      300m    2
    limits.memory   1Gi  8Gi
    pods            2       5
    services        2       5

再度、OpenShift Master GUIで、「Resources」→「Quota」を確認してみてみると、以下のようなリソース利用状況が確認できます。  

![project-Deploy1](./4-1-1-3-3n.jpg)

・このプロジェクトで利用しているCPU/メモリ/Pod数/サービス数  
・このプロジェクトに設定されているクォータ  

![project-Deploy1](./4-1-2.jpg)

クォータの効果を確認するため、「Overview」をクリックし、Podを5に増やしてみましょう。  
![project-Deploy1](./4-1-3n.jpg)

上記でQutaをクリックし、リソースの超過を確認してみます。  
![project-Deploy1](./4-1-4n.jpg)

終了したら、、ホームページに戻り、プロジェクトを削除します。  


## 4.環境変数の設定と確認 (03 04)
アプリケーション作成の際に環境変数を設定し、コンテナに設定が反映されることを確認してみましょう。
　新しくアプリケーションをデプロイし、そのアプリケーションが使用しているCPUやメモリリソースについて確認してみましょう。

### 4-1.環境変数の設定と動作の確認
　まずはプロジェクトを作成します。  
  ※名前はご自由に入力ください。但し、OpenShif内で一意である必要があります。

![project-Deploy1](./6-1-1n.jpg)

### ***カタログ検索で、"Node"と入力し、Node.js + MongoDB (Ephemeral)を選択します。***

![project-Deploy1](./6-1-3n.jpg)

### ***Add to Projectで、作成したプロジェクトを選択します。***

![project-Deploy1](./6-1-5n.jpg)

### ***下方へスクロールし、環境変数として「Administrator Password」欄に適当なパスワードを入力してみましょう。入力がすんだら、「Create」をクリックします。***

![project-Deploy1](./6-1-6n.jpg)
### ***「Applications」→「Pods」をクリックします。***

![project-Deploy1](./6-1-7.jpg)
### ***nodejs-mongodb-example-xxxをクリックします。***

![project-Deploy1](./6-1-8.jpg)
### ***Terminalタブを開き、echo $MONGODB_ADMIN_PASSWORDを入力し、デプロイ時に設定したパスワードが表示されることを確認します。***
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

確認が終了したら、ホームページに戻りプロジェクトを削除します。  


## 5.環境変数によるアプリケーションの動作と再デプロイ (04 01)
異なるアプリケーションの動作を環境変数によって実現してみましょう。また、構成変更した後にアプリケーションの再デプロイを行い、その動きについて確認してみます。

### 5-1.環境変数の設定と動作の確認
　まずはプロジェクトを作成します。  
  ※名前はご自由に入力ください。但し、OpenShif内で一意である必要があります。

![project-Deploy1](./6-1-1n.jpg)

### ***カタログ検索で、"php"と入力し、PHPを選択します。***

![project-Deploy1](./7-1-3n.jpg)

### ***Add to Projectで、作成したプロジェクト、Versionは7.0を選択します。 Name欄に、"cotd"、GitURLに、https://github.com/devops-with-openshift/cotd を入力。さらに、「advanced options」をクリックし、下の方へスクロールします。***

![project-Deploy1](./7-1-4n.jpg)


### ***Deployment Configurationsセクションで、環境変数として、"SELECTOR cities" を入力します。さらに下にスクロールし、Labelsに "app cotd" が設定されていることを確認し（図なし）、「Create」をクリックします。***

![project-Deploy1](./7-1-6n.jpg)

### ***「Overview」で、アプリケーションのルートをクリックします。***  
アプリケーションがデプロイされていること設定した環境変数により、「街」の写真が表示されることを確認します。  

![project-Deploy1](./7-1-8n.jpg)  


### ***さらに、Podアイコン右側の上矢印を2度クリックし、3Podにスケールアップします。***

![project-Deploy1](./7-1-9n.jpg)  

### 5-2.アプリケーションの再デプロイ
作成したアプリケーションを再デプロイしてみます。

### ***「Applications」→「Deployments」をクリックします。***  
![project-Deploy1](./7-2-1.jpg)

### ***「cotd」をクリックします。*** 
![project-Deploy1](./7-2-2-0n.jpg)

### ***「Deploy」をクリックしすぐに「Overview」をクリックします。***
新しいPodが必要なレプリカ数（今回の場合3個）になるまで増え、逆に、旧Pod新Podが作成される度に1つずつ削除されることを確認します。
Podのこの様なアップデート方法を、ローリングアップデートと呼び、アップデートのデフォルト値です。後ほど、別のポリシーに変更してみます。

![project-Deploy1](./7-2-2-1.jpg)
![project-Deploy1](./7-2-2-2.jpg)
  
![project-Deploy1](./7-2-3.jpg)

### ***「Overview」から、アプリケーションへのショートカット、「cotd」をクリックします。***

![project-Deploy1](./7-2-4-2n.jpg)

### ***「Environment」タブで、環境変数の値を、”cats”に変更し、Saveし、すぐに、「Overview」に戻り、Podが再デプロイされていることを確認します。***  
![project-Deploy1](./7-2-4-3n.jpg)

### ***さらに、アプリケーションへのリンクをクリックし、先ほどの「街」ではなく、「ネコ」が表示されることを確認します。***

![project-Deploy1](./7-2-5.jpg)
### ***再度アプリケーションへのショートカット「cotd」をクリックします。***

![project-Deploy1](./7-2-4-2n.jpg)

### ***「Configuration」タブを表示させ、StrategyがRollingであることを確認。その後、「Actions」から、「Edit YAML」をクリックします。
※オートスケーラーやストレージの追加、リソース制限等もこちらから設定可能であることもついでに確認しておきましょう！***

![project-Deploy1](./7-2-6.jpg)

### ***上記の例に従って、Strategy の中を編集し、保存します。***
![project-Deploy1](./7-2-7n2.jpg)

### ***StrategyがRecreateに変更されたことを確認します。***
![project-Deploy1](./7-2-7-2n.jpg)

### ***「Deploy」をクリックし、「Overview」を表示します。***
![project-Deploy1](./7-2-8.jpg)

（確認のみ）Recreateに変更したことにより、レプリカのすべてが0にスケールダウンされた後、新しいPodがデプロイされることを確認します。表示上の時間が短く分かりにくい点もありますが、上記のように推移します。

![project-Deploy1](./7-2-9.jpg)

## 6.アプリケーションのライフサイクル管理
Jenkinsを使ったアプリケーションライフサイクルの管理を行います。  
このラボは、コマンドラインを多用しますので、SSHで接続できることをまず確認してみます。  
OPENTLC アカウントを利用し、sshで接続します。  

##### ***```$ ssh -i ~/.ssh/yourprivatekey.key opentlc-user@bastion.<guid>.example.opentlc.com```***  
続いて、OpenShift環境へログインです。user名 "karla" でOpenShiftマスターにログインします。  
パスワードは、r3dh4t1!です。  
##### ***```$ oc login -u karla https://loadbalancer1.<guid>.example.opentlc.com/```***

### 6-1.プロジェクトの作成とJenkinsのデプロイ
３つの新しいプロジェクト(dev/test/ prod)を作成します。コマンドを4個実行します。 随時、OpenShiftのGUIでも作成の様子を確認してみましょう。

    $ oc new-project pipeline-dev --description="Cat of the Day Development Environment" --display-name="Cat Of The Day - Dev"
    $ oc new-project pipeline-test --description="Cat of the Day Testing Environment" --display-name="Cat Of The Day - Test"
    $ oc new-project pipeline-prod --description="Cat of the Day Production Environment" --display-name="Cat Of The Day - Prod"
    
    
作成したprojectを確認します。

    $ oc get projects
    NAME                   　　DISPLAY NAME         STATUS
    pipeline-mydemo-dev　　　Cat Of The Day - Dev　　　　Active
    pipeline-mydemo-prod　　Cat Of The Day - Prod　　　　Active
    pipeline-mydemo-test　　　Cat Of The Day - Test　　　　Active

作成した3つのうち、pipline-devに入ります。

    $ oc project pipeline-dev
ビルドやデプロイメント管理のため、**Jenkins** をデプロイします。
デプロイが成功することを確認します。

    $ oc new-app jenkins-persistent -p ENABLE_OAUTH=false -p MEMORY_LIMIT=1.5Gi -n pipeline-dev
    （コマンド実行後の表示内容は以下の通りです）
    --> Deploying template "openshift/jenkins-persistent" to project pipeline-dev
     Jenkins (Persistent)
     （途中略）
     --> Success
    Run 'oc status' to view your app.

GUI でPod が立ち上がった（Podの青い丸が表示されている）ことを確認した上で、**Jenkins**にログインしてみます。
 admin / password です。
接続先は、Podのルートで確認です！
![project-Deploy1](./8-1-7.jpg)

Jenkins サービスアカウントに、test, prodプロジェクトに対するリソース管理権限を与えます。

    $ oc policy add-role-to-user edit system:serviceaccount:pipeline-dev:jenkins -n pipeline-test
    $ oc policy add-role-to-user edit system:serviceaccount:pipeline-dev:jenkins -n pipeline-prod

devから、test、prodへのイメージの引用を許可します。

    $ oc policy add-role-to-group system:image-puller system:serviceaccounts:pipeline-test -n pipeline-dev
    $ oc policy add-role-to-group system:image-puller system:serviceaccounts:pipeline-prod -n pipeline-dev

devプロジェクトにモックアプリケーションを作成します。

    $ oc new-app https://github.com/devops-with-openshift/cotd -n pipeline-dev
    （作成完了の確認は以下）
    $ oc logs -f build/cotd-1 -n pipeline-dev 

イメージにtestready とprodreadyのTAGを付けます

    $ oc tag cotd:latest cotd:testready -n pipeline-dev
    $ oc tag cotd:testready cotd:prodready -n pipeline-dev

イメージストリームの内容及び、testreadyと、prodreadyのTAGが追加されていることを確認します。

    $　oc describe is cotd -n pipeline-dev
    
    oc describe is cotd -n pipeline-dev
    
    Name:			cotd
    Created:		About an hour ago
    Labels:			app=cotd
    Annotations:		openshift.io/generated-by=OpenShiftNewApp
    Docker Pull Spec:	172.30.99.85:5000/pipeline-mydemo-dev/cotd
    
    Tag		Spec				Created			PullSpec							Image
    latest		<pushed>			About an hour ago	172.30.99.85:5000/pipeline-mydemo-dev/cotd@sha256:21c16f04309942...	<same>
    prodready	cotd@sha256:21c16f04309942...	About an hour ago	172.30.99.85:5000/pipeline-mydemo-dev/cotd@sha256:21c16f04309942...	<same>
    testready	cotd@sha256:21c16f04309942...	About an hour ago	172.30.99.85:5000/pipeline-mydemo-dev/cotd@sha256:21c16f04309942...	<same>

test, prodプロジェクトにそれぞれ、testreadyのTAG、ProdreadyのTAGがついたイメージからアプリケーションをデプロイします。

    $ oc new-app pipeline-dev/cotd:testready --name=cotd -n pipeline-test
    $ oc new-app pipeline-dev/cotd:prodready --name=cotd -n pipeline-prod

3つ全てのアプリケーションのルートを作成します。

    $ oc expose service cotd -n pipeline-dev
    $ oc expose service cotd -n pipeline-test
    $ oc expose service cotd -n pipeline-prod

自動デプロイメントを無効化します。

    $ oc get dc cotd -o yaml -n pipeline-dev | sed 's/automatic: true/automatic: false/g' | oc replace -f -
    $ oc get dc cotd -o yaml -n pipeline-test| sed 's/automatic: true/automatic: false/g' | oc replace -f -
    $ oc get dc cotd -o yaml -n pipeline-prod | sed 's/automatic: true/automatic: false/g' | oc replace -f -

OpenShift WebUIにログインし、**devプロジェクト**を表示、**「Add to Project」**で、**import YAML / JSON**を表示します。
![project-Deploy1](./8-1-15.jpg)

以下のテキストをコピーペーストします。**「Create」**をクリックし、ビルドコンフィグパイプラインを作成します。

    apiVersion: v1
    items:
    - kind: "BuildConfig"
      apiVersion: "v1"
      metadata:
       name: "pipeline-demo"
      spec:
        triggers:
              - github:
                  secret: 5Mlic4Le
                type: GitHub
              - generic:
                  secret: FiArdDBH
                type: Generic
        strategy:
          type: "JenkinsPipeline"
          jenkinsPipelineStrategy:
            jenkinsfile: |
                                node {
                                    stage ("Build") {
                                          echo '*** Build Starting ***'
                                          openshiftBuild bldCfg: 'cotd', buildName: '', checkForTriggeredDeployments: 'false', commitID: '', namespace: '', showBuildLogs: 'true', verbose: 'true'
                                          openshiftVerifyBuild bldCfg: 'cotd', checkForTriggeredDeployments: 'false', namespace: '', verbose: 'false'
                                          echo '*** Build Complete ***'
                                    }
                                    stage ("Deploy and Verify in Development Env") {
                                          echo '*** Deployment Starting ***'
                                          openshiftDeploy depCfg: 'cotd', namespace: '', verbose: 'false', waitTime: ''
                                          openshiftVerifyDeployment authToken: '', depCfg: 'cotd', namespace: '', replicaCount: '1', verbose: 'false', verifyReplicaCount: 'false', waitTime: ''
                                          echo '*** Deployment Complete ***'
                                     }
                                }

    kind: List
    metadata: {}

### 6-2.パイプラインのテスト
**Jenkins**によるパイプラインの管理を行ってみます。

![project-Deploy1](./8-2-1.jpg)  
**Builds → Pipelines**で、パイプラインが作成されていることを確認します。

![project-Deploy1](./8-2-2.jpg)
**Start Pipeline**をクリックし、パイプラインが作成されるのを待ちます。作成後、ビルド、デプロイにかかった時間を確認します。

![project-Deploy1](./8-2-3.jpg)
**Edit Pipeline**をクリックします。

![project-Deploy1](./8-2-4.jpg)
（確認のみ）以下を確認します。  
　・パイプラインがビルドとデプロイ＆ベリファイの二つのステージを持っている  
　・それぞれのステージは定義可能な複数の動作を持っている  
　・ビルドステージでは、ビルドが実行され、正常に完了させる  
　・デプロイステージでは、新たなデプロイが実行され、コンテナが正常に作成される  
　・シリアル実行ポリシーが設定されている  

### 6-3.継続インテグレーションの確認  
環境は以下の通りです。 
- アプリケーションライフサイクルを意図した3つの段階がある（Dev、Test、Prod）  
- 開発プロジェクトにはJenkinsと開発ステージのコンテナがある  
- Testプロジェクトはコンテナのテストと、限られた環境内での他のアプリケーションやデーターソースとの統合テストを行う  
- Prodプロジェクトには、プロダクションノード上で動作するコンテナがあり、アプリケーションのライブバージョンが稼働している  
  
 この環境からパイプラインの編集を行い、継続インテグレーションの確認を行います。
  
![project-Deploy1](./8-3-2.jpg)
パイプラインを編集します。2行目のGUID=xxxを自身の値に変更した上で、下記テキストをコピーペーストします。  

    node {
       withEnv(['GUID=mydemo']) {

        stage ("Build") {
          echo '*** Build Starting ***'
          openshiftBuild bldCfg: 'cotd', buildName: '', checkForTriggeredDeployments: 'false', commitID: '', namespace: '', showBuildLogs: 'false', verbose: 'false', waitTime: ''
          openshiftVerifyBuild apiURL: 'https://openshift.default.svc.cluster.local', authToken: '', bldCfg: 'cotd', checkForTriggeredDeployments: 'false', namespace: '', verbose: 'false'
          echo '*** Build Complete ***'
        }

        stage ("Deploy and Verify in Development Env") {
          echo '*** Deployment Starting ***'
          openshiftDeploy apiURL: 'https://openshift.default.svc.cluster.local', authToken: '', depCfg: 'cotd', namespace: '', verbose: 'false', waitTime: ''
          openshiftVerifyDeployment apiURL: 'https://openshift.default.svc.cluster.local', authToken: '', depCfg: 'cotd', namespace: '', replicaCount: '1', verbose: 'false', verifyReplicaCount: 'false', waitTime: ''
          echo '*** Deployment Complete ***'

          echo '*** Service Verification Starting ***'
          openshiftVerifyService apiURL: 'https://openshift.default.svc.cluster.local', authToken: '', namespace: 'pipeline-${GUID}-dev', svcName: 'cotd', verbose: 'false'
          echo '*** Service Verification Complete ***'
          openshiftTag(srcStream: 'cotd', srcTag: 'latest', destStream: 'cotd', destTag: 'testready')
        }

        stage ('Deploy and Test in Testing Env') {
          echo "*** Deploy testready build in pipeline-${GUID}-test project  ***"
          openshiftDeploy apiURL: 'https://openshift.default.svc.cluster.local', authToken: '', depCfg: 'cotd', namespace: 'pipeline-${GUID}-test', verbose: 'false', waitTime: ''

          openshiftVerifyDeployment apiURL: 'https://openshift.default.svc.cluster.local', authToken: '', depCfg: 'cotd', namespace: 'pipeline-${GUID}-test', replicaCount: '1', verbose: 'false', verifyReplicaCount: 'false'
          sleep 30
          sh 'curl http://cotd-pipeline-${GUID}-test.apps.na1.openshift.opentlc.com/data/ | grep cats -q'
        }

        stage ('Promote and Verify in Production Env') {
          echo '*** Waiting for Input ***'
          input 'Should we deploy to Production?'
          openshiftTag(srcStream: 'cotd', srcTag: 'testready', destStream: 'cotd', destTag: 'prodready')
          echo '*** Deploying to Production ***'
          openshiftDeploy apiURL: 'https://openshift.default.svc.cluster.local', authToken: '', depCfg: 'cotd', namespace: 'pipeline-${GUID}-prod', verbose: 'false', waitTime: ''
          openshiftVerifyDeployment apiURL: 'https://openshift.default.svc.cluster.local', authToken: '', depCfg: 'cotd', namespace: 'pipeline-${GUID}-prod', replicaCount: '1', verbose: 'false', verifyReplicaCount: 'false'
          sleep 60
          sh 'curl http://cotd-pipeline-${GUID}-prod.apps.na1.openshift.opentlc.com/data/ | grep cats -q'
        }
      }
    }
    
     
 この内容は以下の通りです。  
- 4つのステージを含む  
- 定義可能な複数の動作を有する  
- シリアル実行ポリシーである  
- ビルド  
　　ビルドを実行し、成功を確認する  
- Devでのデプロイと確認  
　開発プロジェクトで最新のアプリケーションをデプロイし、コンテナとサービスが正しくデプロイされていることを確認、イメージを「testready」としてタグ付けします。  
- Testでのデプロイとテスト  
　「testready」タグの付いたイメージをデプロイし、統合テスト(この場合はcURLコマンド)を実行し、成功すると「prodready」としてタグ付けされる  
- プロダクション環境でのプロモーション  
　承認プロセスを経て、Prodでコンテナサービスを展開します。  
 
 ![project-Deploy1](./8-3-3.jpg)
 **「View Log」**をクリックします。
   
 ![project-Deploy1](./8-3-4.jpg)
  Jenkinsが立ち上がりました。
  **admin/password を入力し**ログインします。
ログを確認したら、**Back to Project** をクリックします。

    
 ![project-Deploy1](./8-3-5.jpg)
  以下を確認します。  
- Jenkinsのインターフェースでも、パイプラインの監視、構成、管理が可能。
- 「Build Now」ボタンで新しいパイプラインをスタートさせることが可能。
- 異なるステージの確認が可能。
- 「Pipeline Syntax」をクリックすることにより簡単に新しいパイプラインの作成が可能

 OpenShiftのWebコンソールに戻ります。
**「Input Required」をクリック→Jenkinsの「Proceed」をクリック**し、Prodにアプリケーションをデプロイします！
prod プロジェクトで、cotd が再デプロイされていることを確認します！

## 以上で終了です、お疲れさまでした！！ 
