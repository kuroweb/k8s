# ArgoCDによるGitOps用のMakefile

#
# config
#

registry := registry.local:5000
project := price-monitoring
tag_suffix := $(shell git rev-parse --short HEAD)

#
# util
#

all: apply-backend apply-backend-batch apply-backend-migrate apply-backend-playwright apply-backend-tor apply-all commit push

commit:
	git add .
	git commit -m 'deploy'

push:
	git push

#
# backend.yaml
#

backend_tag := $(registry)/$(project)-backend

apply-backend:
	awk '{gsub("image: $(backend_tag):[^ ]*", "image: $(backend_tag):$(tag_suffix)")}1' \
	backend.yaml > tmpfile && mv tmpfile backend.yaml

#
# backend-batch.yaml
#

backend_batch_tag := $(registry)/$(project)-backend

apply-backend-batch:
	awk '{gsub("image: $(backend_batch_tag):[^ ]*", "image: $(backend_batch_tag):$(tag_suffix)")}1' \
	backend-batch.yaml > tmpfile && mv tmpfile backend-batch.yaml

#
# backend-migrate.yaml
#

backend_migrate_tag := $(registry)/$(project)-backend

apply-backend-migrate:
	awk '{gsub("image: $(backend_migrate_tag):[^ ]*", "image: $(backend_migrate_tag):$(tag_suffix)")}1' \
	backend-migrate.yaml > tmpfile && mv tmpfile backend-migrate.yaml

#
# backend-playwright.yaml
#

backend_playwright_tag := $(registry)/$(project)-backend-playwright

apply-backend-playwright:
	awk '{gsub("image: $(backend_playwright_tag):[^ ]*", "image: $(backend_playwright_tag):$(tag_suffix)")}1' \
	backend-playwright.yaml > tmpfile && mv tmpfile backend-playwright.yaml

#
# backend-tor.yaml
#

backend_tor_tag := $(registry)/$(project)-backend-tor

apply-backend-tor:
	awk '{gsub("image: $(backend_tor_tag):[^ ]*", "image: $(backend_tor_tag):$(tag_suffix)")}1' \
	backend-tor.yaml > tmpfile && mv tmpfile backend-tor.yaml

#
# tor
#

#
# playwright
#

#
# bff
#

#
# frontend
#
