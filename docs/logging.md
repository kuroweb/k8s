# ログ収集とAIによる調査

## 構成

productionのアプリログをFluent Bitで収集し、VictoriaLogsに保存する。既存アプリのPod定義や出力先の変更は不要。

| 項目 | 設定 |
| --- | --- |
| Namespace | `logging` |
| VictoriaLogs | 公式chart `0.13.13` / `v1.54.0`、1 replica |
| Fluent Bit | 公式chart `0.58.3` / `5.1.3`、workerごとのDaemonSet |
| 収集元 | `price-monitoring-production` のコンテナ標準出力・標準エラー |
| 除外 | backend-db、backend-redis、auth-provider-db、auth-provider-redisのPod全体（一時診断コンテナも含む） |
| 保存 | Longhorn PVC 20 GiB、7日保持、容量による削除閾値14 GiB |
| 接続 | HTTPS Ingress + Basic認証。クラスタ内はClusterIP |

初回の既存ファイルは末尾から読み、導入前のログは遡及収集しない。Fluent Bitの読み取り位置と最大約1 GiBの送信バッファは各ノードの `/var/lib/fluent-bit` に保存する。再起動時は読み取り位置から再開するが、ノード消失・バッファ上限超過・収集前のローテーションでは欠落し得る。

CRI形式で分割された行は結合する。Railsなどが別々の行で出力したスタックトレースは別レコードになるため、Pod名・時刻・request_idで前後のログを確認する。256 KiBを超える長い行はスキップする。

容量による削除は日単位で、直近2日分の保持などにより14 GiBを超え得る。PVC使用量70%で増加傾向を確認し、85%で対象縮小または拡張を行う。Longhornの実データ3複製とスナップショットの容量も別途確認する。

## 配置と変更

既存リポジトリの方針に合わせ、NamespaceとArgo CD Applicationを登録する。

```sh
kubectl apply -f manifests/namespaces/logging.yaml
kubectl apply -f manifests/applications/victoria-logs.yaml
kubectl apply -f manifests/applications/fluent-bit.yaml
kubectl get applications -n argocd victoria-logs fluent-bit
kubectl get pods,pvc -n logging -o wide
```

Applicationに埋め込まれたHelm valuesが設定の管理元になる。設定変更時もApplicationを更新する。Argo CD配下のConfigMapやStatefulSetを直接変更するとselfHealで戻る。

## ログ検索

