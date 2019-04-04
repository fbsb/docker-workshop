IMAGE:=webapp

build: Dockerfile
	docker build -t $(IMAGE) .

run: build
	docker run --rm -p 80:8080 $(IMAGE)