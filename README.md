
### Caveats:

- This is a managed plugin only, no legacy support.
- In order to properly support versions use `--alias` when installing the plugin.
- This only supports one glusterfs cluster per instance use `--alias` to define separate instances
- The value of `SERVERS` is initially blank it needs `docker plugin glusterfs set SERVERS=store1,store2` if it is set then it will be used for all servers and low level options will not be allowed.  Primarily this is to control what the deployed stacks can perform.
- There is no robust error handling.  So garbage in -> garbage out

## Operating modes

There are three operating modes listed in order of preference.  Each are mutually exclusive and wlil result in an error when performing a `docker volume create` if more than one operating mode is configured.

### Just the name

This is the *recommended* approach for production systems as it will prevent stacks from specifying any random server.  It also prevents the stack configuration file from containing environment specific servers and instead defers that knowledge to the plugin only which is set on the node level.  This relies on `SERVERS` being configured and will use the name as the volume mount set by [`docker plugin set`](https://docs.docker.com/engine/reference/commandline/plugin_set/) e.g.,

    docker plugin set PLUGINALIAS SERVERS=store1,store2

If there is a need to have a different set of servers, a separate plugin alias should be created with a different set of servers.

Example in docker-compose.yml:

    volumes:
      sample:
        driver: glusterfs
        name: "volume/subdir"

The `volumes.x.name` specifies the volume and optionally a subdirectory mount.  The value of `name` will be used as the `--volfile-id` and `--subdir-mount`.  Note that `volumes.x.name` must not start with `/`.

### Specify the servers

This uses the `driver_opts.servers` to define a comma separated list of servers.  The rules for specifying the volume is the same as the previous section.

Example in docker-compose.yml:

    volumes:
      sample:
        driver: glusterfs
        driver_opts:
          servers: store1,store2
        name: "volume/subdir"

The `volumes.x.name` specifies the volume and optionally a subdirectory mount.  The value of `name` will be used as the `--volfile-id` and `--subdir-mount`.  Note that `volumes.x.name` must not start with `/`.  The values above correspond to the following mounting command:

    glusterfs -s store1 -s store2 --volfile-id=volume \
      --subdir-mount=subdir [generated_mount_point]

### Specify the options

This passes the `driver_opts.glusterfsopts` to the `glusterfs` command followed by the generated mount point.  This is the most flexible method and gives full range to the options of the glusterfs FUSE client.

    volumes:
      sample:
        driver: glusterfs
        driver_opts:
          glusterfsopts: "--volfile-server=SERVER --volfile-id=abc --subdir-mount=sub"
        name: "whatever"

The value of `name` will not be used for mounting; the value of `driver_opts.glusterfsopts` is expected to have all the volume connection information.

