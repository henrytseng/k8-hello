# Kubernetes Overview

Workshop hello world example

Setup Kind
https://github.com/kubernetes-sigs/kind
https://kind.sigs.k8s.io/docs/user/local-registry/

Which context are you configured with?
```
   kubectl config get-contexts
```

Create cluster (context will be `kind-kind`)
```
  kind create cluster --name kind
```

Starting pod
```
   mkdir -p k8-hello/charts
   cd k8-hello
   k run ruby-cmd --rm -i --tty --attach --image=ruby -- /bin/bash
   k get pods
   k describe pod/ruby-cmd
```

Start building a container image with an application, then push it to a registry (https://kind.sigs.k8s.io/docs/user/local-registry/)
```
   gem install rails
   mkdir -p /opt
   cd /opt
   rails new app --skip-javascript

   # locally copy the application out
   k cp ruby-cmd:/opt/app ./hello

   # build up a Dockerfile

   # build and push new container image to your repository
   docker build -t rails-hello:1.0.0 -t localhost:5000/rails-hello:1.0.0 .
   docker push localhost:5000/rails-hello:1.0.0
```

Create database
```
   helm repo add bitnami https://charts.bitnami.com/bitnami
   helm repo update
   helm pull bitnami/postgresql --untardir ./charts/ --untar
   helm install postgres ./charts/postgresql

   k get pods
   k get secrets
   k get persistentvolumes
   k get services
   k describe service/postgres-postgresql
```

Get postgres credentials
```
   export POSTGRES_PASSWORD=$(kubectl get secret --namespace default postgres-postgresql -o jsonpath="{.data.postgresql-password}" | base64 --decode)
```

Setup application deployment and service

Setup application credentials
Gemfile, then `bundle install`
```
   gem 'pg'
```

database.yaml
```
default: &default
  adapter: postgresql
  encoding: unicode
  username: <%= ENV['POSTGRES_USER'] %>
  password: <%= ENV['POSTGRES_PASSWORD'] %>
  pool: 5
  timeout: 5000
  host: <%= ENV['POSTGRES_HOST'] %>
development:
  <<: *default
  database: <%= ENV['POSTGRES_DB'] %>
test:
  <<: *default
  database: <%= ENV['POSTGRES_TEST_DB'] %>
production:
  <<: *default
  database: <%= ENV['POSTGRES_DB'] %>
```

Use deployment template: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#creating-a-deployment

Use service template: https://kubernetes.io/docs/concepts/services-networking/service/#defining-a-service

Run the service
```
   k apply -f deployment.yaml
   k apply -f service.yaml

   # watch it
   for i in `seq 300`; do kubectl get pods; sleep 1; done

   # logs
   k get pods
   k logs hello-d-bd5bf6fff-prphx
```

Port forward to see service locally
```
   k port-forward svc/hello-s 3000:3000

   # localhost
   curl  http://localhost:3000
```

Inspec container
```
   k exec -it hello-d-bd5bf6fff-prphx -- /bin/bash
```

Create a model and database table
```
To get the password for "postgres" run:

    export POSTGRES_PASSWORD=$(kubectl get secret --namespace default postgres-postgresql -o jsonpath="{.data.postgresql-password}" | base64 --decode)

To connect to your database run the following command:

    kubectl run postgres-postgresql-client --rm --tty -i --restart='Never' --namespace default --image docker.io/bitnami/postgresql:11.10.0-debian-10-r60 --env="PGPASSWORD=$POSTGRES_PASSWORD" --command -- psql --host postgres-postgresql -U postgres -d postgres -p 5432
```

Run in psql
```
   create database hello;
   create database hello_test;
   \c hello
   create table users (id int, email varchar);
```

Clean up
```
   k delete -f deployment.yaml
   k delete -f service.yaml
   helm uninstall postgres ./charts/postgresql
```
