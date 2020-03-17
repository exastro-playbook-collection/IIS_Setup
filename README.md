# IIS
===============================
# Trademarks
-----------
* Linuxは、Linus Torvalds氏の米国およびその他の国における登録商標または商標です。
* RedHat、RHEL、CentOSは、Red Hat, Inc.の米国およびその他の国における登録商標または商標です。
* Windows、PowerShell、IIS、.NET Frameworkは、Microsoft Corporation の米国およびその他の国における登録商標または商標です。
* Ansibleは、Red Hat, Inc.の米国およびその他の国における登録商標または商標です。
* pythonは、Python Software Foundationの登録商標または商標です。
* NECは、日本電気株式会社の登録商標または商標です。
* その他、本ロールのコード、ファイルに記載されている会社名および製品名は、各社の登録商標または商標です。

## 概要
本プロジェクトのリポジトリでIIS(Internet Information Server)のインストール、セットアップRoleを公開しています。  
対象バージョンは以下のバージョンとなります。  
・IIS10  
　  
対応OSは以下となります。  
・Windows Server 2016  
  
各Roleの説明はREADME_role.mdをご参照ください。
  
## Roleを利用して、一括でIISのインストールとセットアップを実施すること
RoleごとにIISのインストールとセットアップを行うことができるが、一括でIISをインストールして、セットアップを行うこともできます。  
以下はそのようなPlaybookの構成や利用方法を説明します。

１．構成の例：

~~~~
Playbook	                        #対象ソース（playbook）
 |-- evidence_examples
 |    |-- ansible.cfg   	        #自分の環境で利用するAnsible設定ファイル
 |    |-- IISinstall.yml	        #証拠取得用Playbookのメインファイル
 |    |-- IISinstall_evidence.yml	#IISインストールロールの証拠取得用ファイル
 |    `-- IISsetup_evidence.yml		#IIS設定ロールの証拠取得用ファイル
 |-- roles
 |    |-- IIS_Install		        #IISインストールの用Role
 |    `-- IIS_Setup	                #IIS設定用Role
 |-- IISinstall.yml         		#Playbookのメインファイル
 |-- hosts                   		#ホストファイル
 `-- README.md
~~~~

２．Playbookの例：

■エビデンス収集しないマスタPlaybookの例：

~~~
#IISinstall.yml
---
  - hosts: iis
    roles:
      - role: IIS_Install
        VAR_IIS_INSTALLTYPE: DEFAULT
      - role: IIS_Setup
        VAR_WEBAPPPOOL_NAME: myWebAppPool
        VAR_WEBSITE_NAME: myWebSite1
        VAR_WEBSITE_INFO:
            hostname: myWebSite1.co.jp
            ip: 192.168.0.1
            port: 8080
      - role: IIS_Setup
        VAR_WEBAPPPOOL_NAME: myWebAppPool
        VAR_WEBSITE_NAME: myWebSite2
        VAR_WEBSITE_INFO:
            hostname: myWebSite2.co.jp
            ip: 192.168.0.2
            port: 8090 
~~~

■エビデンス収集するマスタPlaybookの例：

~~~
#IISinstall_evidence.yml
---
- hosts: iis
  roles:
    - role: gathering
      VAR_gathering_definition_role_path: "roles/IIS_Install"
~~~

~~~
#IISsetup_evidence.yml
---
- hosts: iis
  roles:
    - role: gathering
      VAR_gathering_definition_role_path: "roles/IIS_Setup"
~~~

~~~
#IISinstall.yml
---
- import_playbook: IISinstall_evidence.yml VAR_gathering_label=before
  - hosts: iis
    roles:
      - role: IIS_Install
        VAR_IIS_INSTALLTYPE: DEFAULT
- import_playbook: IISinstall_evidence.yml VAR_gathering_label=after

# IISsetup_evidence.ymlを使用するためには、
# 事前にVAR_IISInstall_Pathの設定が必要なため、
# ここで取得する。
- hosts: iis
  tasks:
    #get VAR_IISInstall_Path
  - name: get IISInstall Path
    win_reg_stat:
      path: HKLM:\SOFTWARE\Microsoft\InetStp
      name: InstallPath
    register: installPath_result
  - set_fact:
       VAR_IISInstall_Path: "{{ installPath_result.value }}"
    when: installPath_result.exists == true

- import_playbook: IISsetup_evidence.yml VAR_gathering_label=before
  - hosts: iis
    roles:
      - role: IIS_Setup
        VAR_WEBAPPPOOL_NAME: myWebAppPool
        VAR_WEBSITE_NAME: myWebSite1
        VAR_WEBSITE_INFO:
            hostname: myWebSite1.co.jp
            ip: 192.168.0.1
            port: 8080
      - role: IIS_Setup
        VAR_WEBAPPPOOL_NAME: myWebAppPool
        VAR_WEBSITE_NAME: myWebSite2
        VAR_WEBSITE_INFO:
            hostname: myWebSite2.co.jp
            ip: 192.168.0.2
            port: 8090 

- import_playbook: IISsetup_evidence.yml VAR_gathering_label=after
~~~

３．呼び出し方  
  一括でIISのインストール、IISの設定を実施したい場合、下記のとおりで実行します。  
  ※evidenceを収集したい場合、先に「evidence\_examples」フォルダ下の四つのファイルを「playbook」フォルダにコピーする必要です。  
  　なお、Gathringロールをダウンロードして、「Playbook\roles\」にコピーする必要です。  
  ※①「IISinstall.yml」  
  ※②「IISinstall\_evidence.yml」  
  ※③「IISsetup\_evidence.yml」  
  ※④「ansible.cfg」  

~~~sh
ansible-playbook IISinstall.yml --extra-vars="@{当該パラメータファイルのパス}/xxxxx.yml" -i ./hosts
~~~


# Copyright
---------
Copyright (c) 2019 NEC Corporation

# Author Information
------------------
NEC Corporation
