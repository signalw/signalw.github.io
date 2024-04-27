+++
title = 'GitOps (study notes)'
date = 2021-03-18T18:58:00-04:00
comments = true
math = false
+++

## What is GitOps?

First coined by the team at WeaveWorks who formalized several of their own daily patterns and processes to empower high velocity cloud native application deployments into Kubernetes clusters.

## GitOps Benefits (for Kubernetes Deployments):
* Enhanced developer productivity - the only access the developer ever needed to get an application into the cluster was write access into the Git repo.
* Enhanced operations - Kubernetes can be configured with Flux Operator to periodically look inward into Git and have the cluster continually converge on the declared state held within Git.
* Better security - Since Flux Operator is deployed inside the cluster there is no longer any need to manage and maintain a set of credentials externally to the cluster itself.
* Better compliance and auditing capabilities - Since all of the cluster state is declared and stored within Git, then its very simple to audit and report on the known cluster state.
* Faster restoration - In the rare event of a cluster failure you can confidently restore or rebuild your cluster to a known state very quickly.
* Simplified rollbacks - Rolling back to a previous version of the cluster state is both simple and quick since all cluster state has been versioned and stored within Git.

## Sample Tools:
Helm, Tiller, Flux Operator, Github, Docker Hub
