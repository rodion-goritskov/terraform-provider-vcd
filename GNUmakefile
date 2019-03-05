TEST?=$$(go list ./... |grep -v 'vendor')
GOFMT_FILES?=$$(find . -name '*.go' |grep -v vendor)

ifndef TRAVIS_BRANCH
override TRAVIS_BRANCH=snapshot
endif

default: test build package

install:
	mkdir -p $(HOME)/.terraform.d/plugins
	cp $(GOPATH)/bin/terraform-provider-vcd $(HOME)/.terraform.d/plugins/terraform-provider-vcd_v1.0.0_x4

build: fmtcheck
	CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o bin/terraform-provider-vcd-linux-amd64
	CGO_ENABLED=0 GOOS=darwin GOARCH=amd64 go build -o bin/terraform-provider-vcd-darwin-amd64
	CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build -o bin/terraform-provider-vcd-win-amd64.exe

test: fmtcheck
	go test -i $(TEST) || exit 1
	echo $(TEST) | \
		xargs -t -n4 go test $(TESTARGS) -timeout=30s -parallel=4

testacc: fmtcheck
	TF_ACC=1 go test $(TEST) -v $(TESTARGS) -timeout 120m

vet:
	@echo "go vet ."
	@go vet $$(go list ./... | grep -v vendor/) ; if [ $$? -eq 1 ]; then \
		echo ""; \
		echo "Vet found suspicious constructs. Please check the reported constructs"; \
		echo "and fix them if necessary before submitting the code for review."; \
		exit 1; \
	fi

fmt:
	gofmt -w $(GOFMT_FILES)

fmtcheck:
	@sh -c "'$(CURDIR)/scripts/gofmtcheck.sh'"

errcheck:
	@sh -c "'$(CURDIR)/scripts/errcheck.sh'"

vendor-status:
	@govendor status

test-compile:
	@if [ "$(TEST)" = "./..." ]; then \
		echo "ERROR: Set TEST to a specific package. For example,"; \
		echo "  make test-compile TEST=./aws"; \
		exit 1; \
	fi
	go test -c $(TEST) $(TESTARGS)

package:
	mv bin/terraform-provider-vcd-darwin-amd64 terraform-provider-vcd_$(TRAVIS_BRANCH) && tar -cvzf terraform-provider-vcd_$(TRAVIS_BRANCH)_macos.tar.gz terraform-provider-vcd_$(TRAVIS_BRANCH)
	mv bin/terraform-provider-vcd-linux-amd64 terraform-provider-vcd_$(TRAVIS_BRANCH) && tar -cvzf terraform-provider-vcd_$(TRAVIS_BRANCH)_linux.tar.gz terraform-provider-vcd_$(TRAVIS_BRANCH)
	mv bin/terraform-provider-vcd-win-amd64.exe terraform-provider-vcd_$(TRAVIS_BRANCH).exe

.PHONY: build test testacc vet fmt fmtcheck errcheck vendor-status test-compile

