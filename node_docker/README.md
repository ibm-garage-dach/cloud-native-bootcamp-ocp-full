# Optional Lab (building docker image from node.js source)


### Build a nodejs container image

Use the Dockerfile to build the image:

```bash
podman build -t node-js-demo:1.0 .
```


### Work with the container image

Run the image:
```bash
podman run --rm -d -p 8000:8080 --name my-container node-js-demo:1.0
curl http://localhost:8000
podman ps
```

Run a bash shell in the container and check that `node_modules` were installed:
```bash
podman exec -it my-container /bin/bash
ls
```

Stop the container:
```bash
podman stop my-container
podman ps
```