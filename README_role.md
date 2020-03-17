# Ansible Role: IIS\_Setup
===============================

## Description

本Ansible RoleはWebサーバソフトウェアである"IIS"の設定を行います。
対象バージョンは以下のバージョンです。

- Windows Server 2016 Internet Information Server10

このRoleの実行前提はIISが既にインストールされたことです。  
なお、Ansibleのモジュールを利用して、以下の設定を行います。  

- WebAppPool  
- WebSite

## Supports

- 管理マシン(Ansibleサーバ)
  - Linux系OS（RHEL/CentOS）
  - Ansible バージョン2.6 (動作確認済：2.6.1)
  - Python バージョン2.6 または 2.7

- 管理対象マシン(インストール対象マシン)
  - Windows Server 2016
  - PowerShell 4.0

## Requirements
- 管理マシン(Ansibleサーバ)
  - 管理対象マシンへ	WinRM接続できること
  - WinRM接続設定は次のサイトを参考にすること  
    ・https://docs.ansible.com/ansible/latest/user_guide/windows_setup.html#winrm-setup
- 管理対象マシン(インストール対象マシン)
  - PowerShell 4.0を利用できること
  - ファイアフォールが適切に設定されていること

## Role Variables
### Mandatory Variables

以下の変数は必ず指定します。

- WinRM設定（group\_varsもしくはhost\_varsに設定）
  * ''ansible\_user'': ユーザ名(string)
  * ''ansible\_password'': パスワード(string)
  * ''ansible\_port'': ポート(number)
  * ''ansible\_connection'': 接続方式(string)
  * ''ansible\_winrm\_server\_cert\_validation'': 暗号化有無(string)

- 共通変数
  * ''VAR\_IIS\_OS\_Version'': 管理対象マシンのOSバージョンを設定します (string, 例："Windows Server 2016")

- IIS設定の関連変数
  * ''VAR\_WEBAPPPOOL\_NAME'': WebAppPool名を設定します (string, 例："myWebAppPool")
  * ''VAR\_WEBSITE\_NAME'': Webサイト名を設定します (string, 例："myWebSite1")

### Optional variables

以下の変数は任意で指定します。

- IIS設定(WebAppPool)の関連変数
  * ''VAR\_WEBAPPPOOL\_ATTRI'': アプリケーションプールの基本設定を指定します (詳細情報は子項目に設定します)
	  * ''autoStart'': 自動起動フラグ (boolean, 例：true)
	  * ''CLRConfigFile'': CLRの設定ファイル (string, 例："C:\\\\Temp\\\\net.config")
	  * ''enable32BitAppOnWin64'': 32ビットアプリケーションの有効化 (boolean, 例：false)
	  * ''enableConfigurationOverride'': 構成オーバーライドの有効化 (boolean, 例：true)
	  * ''managedPipelineMode'': マネージパイプラインモード（設定可能の値：Classic、Integrated） (string, 例："Integrated")
	  * ''managedRuntimeLoader'': マネージラインタイムローダー (string, 例："webengine4.dll")
	  * ''managedRuntimeVersion'': マネージラインタイムバージョン（設定可能の値：v1.1、v2.0、v4.0） (string, 例："v4.0")
	  * ''passAnonymousToken'': 匿名トークン (boolean, 例：true)
	  * ''queueLength'': キューの長さ (number, 例：1000)
	  * ''startMode'': 開始モード（設定可能の値：AlwaysRunning、OnDemand） (string, 例："OnDemand")
	  * ''cpu.xxx'': cpuに関する設定項目 (子項目を持っています, 例："cpu.numaNodeAffinityMode: Hard")
	  * ''Failure.xxx'': Failureに関する設定項目 (子項目を持っています, 例："Failure.loadBalancerCapabilities: TcpLevel")
	  * ''processModel.xxx'': processModelに関する設定項目 (子項目を持っています, 例："processModel.identityType: LocalService")
	  * ''recycling.xxx'': recyclingに関する設定項目 (子項目を持っています, 例："recycling.periodicRestart.time: 29:00:00")

- IIS設定(WebSite)の関連変数
  * ''VAR\_WEBSITE\_INFO'': Webサイトの関連情報を設定します (詳細情報は子項目に設定します)
	  * ''hostname'': ホスト名 (string, 例："www.example.com")
	  * ''ip'': IPアドレス (string, 例："192.168.0.1")
	  * ''parameters'': パラメータ、空に設定不可 (string, 例："logfile.directory:C:\\\\sites\\\\logs")
	  * ''physical\_path'': 物理パス (string, 例："C:\\\\sites\\\\acme")
	  * ''port'': ポート番号 (string, 例："8443")
  * ''VAR\_WEBSITE\_State'': Webサイトが起動/停止を指定します（START：起動（デフォルト）；STOP：停止）  
