---
title: GitHub Enterprise Server をアップグレードする
intro: '最新の機能とセキュリティアップデートを入手するために、{% data variables.product.prodname_ghe_server %} をアップグレードしてください。'
redirect_from:
  - /enterprise/admin/installation/upgrading-github-enterprise-server
  - /enterprise/admin/articles/upgrading-to-the-latest-release
  - /enterprise/admin/articles/migrations-and-upgrades
  - /enterprise/admin/guides/installation/upgrading-the-github-enterprise-virtual-machine
  - /enterprise/admin/guides/installation/upgrade-packages-for-older-releases
  - /enterprise/admin/articles/upgrading-older-installations
  - /enterprise/admin/hidden/upgrading-older-installations
  - /enterprise/admin/hidden/upgrading-github-enterprise-using-a-hotpatch-early-access-program
  - /enterprise/admin/hidden/upgrading-github-enterprise-using-a-hotpatch
  - /enterprise/admin/guides/installation/upgrading-github-enterprise
  - /enterprise/admin/enterprise-management/upgrading-github-enterprise-server
  - /admin/enterprise-management/upgrading-github-enterprise-server
versions:
  ghes: '*'
type: how_to
topics:
  - Enterprise
  - Upgrades
shortTitle: Upgrading GHES
---

{% ifversion ghes < 3.3 %}{% data reusables.enterprise.upgrade-ghes-for-features %}{% endif %}

## アップグレードの準備

