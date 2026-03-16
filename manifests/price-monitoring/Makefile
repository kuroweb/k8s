# ArgoCDによるGitOps用のMakefile

registry := docker-registry.kuroweb.net
tag_suffix := $(shell git rev-parse --short HEAD)

all: update-images-production commit push
all-integration: update-images-integration commit push

commit:
	git add .
	git commit -m 'auto: update docker image tag.' || echo "No changes to commit"

push:
	git push || echo "Nothing to push"

update-images-production: apply-backend-production apply-backend-playwright-production apply-frontend-production

apply-backend-production:
	cd overlays/production && \
	awk -v tag="$(tag_suffix)" \
		'/price-monitoring-backend:placeholder/{f=1} f && /newTag:/{sub(/newTag: .*/, "newTag: " tag); f=0} {print}' \
		kustomization.yaml > kustomization.yaml.tmp && \
	mv kustomization.yaml.tmp kustomization.yaml

apply-backend-playwright-production:
	cd overlays/production && \
	awk -v tag="$(tag_suffix)" \
		'/price-monitoring-backend-playwright:placeholder/{f=1} f && /newTag:/{sub(/newTag: .*/, "newTag: " tag); f=0} {print}' \
		kustomization.yaml > kustomization.yaml.tmp && \
	mv kustomization.yaml.tmp kustomization.yaml

apply-frontend-production:
	cd overlays/production && \
	awk -v tag="$(tag_suffix)" \
		'/price-monitoring-frontend:placeholder/{f=1} f && /newTag:/{sub(/newTag: .*/, "newTag: " tag); f=0} {print}' \
		kustomization.yaml > kustomization.yaml.tmp && \
	mv kustomization.yaml.tmp kustomization.yaml

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