※既に存在するWebサイトの起動/停止のみを実施したい場合、VAR\_WEBAPPPOOL\_ATTRIとVAR\_WEBSITE\_INFOを指定する必要なし  
※Webサイトを作成する時、必ずVAR\_WEBSITE\_INFOを設定する必要である。  

- IIS設定(Webbinding)の関連変数
  * ''VAR\_WEBBINDING\_INFO'': Webbindingに関する設定を指定します (詳細情報は子項目に設定します。複数グループを設定できます。)
	  * ''host\_header'': ホスト名 (string, 例："www.example.com")
	  * ''certificate\_hash'': 証明書ハッシュ (string, 例："xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx")
	  * ''certificate\_store\_name'': 証明書ストア名 (string, 例："my")
	  * ''ip'': BindingするIPアドレス (string, 例："192.168.0.1")
	  * ''port'': Bindingするポート番号 (string, 例："8443")
	  * ''protocol'': プロトコル種類 (string, 例："https")
	  * ''ssl\_flags'': SSLフラグ（サーバ名表示を要求する（1）/しない（0））、この項目を設定する時、host\_headerとprotocolも設定する必要 (number, 例：0)

- IIS設定(WebApplication)の関連変数
  * ''VAR\_WEBAPP\_INFO'': WebApplicationに関する設定を指定します (詳細情報は子項目に設定します。複数グループを設定できます。)
	  * ''name'': WebApplication名 (string, 例："myApplication")
	  * ''physical\_path'': 物理パス (string, 例："C:\\\\sites\\\\")

- IIS設定(Virtualdirectory)の関連変数
  * ''VAR\_VIRTUALDIR\_INFO'': Virtualdirectoryに関する設定を指定します (詳細情報は子項目に設定します。複数グループを設定できます。)
	  * ''name'': Virtualdirectory名 (string, 例："myVirtualdirectory")
	  * ''application'': アプリケーション (string, 例："myApplication")
	  * ''physical\_path'': 物理パス (string, 例："C:\\\\sites\\\\")


## Dependencies

特にありません。

## Usage

1. 本Roleを用いたPlaybookを作成します。
2. 必須変数を指定します。  
   注：このRoleの一回呼び出しで一つのWebAppPoolと対応する一つのWebサイトのみ指定してください。
3. 任意変数を必要に応じて指定します。
4. Playbookを実行します。


## Example Playbook

インストールとすべての設定をする場合は、提供した以下のRoleを"roles"ディレクトリ配置したうえで、
以下のようなPlaybookを作成してください。

- フォルダ構成
~~~
  - group_vars/
    ・ server1
    ・ server2
  - host_vars/
    ・ host1
    ・ host2
  - roles/
    ・ IIS_Install/
    ・ IIS_Setup/
  - IISinstall.yml
  - IISsetup.yml
  - conf.yml
  - hosts
~~~

- マスターPlaybook サンプル「IISsetup.yml」（Webサイトを作成・配置する場合）
~~~
# IISsetup.yml
 - name: setup IIS10
   hosts: all
   gather_facts: yes
   roles:
     - role: IIS_Setup
       VAR_WEBAPPPOOL_NAME: DefaultAppPool
       VAR_WEBSITE_NAME: myWebSite
       VAR_WEBSITE_INFO:
           hostname: localhost
           ip: 127.0.0.1
           physical_path: "C:\\inetpub\\wwwroot"
           port: 80
       VAR_WEBSITE_State: START
       tags:
         - IIS_Setup
~~~

- マスターPlaybook サンプル「IISsetup.yml」（Webサイトの起動/停止のみ実施したい場合（事前にWebサイトを作成することが必要））
~~~
# IISsetup.yml
 - name: setup IIS10
   hosts: all
   gather_facts: yes
   roles:
     - role: IIS_Setup
       VAR_WEBAPPPOOL_NAME: DefaultAppPool
       VAR_WEBSITE_NAME: myWebSite
       VAR_WEBSITE_State: START
       tags:
         - IIS_Setup
~~~


## Running Playbook
- extra-vars を利用する場合の実行例
~~~sh
ansible-playbook IISsetup.yml  -i hosts --extra-vars="@conf.yml"
~~~

- group_vars を利用する場合の実行例  
 group_vars で指定したグループ名が webserver1 の場合
~~~sh
ansible-playbook IISsetup.yml  -i hosts -l webserver1
~~~

- host_vars を利用する場合の実行例  
 host_vars で指定したグループ名が server1 の場合
~~~sh
ansible-playbook IISsetup.yml  -i hosts -l server1
~~~

- 本Roleのみを実行する場合は、 --tags "IIS_Setup" を付け加える
~~~sh
ansible-playbook IISsetup.yml  -i hosts --extra-vars="@conf.yml" --tags "IIS_Setup"
~~~


# Copyright
---------
Copyright (c) 2019 NEC Corporation

# Author Information
------------------
NEC Corporation
