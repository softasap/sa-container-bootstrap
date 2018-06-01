python = python2.7
PINPIP=9.0.3
SHELL=/bin/bash
DOCKER_HELPER := $(shell dirname $(abspath $(lastword $(MAKEFILE_LIST))))/docker_helper.sh

ROLE_VERSION=$(shell cat $(PWD)/../../version.txt)
ROLE_NAME=$(shell . $(DOCKER_HELPER) ; getUpDirLevel3)

CONDUCTOR="ansible/container-conductor-alpine-3.5:0.9.2"
CONDUCTOR_ARCH=x86_64-linux-gnu
CONDUCTOR_LD_LIBRARY_PATH="/usr/lib:/usr/lib64:/_usr/lib:/_usr/lib64:/_usr/local/lib:/_usr/lib/$(CONDUCTOR_ARCH):/_usr/local/lib/$(CONDUCTOR_ARCH):/_lib/$(CONDUCTOR_ARCH)"
CONDUCTOR_CPATH="/usr/include:/usr/local/include:/_usr/include:/_usr/local/include"
CONDUCTOR_PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/_usr/sbin:/_usr/bin:/_usr/local/sbin:/_usr/local/bin"


REGISTRY_HOST=docker.io
USERNAME=softasap

all: clean_build

clean_build: clean initialize build

build:
	docker rm -f conductor-$(ROLE_NAME) || true
	docker run --mount 'type=volume,src=conductor_usr,dst=/usr,readonly' --mount 'type=volume,src=conductor_lib,dst=/lib,readonly'  -itd --name "conductor-$(ROLE_NAME)" $(CONDUCTOR) /bin/sh
	packer build -var 'conductor_name=conductor-$(ROLE_NAME)' -var 'image_name=$(ROLE_NAME)' -var 'CONDUCTOR_LD_LIBRARY_PATH=$(CONDUCTOR_LD_LIBRARY_PATH)' -var 'CONDUCTOR_CPATH=$(CONDUCTOR_CPATH)' -var 'CONDUCTOR_PATH=$(CONDUCTOR_PATH)' container.json

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

