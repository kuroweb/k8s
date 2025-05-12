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

all: apply-all commit push

apply-all: apply-backend apply-backend-playwright apply-frontend apply-auth-provider

commit:
	git add .
	git commit -m 'deploy'

push:
	git push

#
# backend
#

backend_tag := $(registry)/$(project)-backend

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

	# backend-migrate.yaml
	awk '{gsub("image: $(backend_tag):[^ ]*", "image: $(backend_tag):$(tag_suffix)")}1' \
	backend-migrate.yaml > tmpfile && mv tmpfile backend-migrate.yaml

#
# backend-playwright.yaml
#

backend_playwright_tag := $(registry)/$(project)-backend-playwright

apply-backend-playwright:
	awk '{gsub("image: $(backend_playwright_tag):[^ ]*", "image: $(backend_playwright_tag):$(tag_suffix)")}1' \
	backend-playwright.yaml > tmpfile && mv tmpfile backend-playwright.yaml

#
# frontend
#

frontend_tag := $(registry)/$(project)-frontend

apply-frontend:
	awk '{gsub("image: $(frontend_tag):[^ ]*", "image: $(frontend_tag):$(tag_suffix)")}1' \
	frontend.yaml > tmpfile && mv tmpfile frontend.yaml

#
# auth-provider.yaml
#

auth_provider_tag := $(registry)/$(project)-auth-provider

apply-auth-provider:
	awk '{gsub("image: $(auth_provider_tag):[^ ]*", "image: $(auth_provider_tag):$(tag_suffix)")}1' \
	auth-provider.yaml > tmpfile && mv tmpfile auth-provider.yaml
