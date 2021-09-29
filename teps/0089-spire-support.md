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
  - [Non-Goals](#non-goals)
  - [Use Cases (optional)](#use-cases-optional)
- [Requirements](#requirements)
- [Proposal](#proposal)
  - [Notes/Caveats (optional)](#notescaveats-optional)
  - [Risks and Mitigations](#risks-and-mitigations)
  - [User Experience (optional)](#user-experience-optional)
  - [Performance (optional)](#performance-optional)
- [Design Details](#design-details)
- [Test Plan](#test-plan)
- [Design Evaluation](#design-evaluation)
- [Drawbacks](#drawbacks)
- [Alternatives](#alternatives)
- [Infrastructure Needed (optional)](#infrastructure-needed-optional)
- [Upgrade &amp; Migration Strategy (optional)](#upgrade--migration-strategy-optional)
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

[TODO: insert diagram here]

1. Entrypoint image creates a Pod for a new TaskRun
1. Entrypoint image is modified to request a signature from SPIRE over the TaskRun yaml
1. Entrypointer needs to make the following available to Chains:
- The SVID x509 certificate from SPIRE
- The signature from SPIRE

Then, Chains gets the TaskRun.
Chains will:
1. Verify the signature against the SPIRE x509 certificate
1. Continue signing stuff! 


<!--
This is where we get down to the specifics of what the proposal actually is.
This should have enough detail that reviewers can understand exactly what
you're proposing, but should not include things like API designs or
implementation.  The "Design Details" section below is for the real
nitty-gritty.
-->

### Risks and Mitigations

Results might not be big enough to store the info we want to send.

## Design Details



<!--
This section should contain enough information that the specifics of your
change are understandable.  This may include API specs (though not always
required) or even code snippets.  If there's any ambiguity about HOW your
proposal will be implemented, this is the place to discuss them.

If it's helpful to include workflow diagrams or any other related images,
add them under "/teps/images/". It's upto the TEP author to choose the name
of the file, but general guidance is to include at least TEP number in the
file name, for example, "/teps/images/NNNN-workflow.jpg".
-->

## Test Plan

We'll probably need a persistent k8s cluster with SPIRE installed to run tests against.


## Drawbacks

<!--
Why should this TEP _not_ be implemented?
-->

## Alternatives

<!--
What other approaches did you consider and why did you rule them out?  These do
not need to be as detailed as the proposal, but should include enough
information to express the idea and why it was not acceptable.
-->

## Infrastructure Needed (optional)

<!--
Use this section if you need things from the project/SIG.  Examples include a
new subproject, repos requested, github details.  Listing these here allows a
SIG to get the process for these resources started right away.
-->

## Upgrade & Migration Strategy (optional)

<!--
Use this section to detail wether this feature needs an upgrade or
migration strategy. This is especially useful when we modify a
behavior or add a feature that may replace and deprecate a current one.
-->

## References (optional)

* [Zero-Trust Supply Chains](https://docs.google.com/document/d/1CRvANkYu0fxJjEZO4KTyyk_1uZm2Q9Nr0ibxplakODg/edit?resourcekey=0-nGnWnCni8IpiXim-WreYMg#heading=h.fyy27kd27z1r) by dlorenc@
