---
title: Azure のコンピューター ビジョン ソリューションを開発する
permalink: index.html
layout: home
---

次の演習は、Microsoft Azure で Computer Vision ソリューションを作成するときに開発者が行う一般的なタスクについて調べる、実践的な学習エクスペリエンスを提供するように設計されています。

> **注**: これらの演習を完了するには、必要な Azure リソースと生成 AI モデルをプロビジョニングするのに十分なアクセス許可とクォータがある Azure サブスクリプションが必要です。 まだお持ちでない場合は、[Azure アカウント](https://azure.microsoft.com/free)にサインアップできます。 新規ユーザーには、最初の 30 日間のクレジットが付属する無料試用版オプションがあります。

## 演習

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %} {% for activity in labs  %}
<hr>
### [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) {{activity.lab.description}} {% endfor %}

> **注**: これらの演習は単独でも完了できますが、[Microsoft Learn](https://learn.microsoft.com/training/paths/create-computer-vision-solutions-azure-ai/) のモジュールを補完するように設計されています。このモジュールでは、これらの演習の基になる概念の一部について詳しく説明しています。
