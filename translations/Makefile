.PHONY: translate deploy build

translate:
	po4a po4a.cfg

deploy: build
	doas ./publish-private

build: translate
	cd .. && mdbook build
