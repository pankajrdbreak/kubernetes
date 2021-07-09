# ConfigMap

It is not safe to create container image with all server configuration files as we may upload docker images on remote. That's why we use configmaps to pass the config files to the container after creating it.

Steps to pass config to container using ConfigMaps :

