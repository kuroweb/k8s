# k8s<!-- omit in toc -->

- 自宅k8sサーバのマニフェスト管理用リポジトリ

## Categories<!-- omit in toc -->

- [Nodes](#nodes)
- [Applications](#applications)
- [Secrets](#secrets)

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

## Applications

| Application | URL |
| -- | -- |
| Longhorn | http://10.15.100.201 |
| docker-registry | https://docker-registry.kuroweb.net |
| Portainer | http://10.15.100.207:9000 |
| Argo CD | https://10.15.100.208 |
| price-monitoring - Frontend | https://price-monitoring.kuroweb.net |
| price-monitoring - Backend API | https://price-monitoring.kuroweb.net/api |
| price-monitoring - Sidekiq管理画面 | https://price-monitoring.kuroweb.net/sidekiq |
| price-monitoring - Auth Provider | https://auth.price-monitoring.kuroweb.net |

## Secrets

- プライベートリポジトリで管理する方針のため、本リポジトリには追加しないこと
