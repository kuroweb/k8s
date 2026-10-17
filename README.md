# k8s<!-- omit in toc -->

- 自宅k8sサーバのマニフェスト管理用リポジトリ

## Categories<!-- omit in toc -->

- [Setup](#setup)
- [Nodes](#nodes)
- [IP allocation (MetalLB)](#ip-allocation-metallb)
- [Applications](#applications)
- [Secrets](#secrets)

## Setup

- クラスタの構築とアプリケーションの導入手順は [infra リポジトリ](https://github.com/kuroweb/infra/blob/master/docs/setup/index.md) を参照

## Nodes

```mermaid
graph
  direction LR

  subgraph k8s[" "]
    subgraph control["Control Node"]
      master-1
    end

    subgraph worker["Worker Node"]
      worker-1
      worker-2
      worker-3
    end
  end

  control-->worker
```

## IP allocation (MetalLB)

- プール: `10.15.100.200-10.15.100.250` (evpnNt01), `10.15.200.200-10.15.200.250` (evpnNt02)
- 本表は実機の LoadBalancer Service の割当（2026-10-18 時点）。新規 Service で LoadBalancer を張る場合は未使用 IP を選ぶこと。
- 確認コマンド: `kubectl get svc -A | grep LoadBalancer`

| IP | Namespace | Service / 用途 |
| -- | -- | -- |
| 10.15.100.200 | price-monitoring-production | backend-camoufox-service |
| 10.15.100.201 | longhorn-system | Longhorn UI（longhorn-frontend） |
| 10.15.100.203 | price-monitoring-integration | backend-batch-4-service |
| 10.15.100.205 | price-monitoring-integration | backend-batch-1-service |
| 10.15.100.206 | price-monitoring-integration | backend-batch-2-service |
| 10.15.100.207 | portainer | Portainer |
| 10.15.100.208 | argocd | Argo CD（argocd-server） |
| 10.15.100.209 | price-monitoring-production | backend-batch-1-service |
| 10.15.100.210 | price-monitoring-production | backend-service |
| 10.15.100.212 | price-monitoring-production | backend-db-service |
| 10.15.100.213 | price-monitoring-production | backend-redis-service |
| 10.15.100.214 | price-monitoring-production | backend-batch-2-service |
| 10.15.100.215 | price-monitoring-production | backend-playwright-service |
| 10.15.100.216 | price-monitoring-production | backend-batch-3-service |
| 10.15.100.217 | price-monitoring-production | frontend-service |
| 10.15.100.220 | ingress-nginx | Ingress Controller（本番・integration 共通） |
| 10.15.100.221 | price-monitoring-production | backend-batch-4-service |
| 10.15.100.222 | price-monitoring-integration | backend-db-service |
| 10.15.100.223 | price-monitoring-integration | backend-redis-service |
| 10.15.100.225 | price-monitoring-integration | backend-playwright-service |
| 10.15.100.226 | price-monitoring-integration | backend-camoufox-service |
| 10.15.100.227 | price-monitoring-integration | frontend-service |
| 10.15.100.228 | price-monitoring-integration | backend-service |
| 10.15.100.230 | auth-provider-production | backend-service |
| 10.15.100.231 | auth-provider-production | backend-db-service |
| 10.15.100.234 | auth-provider-integration | backend-service |
| 10.15.100.235 | auth-provider-integration | backend-db-service |
| 10.15.100.236 | auth-provider-integration | backend-redis-service |
| 10.15.200.200 | price-monitoring-integration | backend-batch-3-service |
| 10.15.200.201 | auth-provider-production | backend-redis-service |

## Applications

| Application | URL |
| -- | -- |
| Longhorn | https://longhorn.kuroweb.net |
| docker-registry | https://docker-registry.kuroweb.net |
| Portainer | https://portainer.kuroweb.net |
| Argo CD | https://argocd.kuroweb.net |
| price-monitoring → Frontend | https://price-monitoring.kuroweb.net |
| price-monitoring → Backend API | https://price-monitoring.kuroweb.net/api |
| price-monitoring → Sidekiq管理画面 | https://price-monitoring.kuroweb.net/sidekiq |
| price-monitoring → Auth Provider | https://auth.price-monitoring.kuroweb.net |

## Secrets

- プライベートリポジトリで管理する方針のため、本リポジトリには追加しないこと
