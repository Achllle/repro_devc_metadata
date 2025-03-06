# devcontainer metadata bug

## repro steps

1. clone this repo, use devcontainer v0.402.0
2. open the img1 directory in vscode
3. 'rebuild and reopen in container'. [img1/.devcontainer/devcontainer.json](img1/.devcontainer/devcontainer.json) mounts `/var/log` to `/var/log`. This builds `base_img`.
4. reopen locally
5. open this folder (repro_devc_metadata) in vscode
6. 'rebuild and reopen in container'. This uses the compose.yaml file which uses the `base_img` built earlier and mounts `/var/log/apt` to `/var/log` instead.

* expected outcome: `/var/log/apt` is mounted into the container
* actual outcome: `/var/log` is mounted into the container

## logs

Run "devcontainer: Show previous log":

```yaml
[...]
services:
  service1:
    command:
      - sleep
      - infinity
    image: base_img
    networks:
      default: null
    volumes:
      - type: bind
        source: /var/log/apt
        target: /var/log
networks:
  default:
    name: repro_devc_metadata_default

[...]

Docker Compose override file for creating container:
services:
  'service1':
    entrypoint: ["/bin/sh", "-c", "echo Container started\n
trap \"exit 0\" 15\n
\n
exec \"$$@\"\n
while sleep 1 & wait $$!; do :; done", "-"]
    labels:
      - 'devcontainer.local_folder=/home/achille/repro_devc_metadata'
      - 'devcontainer.config_file=/home/achille/repro_devc_metadata/.devcontainer/devcontainer.json'
      - 'devcontainer.metadata=[{"mounts":["source=/var/log,target=/var/log,type=bind"]}]'
    volumes:
      - /var/log:/var/log
      - vscode:/vscode
volumes:
  
  vscode:
    external: true
```

which shows that the compose override file doesn't properly use the compose.yaml mounts and uses the image's metadata

`docker inspect base_img | grep metadata -A 1 -B 1`:

```json
"Labels": {
    "devcontainer.metadata": "{\"mounts\":[\"source=/var/log,target=/var/log,type=bind\"]}"
}
```