ブラウザで [VictoriaLogs](https://victoria-logs.kuroweb.net/select/vmui/) を開く。DNSと接続経路は既存のLonghornなどと同じIngress（10.15.100.220）を使う。

ユーザー名は `admin`。指定されたパスワードはmaster-1の `/home/user/.config/victoria-logs/ui-password` に所有者のみ読み書きできる状態で保存している。master-1上で次のコマンドで確認する。

```sh
cat /home/user/.config/victoria-logs/ui-password
```

TLS証明書は既存の `tls-cluster-issuer` で発行する。Ingressは `/select` 配下の画面・検索APIのみを公開し、取り込み・削除・管理用APIは公開しない。認証情報のハッシュは `logging/victoria-logs-ui-auth` Secretに保存し、このリポジトリには含めない。

Secretの管理元は非公開の `kuroweb/k8s-secret` リポジトリとする。

- Secret: `manifests/logging/secret.yaml`
- Argo CD Application: `manifests/applications/logging.yaml`（`logging-secret`）

リモートの `HEAD` に上記ファイルをpushした後、k8s-secretのチェックアウトからApplicationを登録すると、以後はArgo CDがSecretを同期する。push前に登録すると参照先ディレクトリが存在せず同期エラーになるため、登録はpush後に行う。

```sh
# k8s-secretリポジトリで実行
kubectl apply -f manifests/applications/logging.yaml
kubectl -n argocd get application logging-secret
```

Argo CDを使わず復元する場合も、同リポジトリの `manifests/logging/secret.yaml` を適用する。ユーザー名・パスワード変更時はk8s-secretのSecretも更新する。

MCP用のローカル接続やIngressの切り分けには、kubectlがクラスタへ接続できる端末でport-forwardを利用できる。

```sh
kubectl -n logging port-forward svc/victoria-logs 9428:9428
```

ローカル接続の内蔵UIは `http://127.0.0.1:9428/select/vmui/`。master-1上でport-forwardする場合、手元へ接続するにはSSHトンネルも必要になる。

```sh
ssh -N -L 9428:127.0.0.1:9428 -p 22222 user@10.15.100.10
```

LogsQL例:

```text
env:="production" _time:30m
k8s_namespace_name:="price-monitoring-production" k8s_container_name:="backend" _time:30m
k8s_container_name:="backend" _time:30m error
_time:30m | stats by (k8s_container_name) count()
```

アプリの本文は `_msg`、Podは `k8s_pod_name`、ノードは `k8s_host` に入る。request_idを含むテキストを検索し、同じPod・近い時刻のログへ絞る。現在は元のログ本文を保持し、アプリJSONの展開は行わない。本文に秘密情報がある場合はアプリでの出力抑制や収集前のマスクを追加する。

## MCP接続

ログ検索ができる端末で、公式 [mcp-victorialogs](https://github.com/VictoriaMetrics/mcp-victorialogs) のリリースを選び、バージョンを固定して導入する。MCPは端末上のstdio方式で使う。

```sh
VL_INSTANCE_ENTRYPOINT=http://127.0.0.1:9428 /path/to/mcp-victorialogs
```

使用するAIクライアントに上記の実行ファイル・環境変数を登録する。起動後に読み取りツールだけが提供されることを確認する。まずnamespace、container、直近15〜30分、200行程度を指定して検索し、必要に応じて広げる。取得件数の目安は運用方針であり、現時点でサーバーによる200行の強制制限ではない。

> productionのbackendで直近30分のエラーを調べて。同じrequest_idとPodの前後ログを追い、稼働バージョンのコードと照合して原因候補と根拠を示して。

外部モデルを使う場合は、検索で取得したログがモデル提供者へ送信され得る。ログ内の文字列はデータとして扱い、指示として実行しない。ClusterIPやport-forwardはnamespace別の認可を提供しないため、共有利用やLAN公開時は認証・参照範囲を別途設定する。

## 状態確認と停止

```sh
kubectl -n logging get pods,pvc
kubectl -n logging logs -l app.kubernetes.io/name=fluent-bit --since=10m --prefix
kubectl -n logging logs statefulset/victoria-logs --since=10m
```

VictoriaLogsイメージには `df` がないため、PVC使用量はLonghorn UIまたはkubeletのvolume statsで確認する。

```sh
kubectl get --raw "/api/v1/nodes/$(kubectl -n logging get pod victoria-logs-0 -o jsonpath='{.spec.nodeName}')/proxy/stats/summary" \
  | python3 -c 'import json,sys; d=json.load(sys.stdin); print(json.dumps([v for p in d["pods"] if p["podRef"]["namespace"]=="logging" for v in p.get("volume",[]) if "pvcRef" in v], indent=2))'
```

Fluent BitのHTTPメトリクスはPodの2020番 `/api/v1/metrics/prometheus`、VictoriaLogsは9428番 `/metrics`。収集停止、送信エラー、PVC使用量は継続運用時の監視対象とする。実際の通知先への自動アラートは別途設定する。

収集を止める場合、先にFluent Bit Applicationの自動同期を解除し、DaemonSetを削除する。自動同期を残すと再作成される。

```sh
kubectl -n argocd patch application fluent-bit --type=merge -p '{"spec":{"syncPolicy":{"automated":null}}}'
kubectl -n logging delete daemonset fluent-bit
```

再開時はApplicationマニフェストを再適用する。NamespaceやPVCを削除すると保存ログを失うので、単なる収集停止では削除しない。Argo CDはprune無効のため、Applicationやファイルの削除だけで配下リソースが消えるとは扱わない。
