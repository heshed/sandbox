# install docker on mac

## client
- see https://docs.docker.com/engine/install/binaries/#install-client-binaries-on-macos

```console
wget https://download.docker.com/mac/static/stable/x86_64/docker-20.10.8.tgz
tar xvfz docker-20.10.8.tgz
sudo xattr -rc docker
sudo cp docker/docker /usr/local/bin/
```

## server
```console
brew install virtualbox
brew install docker-machine
brew services start docker-machine
docker-machine create --driver virtualbox default
docker-machine restart
eval "$(docker-machine env default)"
docker-machine restart

brew install docker-compose
```

