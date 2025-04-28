# Variables
SECRET_PATH ?= secret/rubin/usdf-consdb-dev
ENV ?= transformed-efd
FILE ?=
SECRET_KEYS := aws-access-key-id \
		aws-secret-access-key \
		butler-password \
		consdb-url \
		consdb-url-testing \
		s3-profile-embargo \
		usdf-efd-password

get-secrets-from-vault:
	mkdir -p etc/.secrets/
	set -e; \
	for i in $(SECRET_KEYS); do \
		vault kv get --field=$$i $(SECRET_PATH)/$(ENV) \
			> etc/.secrets/$$i; \
	done

clean-secrets:
	rm -rf etc/.secrets/

link-patch:
ifndef FILE
	$(error Please specify a patch file with FILE=..., e.g. 'make apply FILE=patch-xyz.yaml')
endif
	@echo "Linking patch file: $(FILE)"
	ln -sf $(FILE) patch.yaml

run-dump: link-patch
	kubectl kustomize .

dump: get-secrets-from-vault run-dump clean-secrets

run-apply: link-patch
	kubectl apply -k .

apply: get-secrets-from-vault run-apply clean-secrets
