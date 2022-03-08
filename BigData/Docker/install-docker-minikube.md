# Install hyperkit and minikube
```shell
brew install hyperkit
brew install minikube
```

# Install Docker CLI
```shell
brew install docker
brew install docker-compose
```

# Start minikube
```shell
minikube start
```

# Tell Docker CLI to talk to minikube's VM
```shell
eval $(minikube docker-env)
```

# Save IP to a hostname
```shell
echo "`minikube ip` docker.local" | sudo tee -a /etc/hosts > /dev/null
```

# Test
```shell
docker run hello-world
```