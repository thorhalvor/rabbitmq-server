.ONESHELL:# single shell invocation for all lines in the recipe
SHELL = bash# we depend on bash expansion for e.g. queue patterns

.DEFAULT_GOAL = help

ifndef SUITE
override SUITE = "test/oauth/with-uaa"
endif

SELENIUM_VERSION := 104.0

### TARGETS ###

help:
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}'

start-chrome: ## Install Selenium Chrome
	@(echo "Installing Selenium Chrome")
	@(docker network inspect rabbitmq_net >/dev/null 2>&1 || docker network create rabbitmq_net)
	@(docker rm -f selenium 2>/dev/null || echo "selenium was not running")
	@(docker run -d --name selenium --net rabbitmq_net -p 4444:4444 --shm-size=2g selenium/standalone-chrome:$(SELENIUM_VERSION))
	#@(docker run -d --name selenium --net rabbitmq_selenimum_net -p 4444:4444 --shm-size=2g selenium/standalone-chrome)
	@(echo "Selenium Standalone is running on http://localhost:4444/ && http://selenium:4444/")
stop-chrome: ## Stop Selenium Chrome
	@(docker rm -f selenium 2>/dev/null || echo "selenium was not running")

build-test-runner-image: ## Prepare docker image to run tests
	@(docker build -t mocha-test  --target test .)

setup: ## Deploy infrastructure for headless mode to run $(SUITE)
	$(SUITE)/setup.sh
teardown: ## Undeploy infrastructure for headless mode to run
	$(SUITE)/teardown.sh

run-test: ## Run tests using selenium hub (headless mode)
	@(docker network inspect rabbitmq_net >/dev/null 2>&1 || docker network create rabbitmq_net)
	@(echo "Running tests in headless mode for suite $(SUITE) ...")
	@(docker run --rm  --name mocha --net rabbitmq_net \
	  --env RABBITMQ_URL=http://rabbitmq:15672 \
		--env RUN_LOCAL=false \
		-v ${PWD}/screens:/screens \
		-v ${PWD}/test:/code/test mocha-test $(SUITE)
		)
