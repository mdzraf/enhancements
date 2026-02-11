<!--
**Note:** When your KEP is complete, all of these comment blocks should be removed.

Follow the guidelines of the [documentation style guide].
In particular, wrap lines to a reasonable length, to make it
easier for reviewers to cite specific portions, and to minimize diff churn on
updates.

[documentation style guide]: https://github.com/kubernetes/community/blob/master/contributors/guide/style-guide.md

To get started with this template:

- [x] **Pick a hosting SIG.**
  Make sure that the problem space is something the SIG is interested in taking
  up. KEPs should not be checked in without a sponsoring SIG.
- [ ] **Create an issue in kubernetes/enhancements**
  When filing an enhancement tracking issue, please make sure to complete all
  fields in that template. One of the fields asks for a link to the KEP. You
  can leave that blank until this KEP is filed, and then go back to the
  enhancement and add the link.
- [ ] **Make a copy of this template directory.**
  Copy this template into the owning SIG's directory and name it
  `NNNN-short-descriptive-title`, where `NNNN` is the issue number (with no
  leading-zero padding) assigned to your enhancement above.
- [x] **Fill out as much of the kep.yaml file as you can.**
  At minimum, you should fill in the "Title", "Authors", "Owning-sig",
  "Status", and date-related fields.
- [ ] **Fill out this file as best you can.**
  At minimum, you should fill in the "Summary" and "Motivation" sections.
  These should be easy if you've preflighted the idea of the KEP with the
  appropriate SIG(s).
- [ ] **Create a PR for this KEP.**
  Assign it to people in the SIG who are sponsoring this process.
- [ ] **Merge early and iterate.**
  Avoid getting hung up on specific details and instead aim to get the goals of
  the KEP clarified and merged quickly. The best way to do this is to just
  start with the high-level sections and fill out details incrementally in
  subsequent PRs.

Just because a KEP is merged does not mean it is complete or approved. Any KEP
marked as `provisional` is a working document and subject to change. You can
denote sections that are under active debate as follows:

```
<<[UNRESOLVED optional short context or usernames ]>>
Stuff that is being argued.
<<[/UNRESOLVED]>>
```

When editing KEPS, aim for tightly-scoped, single-topic PRs to keep discussions
focused. If you disagree with what is already in a document, open a new PR
with suggested changes.

One KEP corresponds to one "feature" or "enhancement" for its whole lifecycle.
You do not need a new KEP to move from beta to GA, for example. If
new details emerge that belong in the KEP, edit the KEP. Once a feature has become
"implemented", major changes should get new KEPs.

The canonical place for the latest set of instructions (and the likely source
of this file) is [here](/keps/NNNN-kep-template/README.md).

**Note:** Any PRs to move a KEP to `implementable`, or significant changes once
it is marked `implementable`, must be approved by each of the KEP approvers.
If none of those approvers are still appropriate, then changes to that list
should be approved by the remaining approvers and/or the owning SIG (or
SIG Architecture for cross-cutting KEPs).
-->
# KEP-draft-20260106-topology-for-volume-snapshot: Topology For Volume Snapshot

<!--
This is the title of your KEP. Keep it short, simple, and descriptive. A good
title can help communicate what the KEP is and should be considered as part of
any review.
-->

<!--
A table of contents is helpful for quickly jumping to sections of a KEP and for
highlighting any additional information provided beyond the standard KEP
template.

Ensure the TOC is wrapped with
  <code>&lt;!-- toc --&rt;&lt;!-- /toc --&rt;</code>
tags, and then generate with `hack/update-toc.sh`.
-->