1. アップグレードの戦略を決定し、アップグレード先のバージョンを選択してください。 For more information, see "[Upgrade requirements](/enterprise/admin/guides/installation/upgrade-requirements/)" and refer to the [{% data variables.enterprise.upgrade_assistant %}](https://support.github.com/enterprise/server-upgrade) to find the upgrade path from your current release version.
1. {% data variables.product.prodname_enterprise_backup_utilities %} で、プライマリインスタンスの新しいバックアップを作成してください。 詳しい情報については、[{% data variables.product.prodname_enterprise_backup_utilities %}README.md ファイル](https://github.com/github/backup-utils#readme)を参照してください。
1. If {% data variables.product.product_location %} uses ephemeral self-hosted runners for {% data variables.product.prodname_actions %} and you've disabled automatic updates, upgrade your runners to the version of the runner application that your upgraded instance will run.
1. アップグレードパッケージを使ってアップグレードをする場合は、{% data variables.product.prodname_ghe_server %} のエンドユーザのためにメンテナンス時間枠をスケジューリングしてください。 ホットパッチを利用しているなら、メンテナンスモードは必要ありません。

  {% note %}

  **注釈:** メンテナンスウィンドウは、実行しようとしているアップグレードの種類によります。 ホットパッチを利用するアップグレードは、通常メンテナンスウィンドウを必要としません。 リブートが必要になることもあります。そのリブートは後で行うことができます。 MAJOR.FEATURE.PATCH というバージョン付けのスキームに従ったパッチリリースをアップグレードパッケージを行うには、通常 5 分未満のダウンタイムだけが必要です。 データの移行を含むフィーチャリリースは、ストレージのパフォーマンス及び移行するデータの量に応じて時間が長くかかります。 詳しい情報については"[メンテナンスモードの有効化とスケジューリング](/enterprise/admin/guides/installation/enabling-and-scheduling-maintenance-mode)"を参照してください。

  {% endnote %}

## スナップショットの取得

スナップショットは、ある時点での仮想マシン（VM）のチェックポイントです。 アップグレードに失敗した場合にスナップショットからVMを回復できるよう、仮想マシンをアップグレードする前にスナップショットを取っておくことを強くおすすめします。 We only recommend taking a VM snapshot when the appliance is powered down or in maintenance mode and all background jobs have finished.

新しいフィーチャリリースにアップグレードするなら、VM のスナップショットを取らなければなりません。 パッチリリースへのアップグレードをするなら、既存のデータディスクをアタッチできます。

スナップショットには2種類あります。

- **VM スナップショット**は、ユーザデータおよび設定データを含む VM の状態全体を保存します。 このスナップショットの手法には大量のディスク領域が必要になり、時間がかかります。
- **データディスクスナップショット**はユーザデータだけを保存します。

  {% note %}

  **ノート:**
  - プラットフォームによっては、ユーザのデータディスクだけのスナップショットを取ることができません。 それらのプラットフォームでは、VM 全体のスナップショットを取る必要があります。
  - ハイパーバイザーが完全なVMスナップショットをサポートしていないなら、ルートディスクとデータディスクのスナップショットを時間をおかずに取らなければなりません。

  {% endnote %}

| プラットフォーム              | スナップショットの取得方法 | スナップショットドキュメンテーションのURL                                                                                                                                                                                                           |
| --------------------- | ------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Amazon AWS            | ディスク          | <https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-creating-snapshot.html>                                                                                                                                                 |
| Azure                 | VM            | <https://docs.microsoft.com/azure/backup/backup-azure-vms-first-look-arm>                                                                                                                                                        |
| Hyper-V               | VM            | <https://docs.microsoft.com/windows-server/virtualization/hyper-v/manage/enable-or-disable-checkpoints-in-hyper-v>                                                                                                               |
| Google Compute Engine | ディスク          | <https://cloud.google.com/compute/docs/disks/create-snapshots>                                                                                                                                                                   |
| VMware                | VM            | [https://pubs.vmware.com/vsphere-50/topic/com.vmware.wssdk.pg.doc_50/PG_Ch11_VM_Manage.13.3.html](https://pubs.vmware.com/vsphere-50/topic/com.vmware.wssdk.pg.doc_50/PG_Ch11_VM_Manage.13.3.html){% ifversion ghes < 3.3 %}
| XenServer             | VM            | <https://docs.citrix.com/en-us/xencenter/current-release/vms-snapshots.html>{% endif %}

## ホットパッチでのアップグレード

{% data reusables.enterprise_installation.hotpatching-explanation %}

Using the {% data variables.enterprise.management_console %}, you can install a hotpatch immediately or schedule it for later installation. 管理シェルを使って `ghe-upgrade` ユーティリティでホットパッチをインストールすることもできます。 詳細は「[アップグレードの要求事項](/enterprise/admin/guides/installation/upgrade-requirements/)」を参照してください。

{% note %}

**{% ifversion ghes %}注釈{% else %}注釈{% endif %}**:

{% ifversion ghes %}
- {% data variables.product.product_location %} がリリース候補ビルドを実行している場合、ホットパッチでアップグレードすることはできません。

- {% endif %}クラスタ環境では、{% data variables.enterprise.management_console %} を使ったホットパッチのインストールはできません。 クラスタ環境でホットパッチをインストールするには、「[クラスタをアップグレードする](/enterprise/admin/clustering/upgrading-a-cluster#upgrading-with-a-hotpatch)」を参照してください。

{% endnote %}

### ホットパッチでの単一のアプライアンスのアップグレード

#### {% data variables.enterprise.management_console %} を使ってホットパッチをインストールする

You can use the {% data variables.enterprise.management_console %} to upgrade with a hotpatch by enabling automatic updates. You will then be presented with the latest available version of {% data variables.product.prodname_ghe_server %} that you can upgrade to.

If the upgrade target you're presented with is a feature release instead of a patch release, you cannot use the {% data variables.enterprise.management_console %} to install a hotpatch. You must install the hotpatch using the administrative shell instead. For more information, see "[Installing a hotpatch using the administrative shell](#installing-a-hotpatch-using-the-administrative-shell)."

1. 自動アップデートを有効化してください。 詳しい情報については「[自動アップデートの有効化](/enterprise/admin/guides/installation/enabling-automatic-update-checks/)」を参照してください。
{% data reusables.enterprise_site_admin_settings.access-settings %}
{% data reusables.enterprise_site_admin_settings.management-console %}
{% data reusables.enterprise_management_console.updates-tab %}
4. 新しいホットパッチがダウンロードされたなら、Install package（パッケージのインストール）ドロップダウンメニューを使ってください。
    - すぐにインストールするなら**Now（即時）**を選択してください。
    - 後でインストールするなら、後の日付を選択してください。 ![ホットパッチインストール日のドロップダウン](/assets/images/enterprise/management-console/hotpatch-installation-date-dropdown.png)
5. [**Install**] をクリックします。 ![ホットパッチインストールボタン](/assets/images/enterprise/management-console/hotpatch-installation-install-button.png)

#### 管理シェルを使ったホットパッチのインストール

{% data reusables.enterprise_installation.download-note %}

{% data reusables.enterprise_installation.ssh-into-instance %}
2. {% data reusables.enterprise_installation.enterprise-download-upgrade-pkg %}アップグレードのホットパッケージ（*.hpkg*ファイル）のURLをコピーしてください。
{% data reusables.enterprise_installation.download-package %}
4. パッケージのファイル名を使って `ghe-upgrade` コマンドを実行してください。
  ```shell
  admin@<em>HOSTNAME</em>:~$ ghe-upgrade <em>GITHUB-UPGRADE.hpkg</em>
  *** verifying upgrade package signature...
  ```
5. カーネル、MySQL、Elasticsearch やその他のプログラムのアップグレードにリブートが必要なら、ホットパッチアップグレードスクリプトが通知してくれます。

### ホットパッチを使ったレプリカインスタンスを持つアプライアンスのアップグレード

{% note %}

**注釈**: ホットパッチをインストールする場合、メンテナンスモードに入ったりレプリケーションを停止したりする必要はありません。

{% endnote %}

High Availability と Geo-replication が設定されたアプライアンスは、プライマリインスタンスに加えてレプリカインスタンスを使います。 これらのアプライアンスをアップグレードするには、プライマリインスタンスとすべてのレプリカインスタンスの両方を、1 つずつアップグレードしなければなりません。

#### プライマリインスタンスのアップグレード

1. 「[管理シェルを使ってホットパッチをインストールする](#installing-a-hotpatch-using-the-administrative-shell)」の指示に従ってプライマリインスタンスをアップグレードしてください。

#### レプリカインスタンスのアップグレード

{% note %}

**メモ:** Geo-replication の一部として複数のレプリカインスタンスを動作させているなら、この手順は各レプリカインスタンス 1 つずつに繰り返してください。

{% endnote %}

1. 「[管理シェルを使ってホットパッチをインストールする](#installing-a-hotpatch-using-the-administrative-shell)」の指示に従ってレプリカインスタンスをアップグレードします。 Geo-replication に複数のレプリカを使用している場合は、この手順を繰り返して、各レプリカを一度に 1 つずつアップグレードする必要があります。
{% data reusables.enterprise_installation.replica-ssh %}
{% data reusables.enterprise_installation.replica-verify %}

## アップグレードパッケージでのアップグレード

フィーチャシリーズ内の最新のパッチリリースへのアップグレードにはホットパッチが利用できますが、新しいフィーチャリリースへのアップグレードにはアップグレードパッケージを使わなければなりません。 たとえば `2.11.10` から `2.12.4` へのアップグレードの場合、これらは異なるフィーチャシリーズなので、アップグレードパッケージを使わなければなりません。 詳細は「[アップグレードの要求事項](/enterprise/admin/guides/installation/upgrade-requirements/)」を参照してください。

### アップグレードパッケージでの単一のアプライアンスのアップグレード

{% data reusables.enterprise_installation.download-note %}

{% data reusables.enterprise_installation.ssh-into-instance %}
2. {% data reusables.enterprise_installation.enterprise-download-upgrade-pkg %} 適切なプラットフォームを選択し、アップグレードパッケージ (*.pkg*ファイル) の URL をコピーしてください。
{% data reusables.enterprise_installation.download-package %}
4. メンテナンスモードを有効にし、{% data variables.product.prodname_ghe_server %} インスタンス上のすべてのアクティブなプロセスが完了するのを待ってください。 詳しい情報については"[メンテナンスモードの有効化とスケジューリング](/enterprise/admin/guides/installation/enabling-and-scheduling-maintenance-mode)"を参照してください。

  {% note %}

  **メモ**: High Availability 構成のプライマリアプライアンスをアップグレードする場合には、「[プライマリインスタンスをアップグレードする](#upgrading-the-primary-instance)」の指示に従っているならアプライアンスはメンテナンスモードになっているはずです。

  {% endnote %}

5. パッケージのファイル名を使って `ghe-upgrade` コマンドを実行してください。
  ```shell
  admin@<em>HOSTNAME</em>:~$ ghe-upgrade <em>GITHUB-UPGRADE.pkg</em>
  *** verifying upgrade package signature...
  ```
6. アップグレードを続けてパッケージ署名検証後に再起動することを承諾します。 新しいルートファイルシステムがセカンダリパーティションに書かれ、インスタンスは自動的にメンテナンスモードで再起動します。
  ```shell
  *** applying update...
  This package will upgrade your installation to version <em>version-number</em>
  Current root partition: /dev/xvda1 [<em>version-number</em>]
  Target root partition:  /dev/xvda2
  Proceed with installation? [y/N]
  ```
{% ifversion ip-exception-list %}
1. Optionally, to validate the upgrade, configure an IP exception list to allow access to a specified list of IP addresses. For more information, see "[Validating changes in maintenance mode using the IP exception list](/admin/configuration/configuring-your-enterprise/enabling-and-scheduling-maintenance-mode#validating-changes-in-maintenance-mode-using-the-ip-exception-list)."
{% endif %}
7. 単一アプライアンスのアップグレードであれば、メンテナンスモードを無効化してユーザが {% data variables.product.product_location %} を利用できるようにしてください。

  {% note %}

  **メモ**: High Availability 設定のアプライアンスをアップグレードする場合、すべてのレプリカをアップグレードし、レプリケーションが最新の状態になるまではメンテナンスモードのままでなければなりません。 詳細は「[レプリカインスタンスをアップグレードする](#upgrading-a-replica-instance)」を参照してください。

  {% endnote %}

### アップグレードパッケージを使ったレプリカインスタンスを持つアプライアンスのアップグレード

High Availability と Geo-replication が設定されたアプライアンスは、プライマリインスタンスに加えてレプリカインスタンスを使います。 これらのアプライアンスをアップグレードするには、プライマリインスタンスとすべてのレプリカインスタンスの両方を、1 つずつアップグレードしなければなりません。

#### プライマリインスタンスのアップグレード

{% warning %}

**警告:** レプリケーションが停止されると、プライマリに障害があった場合には、レプリカがアップグレードされてレプリケーションが再開されるまでに行われた作業は失われます。

{% endwarning %}

1. プライマリインスタンスでメンテナンスモードを有効化し、すべてのアクティブなプロセスが完了するのを待ちます。 詳しい情報については、「[メンテナンスモードの有効化](/enterprise/admin/guides/installation/enabling-and-scheduling-maintenance-mode/)」を参照してください。
{% data reusables.enterprise_installation.replica-ssh %}
3. レプリカインスタンス、あるいは Geo-replication の一部として複数のレプリカインスタンスを動作させている場合は、すべてのレプリカインスタンスで `ghe-repl-stop` を実行してレプリケーションを停止させます。
4. 「[アップグレードパッケージで単一アプライアンスをアップグレードする](#upgrading-a-single-appliance-with-an-upgrade-package)」の指示に従い、プライマリインスタンスをアップグレードしてください。

#### レプリカインスタンスのアップグレード

{% note %}

**メモ:** Geo-replication の一部として複数のレプリカインスタンスを動作させているなら、この手順は各レプリカインスタンス 1 つずつに繰り返してください。

{% endnote %}

1. 「[アップグレードパッケージで単一アプライアンスをアップグレードする](#upgrading-a-single-appliance-with-an-upgrade-package)」の指示に従い、レプリカインスタンスをアップグレードします。 Geo-replication に複数のレプリカを使用している場合は、この手順を繰り返して、各レプリカを一度に 1 つずつアップグレードする必要があります。
{% data reusables.enterprise_installation.replica-ssh %}
{% data reusables.enterprise_installation.replica-verify %}

{% data reusables.enterprise_installation.start-replication %}

{% data reusables.enterprise_installation.replication-status %} コマンドが `Replication is not running` を返すなら、レプリケーションはまだ開始中かもしれません。 `ghe-repl-status` をもう一度実行する前に 1 分間ほど待ってください。

   {% note %}

    **メモ:** resync の処理が進んでいる間は、`ghe-repl-status` はレプリケーションが遅れていることを示す期待どおりのメッセージを返すかもしれません。
    たとえば `CRITICAL: git replication is behind the primary by more than 1007 repositories and/or gists` のようなメッセージです。

   {% endnote %}

   If `ghe-repl-status` did not return `OK`, contact {% data variables.contact.enterprise_support %}. 詳しい情報については、「[{% data variables.contact.github_support %} からの支援を受ける](/admin/enterprise-support/receiving-help-from-github-support)」を参照してください。

6. 最後のレプリカのアップグレードが完了し、resync も完了したなら、ユーザが {% data variables.product.product_location %} を使えるようにメンテナンスモードを無効化してください。

## 失敗したアップグレードからのリストア

アップグレードが失敗もしくは中断したなら、インスタンスを以前の状態に復帰させなければなりません。 この処理を完了させるプロセスは、アップグレードの種類によります。

### パッチリリースのロールバック

パッチリリースをロールバックするには、`--allow-patch-rollback` を付けて `ghe-upgrade` コマンドを使います。 Before rolling back, replication must be temporarily stopped by running `ghe-repl-stop` on all replica instances. {% data reusables.enterprise_installation.command-line-utilities-ghe-upgrade-rollback %}

Once the rollback is complete, restart replication by running `ghe-repl-start` on all replicas.

詳細は「[コマンドラインユーティリティ](/enterprise/admin/guides/installation/command-line-utilities/#ghe-upgrade)」を参照してください。

### フィーチャリリースのロールバック

フィーチャリリースからロールバックするには、ルートおよびデータパーティションが整合した状態になることを保証するため、VM スナップショットからリストアしてください。 詳細は「[スナップショットを取得する](#taking-a-snapshot)」を参照してください。

{% ifversion ghes %}
## 参考リンク

- 「[新しいリリースへのアップグレードについて](/admin/overview/about-upgrades-to-new-releases)」
{% endif %}
