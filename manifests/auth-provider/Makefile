# ArgoCDによるGitOps用のMakefile

tag_suffix := $(shell git rev-parse --short HEAD)
ENV ?= production
OVERLAY_DIR := overlays/$(ENV)

all: update-images commit push

commit:
	git add .
	git commit -m 'auto: update docker image tag.' || echo "No changes to commit"

push:
	git push || echo "Nothing to push"

update-images:
	$(MAKE) update-image ENV=$(ENV) IMAGE=auth-provider-backend

update-image:
	test -n "$(IMAGE)"
	cd $(OVERLAY_DIR) && \
	awk -v image="$(IMAGE)" -v tag="$(tag_suffix)" \
		'$$0 ~ image ":placeholder" {f=1} f && /newTag:/ {sub(/newTag: .*/, "newTag: " tag); f=0} {print}' \
		kustomization.yaml > kustomization.yaml.tmp && \
	mv kustomization.yaml.tmp kustomization.yaml
