python = python2.7
PINPIP=9.0.3
SHELL=/bin/bash
DOCKER_HELPER := $(shell dirname $(abspath $(lastword $(MAKEFILE_LIST))))/docker_helper.sh

ROLE_VERSION=$(shell cat $(PWD)/../../version.txt)
ROLE_NAME=$(shell . $(DOCKER_HELPER) ; getUpDirLevel3)

REGISTRY_HOST=docker.io
USERNAME=softasap

all: clean_build

clean_build: clean initialize build

build:
	packer build -var 'image_name=$(ROLE_NAME)' container.json

clean:
	@rm -rf roles

initialize:
	@./init_quick.sh

tag:
	docker tag $(ROLE_NAME):latest softasap/sa-container-box-examples:$(ROLE_NAME).packer.$(ROLE_VERSION)
	docker tag $(ROLE_NAME):latest softasap/sa-container-box-examples:$(ROLE_NAME).packer.latest

push:
	docker push softasap/sa-container-box-examples:$(ROLE_NAME).packer.$(ROLE_VERSION)
	docker push softasap/sa-container-box-examples:$(ROLE_NAME).packer.latest

sh-conductor:
	until [ "`/usr/bin/docker inspect -f {{.State.Running}} $(ROLE_NAME)_conductor`"=="true" ]; do \
	sleep 0.1; \
        echo "Waiting for $(ROLE_NAME)..." \
	done; \
	docker exec -it $(ROLE_NAME)_conductor /bin/sh

.PHONY: all clean initialize build run stop

