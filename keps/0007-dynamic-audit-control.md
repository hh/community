---
kep-number: 7
title: Dynamic Audit Control
authors:
  - "@pbarker"
owning-sig: sig-auth
participating-sigs:
  - sig-cluster-lifecycle
reviewers:
  - TBD
approvers:
  - TBD
editor: TBD
creation-date: 2018-05-18
last-updated: 2018-05-18
status: provisional
see-also:
  - need to find dynamic admission KEP
---

# Dynamic Audit Control

## Table of Contents

* [Dynamic Audit Control](#dynamic-audit-control)
  * [Table of Contents](#table-of-contents)
  * [Summary](#summary)
  * [Motivation](#motivation)
      * [Goals](#goals)
      * [Non-Goals](#non-goals)
  * [Proposal](#proposal)
      * [Dynamic Configuration of Policy](#dynamic-configuration-of-policy)
      * [Dynamic Configuration of Flags](#dynamic-configuration-of-flags)
      * [User Stories](#user-stories)
        * [Story 1](#story-1)
        * [Story 2](#story-2)
      * [Implementation Details/Notes/Constraints](#implementation-detailsnotesconstraints)
      * [Risks and Mitigations](#risks-and-mitigations)
  * [Graduation Criteria](#graduation-criteria)
  * [Implementation History](#implementation-history)
  * [Alternatives](#alternatives)

## Summary

We want to allow the advanced auditing features to be dynamically configured. Following in the same vein as [Dynamic Admission Control](https://kubernetes.io/docs/admin/extensible-admission-controllers/) we would like to provide a means of configuring the auditing features post cluster provisioning.

## Motivation

The advanced auditing features are a powerfull tool, yet difficult to configure. The configuration requires deep insight into the deployment mechanism of choice and often takes many iterations to configure properly requiring a restart of the apiserver each time. Moreover, the ability to install addon tools that configure and enhance audting is hindered by the overhead in configuration. Such tools frequently run on the cluster requiring future knowledge of how to reach them when the cluster is live. These tools could enhance the security and conformance of the cluster and its applications.

### Goals
- Provide an api and set of objects to configure the advanced auditing kube-apiserver webhook flags dynamically
- Allow the current audit policy object to be configured dynamically

### Non-Goals
- Provide a generic interface to configure all kube-apiserver flags

## Proposal

### Dynamic Configuration of Policy
Dynamic configuration of the existing policy object will be provided through use of a shared informer.
```yaml
apiVersion: audit.k8s.io/v1beta1
kind: Policy
omitStages:
  - "RequestReceived"
rules:
  - level: <request level>
    resources:
    - group: <group>
      resources: <resources>
```
In kube-apiserver today this is only accepted as a file, this KEP would allow it to either be configured as a file on the master node or applied to the `kube-system` namespace as an object and be dynamically configured. This will require the audit API to be registered with the apiserver. If both file and object exist the dynamic object will take precedence.

### Dynamic Configuration of Flags
There are a number of configuration options that are currently set through runtime flags. This KEP would provide a new API for the dynamic configuration of the webhook parameters.
```yaml
apiVersion: audit.k8s.io/v1alpha1
kind: AuditConfiguration
metadata:
  name: <name>
webhook:
  mode: <batch or blocking>
  initialBackoff: <10s>
  batch:
    bufferSize: <400>
    maxSize: <10000>
    maxWait: <30s>
    throttleBurst: <15>
    throttleEnabled: <true>
    throttleQPS: <10>
```
The webhook would share the model of validating and mutating admission control plugins, leveraging a shared informer to provide online configuration. If runtime flags are also provided, the dynamic configuration will override any parameters provided there.

### User Stories

#### Story 1
As a cluster admin I will easily be able to enable the interal auditing features of an existing cluster, and tweak the configurations as necessary.

#### Story 2
As a Kubernetes extension developer, I will be able to provide drop in extensions that utilize audit data.

### Implementation Details/Notes/Constraints

A misunderstanding may occur of both flags are defined and an object is provided. This should be explicitly called out in the 
documentation and monitored within the community. If this becomes a point of contention we will need to reevaluate the implementation.

### Risks and Mitigations

This does open up the attack surface of the audit mechanisms. Having them strictly configured through the api server has 
the advantage of limiting the access of those configurations to those that have access to the master node. However, Dynamic Admission has shown that this can be done in a safe manner. 

## Graduation Criteria

Sucess will be determined by stability of the provided mechanisms and ease of understanding for the end user.

* alpha: Policy and api server flags can be dynamically configured, known issues are tested and resolved.
* beta: Mechanisms have been hardened against any known bugs and the process is validated by the community

## Implementation History

- 05/18/2018: initial design

## Alternatives

We could strive for all kube-apiserver flags to be able to be dynamically provisioned in a common way.
