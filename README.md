
# Citus - DB Setup

**Author:** Matija Gusel  
**Date:** April 2025

## Zahtevane tehnologije
- Minikube
- Helm
- Kubectl
- psql (PostgreSQL client)

## 1. Kubernetes okolje - Minikube

minikube start

## 2. Namestitev Citus preko Helm

helm repo add bitnami https://charts.bitnami.com/bitnami

helm install my-citus bitnami/citus \
  --set coordinator.replicaCount=1 \
  --set worker.replicaCount=2


## 3. Port-forward coordinator 

kubectl port-forward svc/my-citus-coordinator 5432:5432


## 4. Povezava s PostgreSQL

psql -h localhost -U postgres -d postgres

*(Privzeta gesla so običajno prazna.)*

## 5. Baza podatkov in tabela

CREATE DATABASE citus_test;
\c citus_test
CREATE EXTENSION citus;

CREATE TABLE users (
  id serial PRIMARY KEY,
  name text
);

SELECT create_distributed_table('users', 'id');

## 6. Povezava worker nodes

SELECT master_add_node('my-citus-worker-0.my-citus-worker.default.svc.cluster.local', 5432);
SELECT master_add_node('my-citus-worker-1.my-citus-worker.default.svc.cluster.local', 5432);
SELECT rebalance_table_shards();


## 7. Vstavljanje podatkov

INSERT INTO users (name) VALUES ('Matija'), ('Tadej'), ('Maksim'), ('David');

SELECT * FROM users;
