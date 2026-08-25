
.SILENT:
.ONESHELL:
.PHONY: latest specific build

MAKEFLAGS += --no-print-directory

latest:
	set -e;
	# Temporary files and directories
	TMP_RELEASE_OUTPUT=$$(mktemp)

	# Get latest version
	curl -sfL \
		-H "Accept: application/vnd.github+json" \
		-H "X-GitHub-Api-Version: 2022-11-28" \
		https://api.github.com/repos/haproxytech/kubernetes-ingress/releases \
		| jq '.[0]' > $$TMP_RELEASE_OUTPUT
	LATEST_VERSION=$$(jq -r '.tag_name' $$TMP_RELEASE_OUTPUT)

	# Check if latest version already exists
	if [ -d $$LATEST_VERSION ];
	then
		echo "ERR: Version $$LATEST_VERSION already exists, remove the directory to rebuild."
		exit 1
	fi

	# Cleanup
	rm $$TMP_RELEASE_OUTPUT

	# Pass info to build step
	VERSION=$$LATEST_VERSION $(MAKE) build-manifests

specific:
	[ -z "$$VERSION" ] && exit 1
	if [ -d $$VERSION ];
	then
		echo "ERR: Version $$VERSION already exists, remove the directory to rebuild."
		exit 1
	fi
	VERSION=$$VERSION $(MAKE) build-manifests

build-manifests:
	# Temporary files and directories
	TMP_DIR=$$(mktemp -d)

	# Create version location
	mkdir $$VERSION
	echo "Building version: $$VERSION"

	# Pull release from github
	echo "apiVersion: kustomize.config.k8s.io/v1beta1\nkind: Kustomization\n\nresources:\n- https://raw.githubusercontent.com/haproxytech/kubernetes-ingress/refs/tags/$$VERSION/deploy/haproxy-ingress-daemonset.yaml" > $$TMP_DIR/kustomization.yaml
	kustomize build $$TMP_DIR -o $$VERSION/

	# Create kustomization file for version
	echo "apiVersion: kustomize.config.k8s.io/v1beta1\nkind: Kustomization\n\nresources:" > $$VERSION/kustomization.yaml
	for file in $$(ls -1 $$VERSION/ | grep -v 'kustomization.yaml');
	do
		echo "- $$file" >> $$VERSION/kustomization.yaml
	done

	# Cleanup
	[ -z "$$TMP_DIR" ] || rm -rf $$TMP_DIR