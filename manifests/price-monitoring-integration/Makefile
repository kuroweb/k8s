# ArgoCDによるGitOps用のMakefile (integration)

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

apply-all: apply-backend apply-backend-playwright apply-frontend

commit:
	git add .
	git commit -m 'auto: update docker image tag (integration).' || echo "No changes to commit"

push:
	git push || echo "Nothing to push"

#
# backend
#

backend_tag := $(registry)/$(project)-backend-integration

apply-backend:
	awk '{gsub("image: $(backend_tag):[^ ]*", "image: $(backend_tag):$(tag_suffix)")}1' \
	backend.yaml > tmpfile && mv tmpfile backend.yaml
	awk '{gsub("image: $(backend_tag):[^ ]*", "image: $(backend_tag):$(tag_suffix)")}1' \
	backend-batch-1.yaml > tmpfile && mv tmpfile backend-batch-1.yaml
	awk '{gsub("image: $(backend_tag):[^ ]*", "image: $(backend_tag):$(tag_suffix)")}1' \
	backend-batch-2.yaml > tmpfile && mv tmpfile backend-batch-2.yaml
	awk '{gsub("image: $(backend_tag):[^ ]*", "image: $(backend_tag):$(tag_suffix)")}1' \
	backend-batch-3.yaml > tmpfile && mv tmpfile backend-batch-3.yaml
	awk '{gsub("image: $(backend_tag):[^ ]*", "image: $(backend_tag):$(tag_suffix)")}1' \
	backend-batch-4.yaml > tmpfile && mv tmpfile backend-batch-4.yaml
	awk '{gsub("image: $(backend_tag):[^ ]*", "image: $(backend_tag):$(tag_suffix)")}1' \
	backend-migrate.yaml > tmpfile && mv tmpfile backend-migrate.yaml

#
# backend-playwright
#

backend_playwright_tag := $(registry)/$(project)-backend-playwright-integration

apply-backend-playwright:
	awk '{gsub("image: $(backend_playwright_tag):[^ ]*", "image: $(backend_playwright_tag):$(tag_suffix)")}1' \
	backend-playwright.yaml > tmpfile && mv tmpfile backend-playwright.yaml

#
# frontend
#

frontend_tag := $(registry)/$(project)-frontend-integration

apply-frontend:
	awk '{gsub("image: $(frontend_tag):[^ ]*", "image: $(frontend_tag):$(tag_suffix)")}1' \
	frontend.yaml > tmpfile && mv tmpfile frontend.yaml
