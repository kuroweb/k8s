# 概要<!-- omit in toc -->

- よく使用するコマンドなどを記述する

## Categories<!-- omit in toc -->

- [Kubernetes](#kubernetes)
	- [クラスタを初期化する](#クラスタを初期化する)
	- [クラスタ構築](#クラスタ構築)
	- [Workerノードを追加](#workerノードを追加)
	- [クラスタ構築後に各ノードをとりあえずReadyにしたい](#クラスタ構築後に各ノードをとりあえずreadyにしたい)
- [Server](#server)
	- [サーバにssh鍵を登録する](#サーバにssh鍵を登録する)
- [Ansible](#ansible)
	- [簡単な疎通確認コマンド](#簡単な疎通確認コマンド)
	- [各サーバにパッケージをインストールする](#各サーバにパッケージをインストールする)
	- [各サーバを再起動する](#各サーバを再起動する)

## Kubernetes

### クラスタを初期化する

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

### クラスタ構築

```bash
# Masterノードで実行
# userで実行する (rootユーザーで実行すると、毎回rootユーザー以外は毎回sudoつけないと動かない)
sudo kubeadm init --control-plane-endpoint=master-1
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### Workerノードを追加

```bash
# Workerノードで実行するべきコマンドを表示
kubeadm token create --print-join-command

# 上記コマンドをWorkerノードで実行
sudo kubeadm join master-1:6443 --token hogehoge --discovery-token-ca-cert-hash sha256:fugafuga

# Masterノードでノード一覧を確認
kubectl get node
```

### クラスタ構築後に各ノードをとりあえずReadyにしたい

```bash
# calicoを導入する
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml
```

## Helm

### Install

```bash
# Masterノードで実行
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

## Server

### サーバにssh鍵を登録する

```bash
ssh-copy-id username@hostname
```

## Ansible

### 簡単な疎通確認コマンド

```bash
ansible worker -m ping

#=>
# master-1 | SUCCESS => {
#     "changed": false,
#     "ping": "pong"
# }
# worker-1 | SUCCESS => {
#     "changed": false,
#     "ping": "pong"
# }
# worker-2 | SUCCESS => {
#     "changed": false,
#     "ping": "pong"
# }
```

### 各サーバにパッケージをインストールする

```bash
# iptablesをインストールする場合
ansible worker -m apt -b --ask-become-pass -a "name=iptables state=present"
```

### 各サーバを再起動する

```bash
ansible worker -b --ask-become-pass -m shell -a "reboot"
```
