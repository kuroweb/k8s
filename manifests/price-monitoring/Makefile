# ArgoCDによるGitOps用のMakefile

#
# config
#

registry := docker-registry.kuroweb.net
project := price-monitoring
tag_suffix := $(shell git rev-parse --short HEAD)

#
# util
#

all: apply-all commit push

# integration: Kustomize overlay のイメージタグを更新し commit/push まで行う
all-integration: update-images-integration commit push

apply-all: apply-backend apply-backend-playwright apply-frontend

commit:
	git add .
	git commit -m 'auto: update docker image tag.' || echo "No changes to commit"

push:
	git push || echo "Nothing to push"

#
# integration overlay: kustomization.yaml の newTag を置換（kustomize CLI 不要）
#

update-images-integration: apply-backend-integration apply-backend-playwright-integration apply-frontend-integration

apply-backend-integration:
	cd overlays/integration && \
	awk -v tag="$(tag_suffix)" \
		'/price-monitoring-backend:placeholder/{f=1} f && /newTag:/{sub(/newTag: .*/, "newTag: " tag); f=0} {print}' \
		kustomization.yaml > kustomization.yaml.tmp && \
	mv kustomization.yaml.tmp kustomization.yaml

apply-backend-playwright-integration:
	cd overlays/integration && \
	awk -v tag="$(tag_suffix)" \
		'/price-monitoring-backend-playwright:placeholder/{f=1} f && /newTag:/{sub(/newTag: .*/, "newTag: " tag); f=0} {print}' \
		kustomization.yaml > kustomization.yaml.tmp && \
	mv kustomization.yaml.tmp kustomization.yaml

apply-frontend-integration:
	cd overlays/integration && \
	awk -v tag="$(tag_suffix)" \
		'/price-monitoring-frontend:placeholder/{f=1} f && /newTag:/{sub(/newTag: .*/, "newTag: " tag); f=0} {print}' \
		kustomization.yaml > kustomization.yaml.tmp && \
	mv kustomization.yaml.tmp kustomization.yaml

#
# backend
#

backend_tag := $(registry)/$(project)-backend-production

apply-backend:
	# backend.yaml
	awk '{gsub("image: $(backend_tag):[^ ]*", "image: $(backend_tag):$(tag_suffix)")}1' \
	backend.yaml > tmpfile && mv tmpfile backend.yaml

	# backend-batch-1.yaml
	awk '{gsub("image: $(backend_tag):[^ ]*", "image: $(backend_tag):$(tag_suffix)")}1' \
	backend-batch-1.yaml > tmpfile && mv tmpfile backend-batch-1.yaml

	# backend-batch-2.yaml
	awk '{gsub("image: $(backend_tag):[^ ]*", "image: $(backend_tag):$(tag_suffix)")}1' \
	backend-batch-2.yaml > tmpfile && mv tmpfile backend-batch-2.yaml

	# backend-batch-3.yaml
	awk '{gsub("image: $(backend_tag):[^ ]*", "image: $(backend_tag):$(tag_suffix)")}1' \
	backend-batch-3.yaml > tmpfile && mv tmpfile backend-batch-3.yaml

	# backend-batch-4.yaml
	awk '{gsub("image: $(backend_tag):[^ ]*", "image: $(backend_tag):$(tag_suffix)")}1' \
	backend-batch-4.yaml > tmpfile && mv tmpfile backend-batch-4.yaml

	# backend-migrate.yaml
	awk '{gsub("image: $(backend_tag):[^ ]*", "image: $(backend_tag):$(tag_suffix)")}1' \
	backend-migrate.yaml > tmpfile && mv tmpfile backend-migrate.yaml

#
# backend-playwright.yaml
#

backend_playwright_tag := $(registry)/$(project)-backend-playwright-production

apply-backend-playwright:
	awk '{gsub("image: $(backend_playwright_tag):[^ ]*", "image: $(backend_playwright_tag):$(tag_suffix)")}1' \
	backend-playwright.yaml > tmpfile && mv tmpfile backend-playwright.yaml

#
# frontend
#

frontend_tag := $(registry)/$(project)-frontend-production

apply-frontend:
	awk '{gsub("image: $(frontend_tag):[^ ]*", "image: $(frontend_tag):$(tag_suffix)")}1' \
	frontend.yaml > tmpfile && mv tmpfile frontend.yaml
