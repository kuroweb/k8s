# ArgoCDによるGitOps用のMakefile

#
# config
#

registry := docker-registry.kuroweb.net
project := auth-provider
tag_suffix := $(shell git rev-parse --short HEAD)

#
# util
#

all: apply-all commit push

apply-all: apply-backend

commit:
	git add .
	git commit -m 'auto: update docker image tag.' || echo "No changes to commit"

push:
	git push || echo "Nothing to push"

#
# backend
#

backend_tag := $(registry)/$(project)-backend-production

apply-backend:
	# backend.yaml
	awk '{gsub("image: $(backend_tag):[^ ]*", "image: $(backend_tag):$(tag_suffix)")}1' \
	backend.yaml > tmpfile && mv tmpfile backend.yaml

	# backend-migrate.yaml
	awk '{gsub("image: $(backend_tag):[^ ]*", "image: $(backend_tag):$(tag_suffix)")}1' \
	backend-migrate.yaml > tmpfile && mv tmpfile backend-migrate.yaml
