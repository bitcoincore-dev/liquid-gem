DOCKER=docker
TAG=liquid-gem

#/docs is symlinked to the mirror docs.blockstream.com
#so we can comply to gh-pages
PROJECT_NAME := $(notdir $(PWD))
SITE         := $(PWD)/docs.blockstream.com

# USAGE
# make user=your-gh-username
ifeq ($(user),)
USER := blockstream
export USER
else
USER = $(user)
export USER
endif
# make org=your-gh-org
ifeq ($(org),)
ORG := blockstream
export ORG
else
ORG = $(org)
export ORG
endif

##GITHUB CONFIG
#GITHUB_USER_NAME=$(git config user.name)
#GITHUB_USER_EMAIL=$(git config user.email)

VERSION := 0.0.1
export VERSION
export PROJECT_NAME
export USER
#export GITHUB_USER_NAME
#export GITHUB_USER_EMAIL


.PHONY:
	image image_alpine mirror-docsv get-tools server shell

# Build the docker image
image: build-docs
	${DOCKER} build -t ${TAG} .
image_alpine: build-docs
	${DOCKER} build -t ${TAG} . -f Dockerfile.alpine

rebuild-image: build-docs
	${DOCKER} build -t ${TAG} --no-cache .
rebuild-image_alpine: build-docs
	${DOCKER} build -t ${TAG} --no-=cache . -f Dockerfile.alpine

depends:
	bash -c "pip3 install sphinx && pip3 install sphinx_rtd_theme"

#mirror-docs:
#	bash -c "wget -m https://docs.blockstream.com"

build-docs:
	bash -c "rm -rf docs.blockstream.com"
	bash -c "git clone https://github.com/Blockstream/docs.git docs.blockstream.com"
	bash -c "cd docs.blockstream.com && make clean && make html && cd .."

get-tools:
	#TODO addmore demos and tools
	#bash -c "git clone https://github.com/Blockstream/ln-wordpress-store.git"

# Produce a bash shell
shell:
	${DOCKER} run --rm -it \
		-p 4000:4000 \
		-u `id -u`:`id -g` \
		-v ${PWD}:/src/gh/pages-gem \
		${TAG} \
		/bin/bash

server: image
	test -d "${SITE}" || \
		(echo -E "specify SITE e.g.: SITE=/path/to/site make server"; exit 1) && \
	${DOCKER} run --rm -it \
		-p 4000:4000 \
		-u `id -u`:`id -g` \
		-v ${PWD}:/src/gh/pages-gem \
		-v `realpath ${SITE}`:/src/site \
		-w /src/site \
		${TAG}

#######################
package-all: image
	bash -c 'cat ~/GH_TOKEN.txt | docker login docker.pkg.github.com -u $(USER) --password-stdin'
	bash -c 'docker tag $(PROJECT_NAME) docker.pkg.github.com/$(ORg)/liquid-gem/$(notdir $(PWD)):$(VERSION)'
	bash -c 'docker push docker.pkg.github.com/$(ORG)/liquid-gem/$(notdir $(PWD)):$(VERSION)'
#######################
git-fetch-branches:
	bash -c "./bin/git-fetch-branches"
#######################
docker-clean:
	@docker-compose -p $(PROJECT_NAME) down --remove-orphans --rmi all 2>/dev/null \
	&& echo 'Image(s) for "$(PROJECT_NAME)" removed.' \
	|| echo 'Image(s) for "$(PROJECT_NAME)" already removed.'
#######################
docker-prune:
	docker system prune -af
#######################
-include Makefile
-include ln-wordpress-store/Makefile

