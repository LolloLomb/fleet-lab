MAKEFILE_DIR:=$(shell dirname $(realpath $(firstword $(MAKEFILE_LIST))))

SOPS_FP:=949A3BFD4CFA21F9585529F687B88DE4CEBC69FC
SECRETS_TEMPLATES_PATH:=$(MAKEFILE_DIR)/secrets-tpl
ENC_SECRETS_PATH:=$(MAKEFILE_DIR)/secrets

secrets: $(ENC_SECRETS_PATH)/secrets.yaml

local-secrets: $(SECRETS_TEMPLATES_PATH)/secrets.yaml

clean:
	rm $(SECRETS_TEMPLATES_PATH)/secrets.yaml \
		$(MAKEFILE_DIR)/test.yaml

$(SECRETS_TEMPLATES_PATH)/secrets.yaml: $(SECRETS_TEMPLATES_PATH)/*.env
	@kustomize build $(SECRETS_TEMPLATES_PATH) > $(SECRETS_TEMPLATES_PATH)/secrets.yaml

$(ENC_SECRETS_PATH)/secrets.yaml: $(SECRETS_TEMPLATES_PATH)/secrets.yaml
	@sops -p $(SOPS_FP) --encrypt $(SECRETS_TEMPLATES_PATH)/secrets.yaml > $(ENC_SECRETS_PATH)/secrets.yaml

build: secrets
	@kustomize build \
		--enable-alpha-plugins \
		--enable-exec \
		$(MAKEFILE_DIR) \
		> $(MAKEFILE_DIR)/test.yaml

validate: build
	@cat $(MAKEFILE_DIR)/test.yaml | kubeconform -verbose -summary

.PHONY: build clean secrets local-secrets