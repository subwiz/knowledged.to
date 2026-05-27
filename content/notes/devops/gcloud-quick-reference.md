---
title: GCloud Quick Reference
description: Quick reference for common gcloud commands including authentication, project setup, and GKE cluster configuration
tags:
  - notes
date: "2026-04-06T19:06:46+05:30"
lastmod: "2026-04-16T18:38:01+05:30"
draft: false
---
# gcloud Commands

## To login

```
gcloud auth login
```

## Application Default Login

```
gcloud auth application-default login
```

## Project Set

```
gcloud config set project buddyhq-prd
```

## GKE cluster context set

`dev`:

```
gcloud container clusters get-credentials web-dev --zone us-central1-f --project buddyhq
```

`prd`:

```
gcloud container clusters get-credentials cntrlplane-gke-sz-prd --zone us-central1-c --project buddyhq-prd
```
