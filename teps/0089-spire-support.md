---
status: proposed
title: SPIRE Support
creation-date: '2021-10-04'
last-updated: '2021-10-04'
authors:
- '@priyawadhwa'
---

# TEP-0089: SPIRE Support

<!--
A table of contents is helpful for quickly jumping to sections of a TEP and for
highlighting any additional information provided beyond the standard TEP
template.

Ensure the TOC is wrapped with
  <code>&lt;!-- toc --&rt;&lt;!-- /toc --&rt;</code>
tags, and then generate with `hack/update-toc.sh`.
-->

<!-- toc -->
- [Summary](#summary)
- [Motivation](#motivation)
  - [Goals](#goals)
- [Proposal](#proposal)
  - [Risks and Mitigations](#risks-and-mitigations)
- [Design Details](#design-details)
  - [Requesting an SVID/Signature from the Controller Image](#requesting-an-svidsignature-from-the-controller-image)
  - [Enabling SPIRE in Tekton](#enabling-spire-in-tekton)
- [Test Plan](#test-plan)
- [Alternatives](#alternatives)
- [Infrastructure Needed (optional)](#infrastructure-needed-optional)
- [References (optional)](#references-optional)
<!-- /toc -->

## Summary

A zero-trust supply chain for Tekton requires collaboration between Tekton Pipelines, Tekton Chains, and [SPIRE](https://spiffe.io/).
A detailed explanation of why this is necessary can be found in [Zero Trust Supply Chains](https://docs.google.com/document/d/1CRvANkYu0fxJjEZO4KTyyk_1uZm2Q9Nr0ibxplakODg/edit?resourcekey=0-nGnWnCni8IpiXim-WreYMg#heading=h.fyy27kd27z1r) by dlorenc@.

To summarize, we can't rely on only Tekton Pipelines and Tekton Chains to secure our pipeline because Chains relies on Pipelines to tell it what the given state of a TaskRun is.
For example, Chains expects Pipelines to tell it when the TaskRun is complete, what happened in the TaskRun, and what artifacts need to be signed.
This is insecure because any cluster administrator can edit a TaskRun to say anything they want, and Chains will just believe it.
Chains has no way to verify that the TaskRun it gets was actually run as it says it was.

This is where SPIRE comes in! 
SPIRE can be used to attest workloads on a cluster, and in this case it will be used to attest TaskRuns.
Together, Tekton Pipelines, Tekton Chains, and SPIRE can provide a secure zero-trust supply chain on Kubernetes.


## Motivation

Security! =)

We'll also need this feature for Tekton to achieve [SLSA 3](https://slsa.dev/levels), which requires Non-falsifiable Provenance.

### Goals

* Pipelines should attest a TaskRun against SPIRE before it completes
* Pipelines should store info necessary for verification somewhere
* Chains should be able to verify a TaskRun wasn't modified before signing


## Proposal
The basic design looks like this:

1. User installs SPIRE capabilities into Tekton (details discussed below)
1. Pipelines controller creates a Pod for a new TaskRun per usual
1. Pipelines requests a signature from SPIRE over the TaskRun yaml
1. SPIRE returns signature and SVID x509 Certificate for the TaskRun
1. Pipelines stores the signature and SVID on the TaskRun as an annotation


Then, Chains gets the TaskRun.
Chains will:
1. Verify the signature against the SPIRE x509 certificate
1. If verification fails, Chains will mark the TaskRun as "failed" and move on
1. Otherwise, continue signing stuff! 


<!--
This is where we get down to the specifics of what the proposal actually is.
This should have enough detail that reviewers can understand exactly what
you're proposing, but should not include things like API designs or
implementation.  The "Design Details" section below is for the real
nitty-gritty.
-->

### Risks and Mitigations

Annotations might not be big enough to store the info we want to send.

## Design Details

### Requesting an SVID/Signature from the Controller Image

We could add a flag to the controller image, `--validate-spire=true`, which would tell the controller image to request an SVID & signature from SPIRE.

### Enabling SPIRE in Tekton
SPIRE runs as a Unix domain socket on the k8s node.
For Tekton to interact with SPIRE, we would need to make the following changes to the controller deployment:
1. Mount in the SPIRE socket as a volume of type `hostPath`
1. Pass in the `--validate-spire=true` flag to the controller image

This volume mount would look something like this on the Tekton controller:

```
containers:
- name: tekton-pipelines-controller
  volumeMounts:
  - name: spire
    mountPath: /run/spire/sockets/agent.sock
volumes:
- name: spire
  hostPath:
    path: /run/spire/sockets/agent.sock
```

We could write a TaskRun that would be responsible for making the above changes to the controller.
The code for this TaskRun could live in the Chains repo, and be released as part of Chains releases.

## Test Plan
Tests should cover the entire flow mentioned above including:
1. Enabling SPIRE in Tekton with the installation `TaskRun`
1. Requesting an SVID & signature for a TaskRun in the controller
1. Storing those as annotations on the TaskRun
1. Verification of SPIRE with Chains  



## Alternatives

We could also look into other ways of preventing tampering of Tasks.
As of now I don't know of any other alternatives would work.

## Infrastructure Needed (optional)
We'll probably need a persistent k8s cluster with SPIRE installed to run tests against.



## References (optional)

* [Zero-Trust Supply Chains](https://docs.google.com/document/d/1CRvANkYu0fxJjEZO4KTyyk_1uZm2Q9Nr0ibxplakODg/edit?resourcekey=0-nGnWnCni8IpiXim-WreYMg#heading=h.fyy27kd27z1r) by dlorenc@
