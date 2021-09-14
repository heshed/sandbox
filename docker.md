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
docker-machine create --driver "virtualbox" --virtualbox-disk-size "20000" --virtualbox-memory "4096" --virtualbox-cpu-count "2" default
docker-machine restart
eval "$(docker-machine env default)"
docker-machine restart

brew install docker-compose
```

## set docker-compose on clion
![D187374A-97A1-475C-86AF-CDE3D627AE26](https://user-images.githubusercontent.com/1585417/131607809-5eecd696-e2c8-4465-9ed6-2b04162e9cc2.png)

![39734BD9-DD2F-47E8-A51B-EF6568658435](https://user-images.githubusercontent.com/1585417/131607821-a0290ef9-6df3-4ce6-9151-f550d98f10ad.png)

