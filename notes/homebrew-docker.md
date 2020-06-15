---
tags: [ programming ]
---

# Running Docker on macOS without installing the Docker app

```sh
brew install docker docker-machine
brew cask install virtualbox

docker-machine create --driver virtualbox default
brew services start docker-machine

# This needs to be run each time
eval $(docker-machine env default)
```