<!-- toc -->
- [Release Signoff Checklist](#release-signoff-checklist)
- [Summary](#summary)
- [Motivation](#motivation)
  - [Goals](#goals)
  - [Non-Goals](#non-goals)
- [Proposal](#proposal)
  - [User Stories (Optional)](#user-stories-optional)
    - [Story 1](#story-1)
    - [Story 2](#story-2)
  - [Notes/Constraints/Caveats (Optional)](#notesconstraintscaveats-optional)
  - [Risks and Mitigations](#risks-and-mitigations)
- [Design Details](#design-details)
  - [Feature Gate](#feature-gate)
  - [CSI Spec Changes](#csi-spec-changes)
  - [Volume Snapshot Components Changes](#volume-snapshot-components-changes)
    - [Volume Snapshot CRD](#volume-snapshot-crd)
    - [VolumeSnapshotClass CRD](#volumesnapshotclass-crd)
    - [Snapshotter (pkg/snapshotter)](#snapshotter-pkgsnapshotter)
    - [CSI Handler (pkg/sidecar-controller/csi_handler.go)](#csi-handler-pkgsidecar-controllercsi_handlergo)
    - [CSI Snapshotter (pkg/sidecar-controller/snapshot_controller.go)](#csi-snapshotter-pkgsidecar-controllersnapshot_controllergo)
  - [Error Handling](#error-handling)
  - [Test Plan](#test-plan)
      - [Prerequisite testing updates](#prerequisite-testing-updates)
      - [Unit tests](#unit-tests)
      - [Integration tests](#integration-tests)
      - [e2e tests](#e2e-tests)
  - [Graduation Criteria](#graduation-criteria)
    - [Alpha](#alpha)
    - [Beta](#beta)
    - [GA](#ga)
    - [Deprecation](#deprecation)
  - [Upgrade / Downgrade Strategy](#upgrade--downgrade-strategy)
  - [Version Skew Strategy](#version-skew-strategy)
- [Production Readiness Review Questionnaire](#production-readiness-review-questionnaire)
  - [Feature Enablement and Rollback](#feature-enablement-and-rollback)
  - [Rollout, Upgrade and Rollback Planning](#rollout-upgrade-and-rollback-planning)
  - [Monitoring Requirements](#monitoring-requirements)
  - [Dependencies](#dependencies)
  - [Scalability](#scalability)
  - [Troubleshooting](#troubleshooting)
- [Implementation History](#implementation-history)
- [Drawbacks](#drawbacks)
- [Alternatives](#alternatives)
  - [Inherit topology from the source PersistentVolume](#inherit-topology-from-the-source-persistentvolume)
- [Infrastructure Needed (Optional)](#infrastructure-needed-optional)
<!-- /toc -->

## Release Signoff Checklist

<!--
**ACTION REQUIRED:** In order to merge code into a release, there must be an
issue in [kubernetes/enhancements] referencing this KEP and targeting a release
milestone **before the [Enhancement Freeze](https://git.k8s.io/sig-release/releases)
of the targeted release**.

For enhancements that make changes to code or processes/procedures in core
Kubernetes—i.e., [kubernetes/kubernetes], we require the following Release
Signoff checklist to be completed.

Check these off as they are completed for the Release Team to track. These
checklist items _must_ be updated for the enhancement to be released.
-->

Items marked with (R) are required *prior to targeting to a milestone / release*.

- [ ] (R) Enhancement issue in release milestone, which links to KEP dir in [kubernetes/enhancements] (not the initial KEP PR)
- [ ] (R) KEP approvers have approved the KEP status as `implementable`
- [x] (R) Design details are appropriately documented
- [ ] (R) Test plan is in place, giving consideration to SIG Architecture and SIG Testing input (including test refactors)
  - [ ] e2e Tests for all Beta API Operations (endpoints)
  - [ ] (R) Ensure GA e2e tests meet requirements for [Conformance Tests](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/conformance-tests.md)
  - [ ] (R) Minimum Two Week Window for GA e2e tests to prove flake free
- [ ] (R) Graduation criteria is in place
  - [ ] (R) [all GA Endpoints](https://github.com/kubernetes/community/pull/1806) must be hit by [Conformance Tests](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/conformance-tests.md) within one minor version of promotion to GA
- [ ] (R) Production readiness review completed
- [ ] (R) Production readiness review approved
- [ ] "Implementation History" section is up-to-date for milestone
- [ ] User-facing documentation has been created in [kubernetes/website], for publication to [kubernetes.io]
- [ ] Supporting documentation—e.g., additional design documents, links to mailing list discussions/SIG meetings, relevant PRs/issues, release notes

<!--
**Note:** This checklist is iterative and should be reviewed and updated every time this enhancement is being considered for a milestone.
-->

[kubernetes.io]: https://kubernetes.io/
[kubernetes/enhancements]: https://git.k8s.io/enhancements
[kubernetes/kubernetes]: https://git.k8s.io/kubernetes
[kubernetes/website]: https://git.k8s.io/website

## Summary

<!--
This section is incredibly important for producing high-quality, user-focused
documentation such as release notes or a development roadmap. It should be
possible to collect this information before implementation begins, in order to
avoid requiring implementors to split their attention between writing release
notes and implementing the feature itself. KEP editors and SIG Docs
should help to ensure that the tone and content of the `Summary` section is
useful for a wide audience.

A good summary is probably at least a paragraph in length.
-->

This KEP proposes adding topology information to volume snapshots in Kubernetes by extending the `VolumeSnapshotContentSpec` Object with an `AccessibleTopology` field. Currently, volume snapshots lack any topology information, creating a foundational gap that prevents the implementation of topology-aware snapshot features such as cross-region snapshot cloning. The enhancement will add topology information to volume snapshot contents by having it as a return value in the `CreateSnapshotResponse`, which will then be stored in the resulting `VolumeSnapshotContent`.

## Motivation

This enhancement emerged from discussions in the 11/27/2025 Kubernetes CSI Implementation Meeting [(see notes)](https://docs.google.com/document/d/1_hvq3nleqQEYatH9V_Gfep39jMzaFJRSN2ioA0PFq-Q/edit?tab=t.0#heading=h.n1nyj67yzejb) around implementing cross-region snapshot cloning capabilities. During the discussions, it became clear that topology support in snapshots is a prerequisite for a feature of this kind. Rather than implementing snapshot cloning without the proper topology foundations, it was decided to first establish topology for snapshots, creating a solid base for future enhancements.

A PR for adding topology for snaphots to the CSI Spec was previously in review ([Add Topology for Snapshot #274](https://github.com/container-storage-interface/spec/pull/274)). At the time there did not seem to be any consideration for features such as cross region snapshot cloning and as such there was no valid use case(s) to justify implementing this so further development/review was halted.

While CSI volumes have comprehensive topology support, snapshots operate without any topology context. The lack of topology information in snapshots creates a foundational gap that this KEP addresses by establishing topology awareness as a core capability of the volume snapshot.

### Goals

- Add topology information to `VolumeSnapshotContent` objects.
- Extend CSI spec to support optional topology fields in `CreateSnapshotRequest` and `CreateSnapshotResponse`.

### Non-Goals

- Implementing cross region/az snapshot cloning functionality.
- Add ability to modify any existing volume snapshot fields.

## Proposal

<!--
This is where we get down to the specifics of what the proposal actually is.
This should have enough detail that reviewers can understand exactly what
you're proposing, but should not include things like API designs or
implementation. What is the desired outcome and how do we measure success?.
The "Design Details" section below is for the real
nitty-gritty.
-->

### User Stories (Optional)

<!--
Detail the things that people will be able to do if this KEP is implemented.
Include as much detail as possible so that people can understand the "how" of
the system. The goal here is to make this feel real for users without getting
bogged down.
-->

#### Story 1

As a contributor to the Kubernetes project, I want to propose/implement snapshot features that would require topology information to be available. 

#### Story 2

As a cluster operator, I want to see topology information for each volume snapshot, so that I can audit snapshot distribution, understand disaster recovery exposure, and identify zone-specific risks in my backup strategy.

### Notes/Constraints/Caveats (Optional)

<!--
What are the caveats to the proposal?
What are some important details that didn't come across above?
Go in to as much detail as necessary here.
This might be a good place to talk about core concepts and how they relate.
-->

This KEP does not introduce any topology validation or intersection analysis between candidate nodes, snapshots, and volume accessibility requirements. The topology information added to snapshots is purely informational and will not be used by the scheduler or provisioner to validate topology compatibility. For example, if a volume is being restored from a snapshot with incompatible topology, the progra will not return a topology-specific error. Instead, the existing behavior will be maintained where the operation fails with the same errors returned today (e.g., snapshot not found or provisioning failure).

### Risks and Mitigations

<!--
What are the risks of this proposal, and how do we mitigate? Think broadly.
For example, consider both security and how this will impact the larger
Kubernetes ecosystem.

How will security be reviewed, and by whom?

How will UX be reviewed, and by whom?

Consider including folks who also work outside the SIG or subproject.
-->

## Design Details

<!--
This section should contain enough information that the specifics of your
change are understandable. This may include API specs (though not always
required) or even code snippets. If there's any ambiguity about HOW your
proposal will be implemented, this is the place to discuss them.
-->

### Feature Gate

A new feature gate,`VolumeSnapshotTopology`, will be introduced to control the functionality implemented by this KEP. When the feature gate is disabled the `AccessibleTopology` field in the `VolumeSnapshotContent.spec` will not be filled. 

### CSI Spec Changes

```protobuf
message CreateSnapshotRequest {
  ... Existing CreateSnapshotRequest fields
  // Specifies where (regions, zones, racks, etc.) the provisioned
  // snapshot MUST be accessible from.
  // An SP SHALL advertise the requirements for topological
  // accessibility information in documentation. COs SHALL only specify
  // topological accessibility information supported by the SP.
  // This field is OPTIONAL.
  TopologyRequirement accessibility_requirements = 5;
}
```
```protobuf
message Snapshot {
  ... Existing fields
  // Specifies where (regions, zones, racks, etc.) the provisioned
  // snapshot is accessible from.
  // An SP MAY specify multiple topologies to indicate the snapshot is
  // accessible from multiple locations.
  // This field is OPTIONAL. If it is not specified, the CO MAY assume
  // the snapshot is equally accessible in the cluster and
  // MAY create volumes referencing the snapshot as a sources.
  repeated Topology accessible_topology = 7;
}
```

### Volume Snapshot Components Changes

There will be changes required in the Volume Snapshot CRDs and the CSI snapshotter sidecar controller.

#### Volume Snapshot CRD

Add topology field to `VolumeSnapshotContentSpec` object:

```go
type VolumeSnapshotContentSpec struct {
	// ... existing fields ...

	// accessibleTopology represents where (regions, zones, racks, etc.) the snapshot
	// is accessible from. This information is returned by the CSI driver in the
	// CreateSnapshotResponse and stored here.
	// This field is kept separate from other source-related fields to allow for
	// future enhancements where topology may be sourced from other places
	// This field is immutable.
	// +optional
	// +kubebuilder:validation:XValidation:rule="self == oldSelf",message="accessibleTopology is immutable"
	AccessibleTopology map[string]string `json:"accessibleTopology,omitempty" protobuf:"bytes,7,rep,name=accessibleTopology"`
}
```

Example VolumeSnapshotContent with topology:

```yaml
Name:         snapcontent-123-456-789
Namespace:    
Labels:       <none>
Annotations:  <none>
API Version:  snapshot.storage.k8s.io/v1
Kind:         VolumeSnapshotContent
Metadata:
  Creation Timestamp:  2030-02-12T00:06:57Z
  Finalizers:
    snapshot.storage.kubernetes.io/volumesnapshotcontent-bound-protection
  Generation:        1
  Resource Version:  345-678-912
Spec:
  Deletion Policy:  Delete
  Driver:           ebs.csi.aws.com
  Source:
    Volume Handle:             vol-123456789
  Source Volume Mode:          Filesystem
  Volume Snapshot Class Name:  csi-aws-vsc
  Volume Snapshot Ref:
    API Version:       snapshot.storage.k8s.io/v1
    Kind:              VolumeSnapshot
    Name:              ebs-volume-snapshot
    Namespace:         default
    Resource Version:  111691
    UID:               123-456-789
  # Topology field
  Accessible Topology:
    topology.kubernetes.io/region: us-west-2
Status:
  Creation Time:    1234567890000000
  Ready To Use:     true
  Restore Size:     4294967296
  Snapshot Handle:  snap-123456789
Events:             <none>
```

#### VolumeSnapshotClass CRD

Add an optional topology requirements field to `VolumeSnapshotClass` to allow admins to specify their desired snapshot topology requirements. This maps to the `AccessibilityRequirements` field in the CSI `CreateSnapshotRequest`.

```go
type VolumeSnapshotClass struct {
	// ... existing fields ...

	// topologyRequirements specifies where (regions, zones, racks, etc.) snapshots
	// created using this class should be accessible from. This is passed to the CSI
	// driver as AccessibilityRequirements in the CreateSnapshotRequest.
	// If not specified, the CSI driver may determine another method to declare
  // topology requirements.
	// +optional
	TopologyRequirements map[string]string `json:"topologyRequirements,omitempty" protobuf:"bytes,5,rep,name=topologyRequirements"`
}
```

Example VolumeSnapshotClass with topology requirements:

```yaml
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshotClass
metadata:
  name: csi-aws-vsc
driver: ebs.csi.aws.com
deletionPolicy: Delete
topologyRequirements:
  topology.kubernetes.io/region: us-west-2
```

When present, the sidecar controller reads `TopologyRequirements` from the class and converts it to a `TopologyRequirement` for the CSI `CreateSnapshotRequest.AccessibilityRequirements`. If not specified, the request is sent without topology requirements and the SP determines placement.

#### Snapshotter (pkg/snapshotter)

Update the `Snapshotter` interface and implementation to accept topology requirements as input and return topology from the CSI driver response.

```golang
// Updated Snapshotter interface
type Snapshotter interface {
	CreateSnapshot(ctx context.Context, snapshotName string, volumeHandle string, parameters map[string]string, snapshotterCredentials map[string]string, /* new parameter */ accessibilityRequirements *csi.TopologyRequirement) (driverName string, snapshotId string, timestamp time.Time, size int64, readyToUse bool, /* new return value */ accessibleTopology []*csi.Topology, err error)
	// ... existing methods unchanged ...
}

func (s *snapshot) CreateSnapshot(ctx context.Context, snapshotName string, volumeHandle string, parameters map[string]string, snapshotterCredentials map[string]string, /* new parameter */ accessibilityRequirements *csi.TopologyRequirement) (string, string, time.Time, int64, bool, /* new return value */ []*csi.Topology, error) {
	klog.V(5).Infof("CSI CreateSnapshot: %s", snapshotName)
	client := csi.NewControllerClient(s.conn)

	// ... Existing Code ...

	req := csi.CreateSnapshotRequest{
		SourceVolumeId:            volumeHandle,
		Name:                      snapshotName,
		Parameters:                parameters,
		Secrets:                   snapshotterCredentials,
		AccessibilityRequirements: accessibilityRequirements, // optional new field.
	}

	rsp, err := client.CreateSnapshot(ctx, &req)
	if err != nil {
		return "", "", time.Time{}, 0, false, nil, err
	}

	// ... Existing Code ...
	creationTime := rsp.Snapshot.CreationTime.AsTime()
	return driverName, rsp.Snapshot.SnapshotId, creationTime, rsp.Snapshot.SizeBytes, rsp.Snapshot.ReadyToUse, /* new return value containing topology from CreateSnapshotResponse*/ rsp.Snapshot.AccessibleTopology, nil
}
```

#### CSI Handler (pkg/sidecar-controller/csi_handler.go)

Update Handler interface and implementation to accept topology requirements and return topology.

```golang
type Handler interface {
	CreateSnapshot(content *crdv1.VolumeSnapshotContent, parameters map[string]string, snapshotterCredentials map[string]string, /* new parameter */ accessibilityRequirements *csi.TopologyRequirement) (string, string, time.Time, int64, bool, /* new return value */ []*csi.Topology, error)
	// ... existing methods unchanged ...
}

func (handler *csiHandler) CreateSnapshot(content *crdv1.VolumeSnapshotContent, parameters map[string]string, snapshotterCredentials map[string]string, /* new parameter */ accessibilityRequirements *csi.TopologyRequirement) (string, string, time.Time, int64, bool, /* new return value */ []*csi.Topology, error) {
	// ... existing validation ...
	return handler.snapshotter.CreateSnapshot(ctx, snapshotName, *content.Spec.Source.VolumeHandle, parameters, snapshotterCredentials, accessibilityRequirements)
}
```

#### CSI Snapshotter (pkg/sidecar-controller/snapshot_controller.go)

Update `createSnapshotWrapper` to read topology requirements from the `VolumeSnapshotClass`, pass them to the CSI handler, and persist the resulting topology to the `VolumeSnapshotContent`.

```golang
func (ctrl *csiSnapshotSideCarController) createSnapshotWrapper(content *crdv1.VolumeSnapshotContent) (*crdv1.VolumeSnapshotContent, error) {
	klog.Infof("createSnapshotWrapper: Creating snapshot for content %s through the plugin ...", content.Name)

	// ... existing code ...

	// Try to build requirrements from VolumeSnapshotClass if feature gate is enabled
	var accessibilityRequirements *csi.TopologyRequirement
	if feature.DefaultFeatureGate.Enabled(features.VolumeSnapshotTopology) && class != nil && len(class.TopologyRequirements) > 0 {
		accessibilityRequirements = &csi.TopologyRequirement{
			Preferred: []*csi.Topology{
				{Segments: class.TopologyRequirements},
			},
		}
	}

	driverName, snapshotID, creationTime, size, readyToUse, accessibleTopology, err := ctrl.handler.CreateSnapshot(content, parameters, snapshotterCredentials, accessibilityRequirements)
	if err != nil {
		// ... existing error handling ...
	}

	// ... existing status update and annotation removal ...

	// Patch VolumeSnapshotContent Spec with topology from CSI driver response
	if feature.DefaultFeatureGate.Enabled(features.VolumeSnapshotTopology) && len(accessibleTopology) > 0 {
		topologyMap := convertCSITopologyToMap(accessibleTopology)
		if len(topologyMap) > 0 {
			patches := []utils.PatchOp{
				{
					Op:    "add",
					Path:  "/spec/accessibleTopology",
					Value: topologyMap,
				},
			}
			content, err = utils.PatchVolumeSnapshotContent(content, patches, ctrl.clientset, "")
			if err != nil {
				return content, fmt.Errorf("failed to patch topology for volume snapshot content %s: %v", content.Name, err)
			}
		}
	}

	return content, nil
}

// convertCSITopologyToMap converts CSI Topology to the map[string]string format
// used by VolumeSnapshotContentSpec.AccessibleTopology.
func convertCSITopologyToMap(csiTopology []*csi.Topology) map[string]string {
	topologyMap := make(map[string]string)
	for _, topo := range csiTopology {
		for key, value := range topo.Segments {
			topologyMap[key] = value
		}
	}
	return topologyMap
}
```

### Error Handling

**Topology requirement cannot be satisfied (e.g., no capacity in requested region/zone):**
The CSI driver returns a gRPC error (typically `ResourceExhausted` or `InvalidArgument`) from `CreateSnapshot`. The sidecar controller handles this like any other `CreateSnapshot` failure. The error is set on `VolumeSnapshotContent.Status.Error`, a warning event is emitted, and the operation is retried. No topology is written to the Spec since the snapshot was not created.

**Illegal topology request:**
This is treated identically to any other creation failure. The snapshot controller does not validate whether the requested topology is compatible with the source volume, that responsibility lies with the SP.

**Topology is valid but temporarily unavailable:**
The SP may return a transient gRPC error. The sidecar controller's existing retry logic handles this. The `CreateSnapshot` call will be retried.

**CSI driver does not return topology in the response:**
If the driver does not populate `AccessibleTopology` in the `CreateSnapshotResponse`, the sidecar controller simply skips the topology patch. The `VolumeSnapshotContent` is created successfully without topology information. This ensures backward compatibility with drivers that do not support topology.

### Test Plan

<!--
**Note:** *Not required until targeted at a release.*
The goal is to ensure that we don't accept enhancements with inadequate testing.

All code is expected to have adequate tests (eventually with coverage
expectations). Please adhere to the [Kubernetes testing guidelines][testing-guidelines]
when drafting this test plan.

[testing-guidelines]: https://git.k8s.io/community/contributors/devel/sig-testing/testing.md
-->

[X] I/we understand the owners of the involved components may require updates to
existing tests to make this code solid enough prior to committing the changes necessary
to implement this enhancement.

##### Prerequisite testing updates

<!--
Based on reviewers feedback describe what additional tests need to be added prior
implementing this enhancement to ensure the enhancements have also solid foundations.
-->

##### Unit tests

<!--
In principle every added code should have complete unit test coverage, so providing
the exact set of tests will not bring additional value.
However, if complete unit test coverage is not possible, explain the reason of it
together with explanation why this is acceptable.
-->

<!--
Additionally, for Alpha try to enumerate the core package you will be touching
to implement this enhancement and provide the current unit coverage for those
in the form of:
- <package>: <date> - <current test coverage>
The data can be easily read from:
https://testgrid.k8s.io/sig-testing-canaries#ci-kubernetes-coverage-unit

This can inform certain test coverage improvements that we want to do before
extending the production code to implement this enhancement.

- `<package>`: `<date>` - `<test coverage>`
-->

Since the e2e framework does not currently support enabling or disabling feature gates, there will be various unit tests that are exercising the `switch` of feature gate itself and handling of relevant data.

##### Integration tests

<!--
Integration tests are contained in https://git.k8s.io/kubernetes/test/integration.
Integration tests allow control of the configuration parameters used to start the binaries under test.
This is different from e2e tests which do not allow configuration of parameters.
Doing this allows testing non-default options and multiple different and potentially conflicting command line options.
For more details, see https://github.com/kubernetes/community/blob/master/contributors/devel/sig-testing/testing-strategy.md

If integration tests are not necessary or useful, explain why.
-->

<!--
This question should be filled when targeting a release.
For Alpha, describe what tests will be added to ensure proper quality of the enhancement.

For Beta and GA, document that tests have been written,
have been executed regularly, and have been stable.
This can be done with:
- permalinks to the GitHub source code
- links to the periodic job (typically https://testgrid.k8s.io/sig-release-master-blocking#integration-master), filtered by the test name
- a search in the Kubernetes bug triage tool (https://storage.googleapis.com/k8s-triage/index.html)
-->

N/A, this enhacement does not introduce configuration parameters or CLI options that are used to start binaries. 

##### e2e tests

<!--
This question should be filled when targeting a release.
For Alpha, describe what tests will be added to ensure proper quality of the enhancement.

For Beta and GA, document that tests have been written,
have been executed regularly, and have been stable.
This can be done with:
- permalinks to the GitHub source code
- links to the periodic job (typically a job owned by the SIG responsible for the feature), filtered by the test name
- a search in the Kubernetes bug triage tool (https://storage.googleapis.com/k8s-triage/index.html)

We expect no non-infra related flakes in the last month as a GA graduation criteria.
If e2e tests are not necessary or useful, explain why.
-->

- Test e2e workflow of having the feature flag enabled and having VolumeSnapshotContents AccessibleTopology fields be populated.

### Graduation Criteria

#### Alpha

- Feature implemented behind a feature flag.
- Initial unit/e2e tests completed and enabled.

#### Beta
- Allowing time for feedback (at least 2 releases between beta and GA).
- All unit tests/integration/e2e tests completed and enabled.
- Validate that the `AccessibleTopology` field is being accurately populated. 
- Validate snapshot-controller behavior with and without volume snapshot topology enabled. 

#### GA
- No bug reports / feedback / improvements to address.

#### Deprecation

<!--
- Announce deprecation and support policy of the existing flag
- Two versions passed since introducing the functionality that deprecates the flag (to address version skew)
- Address feedback on usage/changed behavior, provided on GitHub issues
- Deprecate the flag
-->

No deprecation plan.

### Upgrade / Downgrade Strategy

<!--
If applicable, how will the component be upgraded and downgraded? Make sure
this is in the test plan.

Consider the following in developing an upgrade/downgrade strategy for this
enhancement:
- What changes (in invocations, configurations, API use, etc.) is an existing
  cluster required to make on upgrade, in order to maintain previous behavior?
- What changes (in invocations, configurations, API use, etc.) is an existing
  cluster required to make on upgrade, in order to make use of the enhancement?
-->

- Upgrade Strategy: 
  - Upgrade to the external-snapshotter version that has the udpated controller behavior and CRDs.
  - Make sure to have the `VolumeSnapshotTopology` feature gate enabled.

- Downgrade Strategy: 
  - Disable `VolumeSnapshotTopology` feature gate and restart snapshot-controller.

### Version Skew Strategy

<!--
If applicable, how will the component handle version skew with other
components? What are the guarantees? Make sure this is in the test plan.

Consider the following in developing a version skew strategy for this
enhancement:
- Does this enhancement involve coordinating behavior in the control plane and nodes?
- How does an n-3 kubelet or kube-proxy without this feature available behave when this feature is used?
- How does an n-1 kube-controller-manager or kube-scheduler without this feature available behave when this feature is used?
- Will any other components on the node change? For example, changes to CSI,
  CRI or CNI may require updating that component before the kubelet.
-->

Since the changes of this enhacement are limited to the external-snapshotter components and the topology information is not used/accessed by any other components, a version skew strategy is not applicable.

## Production Readiness Review Questionnaire

<!--

Production readiness reviews are intended to ensure that features merging into
Kubernetes are observable, scalable and supportable; can be safely operated in
production environments, and can be disabled or rolled back in the event they
cause increased failures in production. See more in the PRR KEP at
https://git.k8s.io/enhancements/keps/sig-architecture/1194-prod-readiness.

The production readiness review questionnaire must be completed and approved
for the KEP to move to `implementable` status and be included in the release.

In some cases, the questions below should also have answers in `kep.yaml`. This
is to enable automation to verify the presence of the review, and to reduce review
burden and latency.

The KEP must have a approver from the
[`prod-readiness-approvers`](http://git.k8s.io/enhancements/OWNERS_ALIASES)
team. Please reach out on the
[#prod-readiness](https://kubernetes.slack.com/archives/CPNHUMN74) channel if
you need any help or guidance.
-->

### Feature Enablement and Rollback

<!--
This section must be completed when targeting alpha to a release.
-->

###### How can this feature be enabled / disabled in a live cluster?

<!--
Pick one of these and delete the rest.

Documentation is available on [feature gate lifecycle] and expectations, as
well as the [existing list] of feature gates.

[feature gate lifecycle]: https://git.k8s.io/community/contributors/devel/sig-architecture/feature-gates.md
[existing list]: https://kubernetes.io/docs/reference/command-line-tools-reference/feature-gates/
-->

- [X] Feature gate (also fill in values in `kep.yaml`)
  - Feature gate name: VolumeSnapshotTopology
  - Components depending on the feature gate: snapshot-controller.

###### Does enabling the feature change any default behavior?

<!--
Any change of default behavior may be surprising to users or break existing
automations, so be extremely careful here.
-->

It does not change any default behavior, all of the existing filed in the VSC will still be there. Users will just see the topology information being added to the VSC.

###### Can the feature be disabled once it has been enabled (i.e. can we roll back the enablement)?

<!--
Describe the consequences on existing workloads (e.g., if this is a runtime
feature, can it break the existing applications?).

Feature gates are typically disabled by setting the flag to `false` and
restarting the component. No other changes should be necessary to disable the
feature.

NOTE: Also set `disable-supported` to `true` or `false` in `kep.yaml`.
-->

- Yes the feature can be disabled by turning off the feature gate and restarting the snapshot-controller.

###### What happens if we reenable the feature if it was previously rolled back?

The `VolumeSnapshotContents` will have have topology information from the `CreateSnapshotResponse`. There may be some that do not have it if they were created while the feature was disabled.

###### Are there any tests for feature enablement/disablement?

<!--
The e2e framework does not currently support enabling or disabling feature
gates. However, unit tests in each component dealing with managing data, created
with and without the feature, are necessary. At the very least, think about
conversion tests if API types are being modified.

Additionally, for features that are introducing a new API field, unit tests that
are exercising the `switch` of feature gate itself (what happens if I disable a
feature gate after having objects written with the new field) are also critical.
You can take a look at one potential example of such test in:
https://github.com/kubernetes/kubernetes/pull/97058/files#diff-7826f7adbc1996a05ab52e3f5f02429e94b68ce6bce0dc534d1be636154fded3R246-R282
-->

Yes, there will be a combination of e2e tests and unit tests.

### Rollout, Upgrade and Rollback Planning

<!--
This section must be completed when targeting beta to a release.
-->

###### How can a rollout or rollback fail? Can it impact already running workloads?

<!--
Try to be as paranoid as possible - e.g., what if some components will restart
mid-rollout?

Be sure to consider highly-available clusters, where, for example,
feature flags will be enabled on some API servers and not others during the
rollout. Similarly, consider large clusters and how enablement/disablement
will rollout across nodes.
-->

###### What specific metrics should inform a rollback?

<!--
What signals should users be paying attention to when the feature is young
that might indicate a serious problem?
-->

If the users are seeing failures in the creation of `VolumeSnapshotContents`, that should be a red flag.

###### Were upgrade and rollback tested? Was the upgrade->downgrade->upgrade path tested?

<!--
Describe manual testing that was done and the outcomes.
Longer term, we may want to require automated upgrade/rollback tests, but we
are missing a bunch of machinery and tooling and can't do that now.
-->

###### Is the rollout accompanied by any deprecations and/or removals of features, APIs, fields of API types, flags, etc.?

<!--
Even if applying deprecation policies, they may still surprise some users.
-->

No deprecations and/or removals of features, APIs, fields of API types, or flags as part of this rollout.

### Monitoring Requirements

<!--
This section must be completed when targeting beta to a release.

For GA, this section is required: approvers should be able to confirm the
previous answers based on experience in the field.
-->

###### How can an operator determine if the feature is in use by workloads?

<!--
Ideally, this should be a metric. Operations against the Kubernetes API (e.g.,
checking if there are objects with field X set) may be a last resort. Avoid
logs or events for this purpose.
-->

An operator can determine if the feature is in use by workloads if they see topology in `VolumeSnapshotContent` objects spec.

###### How can someone using this feature know that it is working for their instance?

<!--
For instance, if this is a pod-related feature, it should be possible to determine if the feature is functioning properly
for each individual pod.
Pick one more of these and delete the rest.
Please describe all items visible to end users below with sufficient detail so that they can verify correct enablement
and operation of this feature.
Recall that end users cannot usually observe component logs or access metrics.
-->

- [ ] Events
  - Event Reason: 
- [X] API .spec
  - Field name: AccessibleTopology
- [ ] Other (treat as last resort)
  - Details:

###### What are the reasonable SLOs (Service Level Objectives) for the enhancement?

<!--
This is your opportunity to define what "normal" quality of service looks like
for a feature.

It's impossible to provide comprehensive guidance, but at the very
high level (needs more precise definitions) those may be things like:
  - per-day percentage of API calls finishing with 5XX errors <= 1%
  - 99% percentile over day of absolute value from (job creation time minus expected
    job creation time) for cron job <= 10%
  - 99.9% of /health requests per day finish with 200 code

These goals will help you determine what you need to measure (SLIs) in the next
question.
-->

###### What are the SLIs (Service Level Indicators) an operator can use to determine the health of the service?

<!--
Pick one more of these and delete the rest.
-->

- [ ] Metrics
  - Metric name:
  - [Optional] Aggregation method:
  - Components exposing the metric:
- [ ] Other (treat as last resort)
  - Details:

###### Are there any missing metrics that would be useful to have to improve observability of this feature?

<!--
Describe the metrics themselves and the reasons why they weren't added (e.g., cost,
implementation difficulties, etc.).
-->

### Dependencies

<!--
This section must be completed when targeting beta to a release.
-->

###### Does this feature depend on any specific services running in the cluster?

<!--
Think about both cluster-level services (e.g. metrics-server) as well
as node-level agents (e.g. specific version of CRI). Focus on external or
optional services that are needed. For example, if this feature depends on
a cloud provider API, or upon an external software-defined storage or network
control plane.

For each of these, fill in the following—thinking about running existing user workloads
and creating new ones, as well as about cluster-level services (e.g. DNS):
  - [Dependency name]
    - Usage description:
      - Impact of its outage on the feature:
      - Impact of its degraded performance or high-error rates on the feature:
-->

### Scalability

<!--
For alpha, this section is encouraged: reviewers should consider these questions
and attempt to answer them.

For beta, this section is required: reviewers must answer these questions.

For GA, this section is required: approvers should be able to confirm the
previous answers based on experience in the field.
-->

###### Will enabling / using this feature result in any new API calls?

<!--
Describe them, providing:
  - API call type (e.g. PATCH pods)
  - estimated throughput
  - originating component(s) (e.g. Kubelet, Feature-X-controller)
Focusing mostly on:
  - components listing and/or watching resources they didn't before
  - API calls that may be triggered by changes of some Kubernetes resources
    (e.g. update of object X triggers new updates of object Y)
  - periodic API calls to reconcile state (e.g. periodic fetching state,
    heartbeats, leader election, etc.)
-->

Yes, one additional PATCH call to VolumeSnapshotContent per snapshot creation to set the `AccessibleTopology` field on the Spec. This is a one-time call per snapshot, originating from the CSI snapshotter sidecar controller.

###### Will enabling / using this feature result in introducing new API types?

<!--
Describe them, providing:
  - API type
  - Supported number of objects per cluster
  - Supported number of objects per namespace (for namespace-scoped objects)
-->

No, this feature adds a new field (`AccessibleTopology`) to the existing `VolumeSnapshotContentSpec` type.

###### Will enabling / using this feature result in any new calls to the cloud provider?

<!--
Describe them, providing:
  - Which API(s):
  - Estimated increase:
-->

No, the topology information is returned as part of the existing CSI `CreateSnapshot` response. No additional cloud provider calls are made.

###### Will enabling / using this feature result in increasing size or count of the existing API objects?

<!--
Describe them, providing:
  - API type(s):
  - Estimated increase in size: (e.g., new annotation of size 32B)
  - Estimated amount of new objects: (e.g., new Object X for every existing Pod)
-->

Yes, VolumeSnapshotContent objects will have an additional `accessibleTopology` map field. Estimated increase is small depending on the number of topology key-value pairs (e.g., region and zone).

###### Will enabling / using this feature result in increasing time taken by any operations covered by existing SLIs/SLOs?

<!--
Look at the [existing SLIs/SLOs].

Think about adding additional work or introducing new steps in between
(e.g. need to do X to start a container), etc. Please describe the details.

[existing SLIs/SLOs]: https://git.k8s.io/community/sig-scalability/slos/slos.md#kubernetes-slisslos
-->

No, The additional PATCH call to set topology will likely add negligible latency to the snapshot creation flow.

###### Will enabling / using this feature result in non-negligible increase of resource usage (CPU, RAM, disk, IO, ...) in any components?

<!--
Things to keep in mind include: additional in-memory state, additional
non-trivial computations, excessive access to disks (including increased log
volume), significant amount of data sent and/or received over network, etc.
This through this both in small and large cases, again with respect to the
[supported limits].

[supported limits]: https://git.k8s.io/community//sig-scalability/configs-and-limits/thresholds.md
-->

No, the additional topology data is a small map stored on existing objects. There are no new computations, watchers, or reconciliation loops introduced.

###### Can enabling / using this feature result in resource exhaustion of some node resources (PIDs, sockets, inodes, etc.)?

<!--
Focus not just on happy cases, but primarily on more pathological cases
(e.g. probes taking a minute instead of milliseconds, failed pods consuming resources, etc.).
If any of the resources can be exhausted, how this is mitigated with the existing limits
(e.g. pods per node) or new limits added by this KEP?

Are there any tests that were run/should be run to understand performance characteristics better
and validate the declared limits?
-->

No, this feature only adds a small amount of data to existing API objects and does not affect node resources.

### Troubleshooting

<!--
This section must be completed when targeting beta to a release.

For GA, this section is required: approvers should be able to confirm the
previous answers based on experience in the field.

The Troubleshooting section currently serves the `Playbook` role. We may consider
splitting it into a dedicated `Playbook` document (potentially with some monitoring
details). For now, we leave it here.
-->

###### How does this feature react if the API server and/or etcd is unavailable?

###### What are other known failure modes?

<!--
For each of them, fill in the following information by copying the below template:
  - [Failure mode brief description]
    - Detection: How can it be detected via metrics? Stated another way:
      how can an operator troubleshoot without logging into a master or worker node?
    - Mitigations: What can be done to stop the bleeding, especially for already
      running user workloads?
    - Diagnostics: What are the useful log messages and their required logging
      levels that could help debug the issue?
      Not required until feature graduated to beta.
    - Testing: Are there any tests for failure mode? If not, describe why.
-->

###### What steps should be taken if SLOs are not being met to determine the problem?

## Implementation History

<!--
Major milestones in the lifecycle of a KEP should be tracked in this section.
Major milestones might include:
- the `Summary` and `Motivation` sections being merged, signaling SIG acceptance
- the `Proposal` section being merged, signaling agreement on a proposed design
- the date implementation started
- the first Kubernetes release where an initial version of the KEP was available
- the version of Kubernetes where the KEP graduated to general availability
- when the KEP was retired or superseded
-->

- 2025-11-19 - Enhacement was discussed in CSI Implementation Meeting.

## Drawbacks

<!--
Why should this KEP _not_ be implemented?
-->

## Alternatives

<!--
What other approaches did you consider, and why did you rule them out? These do
not need to be as detailed as the proposal, but should include enough
information to express the idea and why it was not acceptable.
-->

### Inherit topology from the source PersistentVolume

An alternative approach considered was to have the common snapshot controller copy topology information from the source PersistentVolume's `NodeAffinity` into the VolumeSnapshotContent at creation time, rather than receiving it from the SP in the `CreateSnapshotResponse`.

This was ruled out because:

- **Snapshot topology may differ from volume topology.** A storage backend may store snapshots in a different location than the source volume (e.g., a volume is zonal but its snapshot is regional). Only the SP knows where the snapshot actually resides.

- **Pre-provisioned snapshots have no source volume.** When a VolumeSnapshotContent is created with a `snapshotHandle` (pre-existing snapshot), there is no source PV to inherit from. Getting topology from the CSI driver response handles both dynamic and pre-provisioned cases consistently.

- **Single source of truth.** Having the storage backend report topology directly avoids assumptions about the relationship between volume and snapshot placement, making the design more accurate and portable across different storage systems.

## Infrastructure Needed (Optional)

<!--
Use this section if you need things from the project/SIG. Examples include a
new subproject, repos requested, or GitHub details. Listing these here allows a
SIG to get the process for these resources started right away.
-->
