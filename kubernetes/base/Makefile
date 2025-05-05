# -------------------------------------------------------------------
# Makefile for Kubernetes deployment with conditional butler-password
#
# This Makefile handles:
#   • Fetching secrets from Vault (auto-selects butler-password).
#   • Linking your Kustomize patch file.
#   • Rendering or applying your Kustomize overlay.
#   • Cleaning up local secrets.
#
# Usage:
#   - Default (embargo=true):
#       make apply file=patch-embargo.yaml
#   - Without embargo (embargo=false):
#       make apply file=patch-normal.yaml embargo=false
# -------------------------------------------------------------------


# ----------------------------
# Configuration
# ----------------------------
# Vault path (with the 'secret/' mount prefix)
SECRET_PATH ?= secret/rubin/usdf-consdb-dev

# Kustomize environment subpath
ENV         ?= transformed-efd

# Patch file to link
file        ?=

# Toggle embargo vs main butler password (default: true)
embargo     ?= true

# Select butler-password field based on embargo flag
ifeq ($(embargo),true)
  BUTLER_KEY := butler-embargo-password
else
  BUTLER_KEY := main-butler-db-password
endif

# Other Vault fields to fetch
SECRET_KEYS := aws-access-key-id \
               aws-secret-access-key \
               consdb-url \
               consdb-url-testing \
               s3-profile-embargo \
               usdf-efd-password


# ----------------------------
# Workflow Targets
# ----------------------------

# Fetch secrets from Vault
get-secrets-from-vault:
	@echo "Pulling secrets (embargo=$(embargo))..."
	mkdir -p etc/.secrets/
	set -e; \
	for key in $(SECRET_KEYS); do \
	  vault kv get --field=$$key $(SECRET_PATH)/$(ENV) > etc/.secrets/$$key; \
	done; \
	vault kv get --field=$(BUTLER_KEY) $(SECRET_PATH)/$(ENV) > etc/.secrets/butler-password


# Clean up secrets folder
clean-secrets:
	@echo "Removing secrets..."
	rm -rf etc/.secrets/


# Link the chosen patch file
link-patch:
ifndef file
	$(error Specify FILE, e.g. make apply file=patch.yaml embargo=true)
endif
	@echo "Linking $(file) → patch.yaml"
	ln -sf $(file) patch.yaml


# Dry-run: render Kustomize output
run-dump: link-patch
	@echo "Rendering Kustomize output..."
	kubectl kustomize .


# Combined: fetch secrets + dry-run + clean
dump: get-secrets-from-vault run-dump clean-secrets


# Apply: link patch + apply to cluster
run-apply: link-patch
	@echo "Applying to cluster..."
	kubectl apply -k .


# Combined: fetch secrets + apply + clean
apply: get-secrets-from-vault run-apply clean-secrets
