---
title: Kubernetes Port Forward
description: Quick reference for kubectl port-forward command syntax
tags:
  - notes
date: "2026-04-06T18:59:45+05:30"
lastmod: "2026-04-06T18:59:45+05:30"
draft: false
---
# Kubernetes Port Forward

Use this command:

```
kubectl port-forward svc/temporal-web 8080:8080
```

The first port is the localhost port, the second the service port.