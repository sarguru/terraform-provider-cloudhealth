.PHONY: itest_% clean shell
PROJECT = terraform-provider-cloudhealth

VERSION = 2.0
ITERATION = yelp0
TFVERSION = 0.10
ARCH := $(shell facter architecture)

PACKAGE_NAME := $(PROJECT)-${TFVERSION}_$(VERSION)-$(ITERATION)_$(ARCH).deb
PACKAGE_FILE := dist/$(PACKAGE_NAME)

DOCKER_TAG = $(PROJECT)_$(shell date +%s)

CLEAN_CONTAINER := [ -e .docker_container_id ] && docker rm --force $$(cat .docker_container_id) || true; rm -f .docker_container_id

itest_%: $(PACKAGE_FILE)
	docker run --rm -v $(CURDIR)/../dist:/dist:ro -v $(CURDIR)/itest.sh:/itest.sh:ro docker-dev.yelpcorp.com/$*_yelp:latest bash /itest.sh /$(PACKAGE_FILE) $(TFVERSION)

$(PACKAGE_FILE): .docker_container_id
	docker cp $$(cat .docker_container_id):/$(PACKAGE_FILE) ../dist/

.docker_container_id: .docker_image_id
	docker run --rm=false \
		-v $(CURDIR)/..:/go/src/github.com/Yelp/terraform-provider-cloudhealth:ro \
		-v $(CURDIR)/build.sh:/build.sh:ro \
		--cidfile=$(CURDIR)/.docker_container_id \
		$$(cat .docker_image_id) \
		bash /build.sh $(PROJECT) $(VERSION) $(ITERATION) $(TFVERSION)|| \
	(retval=$$?; $(CLEAN_CONTAINER); exit $$retval; )

.docker_image_id: Dockerfile
	docker build -t $(DOCKER_TAG) .
	docker inspect -f '{{ .Id }}' $(DOCKER_TAG) > .docker_image_id

clean:
	$(CLEAN_CONTAINER)
	rm .docker_image_id
	rm -rf ../dist

shell: .docker_image_id
	docker run --rm -t -i $$(cat .docker_image_id) /bin/bash
