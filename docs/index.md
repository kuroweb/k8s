# 概要<!-- omit in toc -->

- よく使用するコマンドなどを記述する

## Categories<!-- omit in toc -->

- [クラスタを初期化する](#クラスタを初期化する)
- [クラスタ構築](#クラスタ構築)
- [Workerノードを追加](#workerノードを追加)
- [クラスタ構築後に各ノードをとりあえずReadyにしたい](#クラスタ構築後に各ノードをとりあえずreadyにしたい)

## クラスタを初期化する

1. 対象クラスタにssh
2. `sudo su`
3. 初期化コマンド実行

	```bash
	sudo kubeadm reset --force
	sudo systemctl stop kubelet
	sudo rm -rf /etc/kubernetes/
	rm -rf ~/.kube/
	sudo rm -rf /var/lib/kubelet/
	sudo rm -rf /var/lib/cni/
	sudo rm -rf /etc/cni/
	sudo rm -rf /var/lib/etcd/
	sudo iptables -F && iptables -X
	```

## クラスタ構築

```bash
# userで実行する (rootユーザーで実行すると、毎回rootユーザー以外は毎回sudoつけないと動かない)
sudo kubeadm init --control-plane-endpoint=k8s-master
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## Workerノードを追加

```bash
# Workerノードで実行するべきコマンドを表示
kubeadm token create --print-join-command

# 上記コマンドをWorkerノードで実行
sudo kubeadm join k8s-master:6443 --token hogehoge --discovery-token-ca-cert-hash sha256:fugafuga

# Masterノードでノード一覧を確認
kubectl get node
```

## クラスタ構築後に各ノードをとりあえずReadyにしたい

```bash
# calicoを導入する
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml
```
