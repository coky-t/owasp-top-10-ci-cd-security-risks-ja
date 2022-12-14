---

layout: col-sidebar
title: "CICD-SEC-8: サードパーティサービスの無秩序な使用 (Ungoverned Usage of 3rd Party Services)"

---
## 定義

CI/CD の攻撃対象領域は SCM や CI などの組織の有機的資産と、それらの有機的資産へのアクセスが許可されているサードパーティサービスで構成されます。サードパーティサービスの無秩序な使用に伴うリスクは、サードパーティサービスが CI/CD システムのリソースへのアクセスを非常に簡単に許可できることに依存しており、組織の攻撃対象領域を効果的に拡大します。


## 解説

CI/CD システムとプロセスにほとんどサードパーティが接続されていない組織を見つけることは稀です。その実装の容易さと即時的な価値により、サードパーティは日々のエンジニアリングに不可欠なものとなっています。サードパーティへの組み込みやアクセスの付与の手法は多様化し、それらの実装に伴う複雑さは減少しています。

一般的な SCM である GitHub SAAS を例にとると、サードパーティアプリケーションは以下の 5 つの手法のいずれかひとつまたは複数を介して接続できます。



* GitHub アプリケーション
* OAuth アプリケーション
* サードパーティアプリケーションに提供されるアクセストークンのプロビジョニング
* サードパーティアプリケーションに提供される SSH キーのプロビジョニング
* サードパーティに送信される webhook イベントの構成

各手法は数秒から数分で実装でき、単一リポジトリのコードの読み取りから GitHub 組織の完全な管理まで、多くの機能をサードパーティに付与します。これらのサードパーティはシステムに対して高いレベルのパーミッションが付与される可能性がありますが、多くの場合、実際の実装に先立って組織による特別な許可や承認は必要としません。

ビルドシステムにはサードパーティを簡単に統合することもできます。サードパーティをビルドパイプラインに統合するには、通常、パイプライン構成ファイル内に 1-2 行のコードを追加するか、ビルドシステムのマーケットプレイスからプラグインをインストールするだけです (例: Github Actions の actions, CircleCI の Orbs) 。サードパーティの機能はビルドプロセスの一部としてインポートおよび実行され、それが実行されるパイプラインステージから利用可能なあらゆるリソースにフルアクセスできます。

同様の接続手法がほとんどの CI/CD システムにおいてさまざまな形で利用できるため、エンジニアリングエコシステム全体でサードパーティの使用に関する最小権限を管理および維持するプロセスが非常に複雑になります。組織はどのサードパーティがさまざまなシステムにアクセスできるか、どのような手法でアクセスできるか、どのレベルのパーミッションやアクセスが付与されているか、どのレベルのパーミッションやアクセスが実際に使用されているか、を完全に可視化する課題に取り組んでいます。


## 影響

サードパーティの実装に関するガバナンスと可視性の欠如により、組織は CI/CD システム内で RBAC を維持できません。サードパーティが寛容な傾向にあることを考えると、組織のセキュリティは彼らが実装するサードパーティと同程度にしかなりません。サードパーティに関する RBAC と最小権限の実装が不十分であることに加え、サードパーティの実装プロセスに関するガバナンスとデリジェンスも最小限であるため、組織の攻撃対象領域が大幅に増加します。

CI/CD システムと環境が高度に相互接続されているため、単一のサードパーティの侵害を利用して、そのサードパーティが接続しているシステムのスコープをはるかに超えて損害を引き起こす可能性があります (例えば、リポジトリへの書き込みパーミッションを持つサードパーティを利用して、攻撃者がリポジトリにコードをプッシュすると、それによってビルドがトリガーされ、ビルドシステム上で攻撃者の悪意のあるコードが実行される可能性があります) 。


## 推奨事項

サードパーティサービスに関するガバナンスコントロールはサードパーティ使用ライフサイクルのすべてのステージにおいて実装すべきです。



* **承認 (Approval)** - エンジニアリングエコシステムのあらゆるリソースへのアクセスを付与されたサードパーティが、環境へのアクセスを付与される前に承認されるようにするための審査手順を確立し、付与されるパーミッションレベルが最小権限の原則に沿うようにします。
* **統合 (Integration)** - CI/CD システムに総合されたすべてのサードパーティを継続的に可視化するために、以下のようなコントロールと手順を導入します。
    * 統合の手法。各システムのすべての統合手法がカバーされていることを確認します (マーケットプレイスアプリ、プラグイン、OAuth アプリケーション、プログラマチックアクセストークンなどを含む) 。
    * サードパーティに付与されるパーミッションのレベル。
    * サードパーティによって実際に使用されているパーミッションのレベル。
* **継続的な使用状況の可視化 (Visibility over ongoing usage)** - 各サードパーティがアクセスする必要がある特定のリソースに制限およびスコープされていることを確認し、使用されていないパーミッションや冗長なパーミッションを削除します。ビルドプロセスの一部として統合されるサードパーティはシークレットとコードへのアクセスが制限され、厳格なイングレスフィルタとエグレスフィルタを使用して、スコープされたコンテキスト内で実行すべきです。
* **プロビジョニング解除 (Deprovisioning)** - 統合されたすべてのサードパーティを定期的にレビューし、使用されなくなったものを削除します。


## 参考情報



1. CI で使用される一般的なコードカバレッジツールである Codecov が侵害され、ビルドから環境変数が窃取されました。

    [https://about.codecov.io/security-update/](https://about.codecov.io/security-update/)

2. 攻撃者は DeepSource (静的解析プラットフォーム) エンジニアの GitHub ユーザーアカウントを侵害しました。侵害したアカウントを使用して、DeepSource GitHub アプリケーションのパーミッションを取得し、侵害された GitHub アプリケーションをインストールしたすべての DeepSource クライアントのコードベースへのフルアクセスを付与しました。

    [https://discuss.deepsource.io/t/security-incident-on-deepsource-s-github-application/131](https://discuss.deepsource.io/t/security-incident-on-deepsource-s-github-application/131)

3. 攻撃者は git 分析プラットフォームである Waydev のデータベースへのアクセスを獲得し、顧客の GitHub および GitLab OAuth トークンを窃取しました。

    [https://changelog.waydev.co/github-and-gitlab-oauth-security-update-dw98s](https://changelog.waydev.co/github-and-gitlab-oauth-security-update-dw98s)
