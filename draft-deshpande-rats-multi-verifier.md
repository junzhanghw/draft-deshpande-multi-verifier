---
title: Remote Attestation MultiVerifier
abbrev: RATS MultiVerifier
docname: draft-deshpande-rats-multi-verifier-latest
date: {DATE}
category: info
ipr: trust200902
area: Security
workgroup: RATS

stand_alone: yes
pi:

  rfcedstyle: yes
  toc: yes
  tocindent: yes
  sortrefs: yes
  symrefs: yes
  strict: yes
  comments: yes
  text-list-symbols: -o*+
  docmapping: yes

author:

-
  name: Yogesh Deshpande
  org: Arm Ltd
  email: yogesh.deshpande@arm.com

-
  name: Jun Zhang
  org: Huawei Technologies France S.A.S.U.
  email:  junzhang1@huawei.com

contributor:
 -  name: Thomas Fossati
    organization: Linaro
    email: Thomas.Fossati@linaro.org

 -  name: Henk Birkholtz
    organization: Fraunhofer SIT
    email:  henk.birkholz@sit.fraunhofer.de

informative:
  RATS-ARCH: RFC9334

--- abstract

IETF RATS Architecture, defines the key role of a Verifier.  In a complex system, this role needs to be performed by multiple Verfiers coordinating together to assess the full trustworthiness of an Attester. This document focuses on various topological patterns for a multiple Verifier system.

--- middle

# Introduction

A Verifier plays a Central Role in any Remote Attestation System. A Verifier appraises the Attester and produces Attestation Results, which is essentially a verdict of attestation. The results are consumed by the Relying Party to conclude the trustworthiness of the Attester, prior to making any critical decisions about the Attester, such as admitting to network or releasing confidential resources to it.
Attesters can come in wide varieties of shape and form. For example Attesters can be endpoints (edge or IoT devices) or complex machines in the cloud. Composite Attesters, also known as Layered Attesters or Composite Devices(Sections 3.2 and 3.3 of [RFC9334]) generate Evidence that consists of multiple parts. For example, in data center servers, it is not uncommon for separate attesting environments (AE) to serve a subsection of the entire machine. One AE might measure and attest to what was booted on the main CPU, while another AE might measure and attest to what was booted machine's GPU. Throughout this document we use the term sub-Attester to address the sub-entity or an individual layer which produces its own Evidence in a Composite Attester system.

A Verifier needs Reference Values and Endorsements from the supply chain actors (for example OEM/ODM) to conduct the appraisal of an Attester. Given the range of sub-Attesters in a Composite Attester, it is possible that a single Verifier may not have all the capabilities or the information required to conduct the complete appraisal of the Composite Attester. In this case, multiple Verifiers need to work in tandem to conclude the appraisal and produce the Attestation Results.


This document describes various topological patterns of multiple Verifiers that work in a coordinated manner to conduct appraisal of a Composite Attester to produce an Attestation Results.

# Conventions and Definitions

{::boilerplate bcp14}

The reader is assumed to be familiar with the terms defined in {{RATS-ARCH}}.

# Multi Verifier topological patterns
{: #sec-multi-verifier }

A composite Attester as per RATS definition has multiple layers. Each layer requires a different set of Verifiers. Hence multi Verifiers work in tandem to appraise a composite Attester.

## Hierarchical Pattern {: #sec-lead-verifier }

Figure below shows the block diagram of a Hierarchical Pattern.

                                                     +----------+
                                                     |          |               +-----------+
                                                     |          |               |           |
                                                     |          |  Evidence 1   |           |
                                                     |          +-------------->+ Verifier 1|
                                                     |          |               |           |
                                                     |          +<--------------+           |
                                                     |          |    AR 1       +-----------+
                                                     |          |
               +---------------+  Composite Evidence |          |
               |               +--------------------->          |  Evidence 2   +-----------+
               |  Attester     |                     | Lead     +-------------->+           |
               |   or          |  Composite          | Verifier |               |           |
               |  RP           |<--------------------+          |               | Verifier 2|
               +---------------+  Attestation Result |          +<--------------+           |
                                    (AR)             |          |   AR 2        |           |
                                                     |          |               +-----+-----+
                                                     |          |                     |
                                                     |          |                     |
                                                     |          |                     .
                                                     |          |                     |
                                                     |          |                     |
                                                     |          |                     |
                                                     |          |   Evidence N   +----+------+
                                                     |          +--------------->+           |
                                                     |          |                |           |
                                                     |          +<---------------+ Verifier N|
                                                     |          |   AR N         |           |
                                                     |          |                |           |
                                                     |          |                +-----------+
                                                     +----------+

The following sub-sections describe the various roles that exist in this pattern.

### Lead Verifier

In this topological pattern, there is an Entity known as Lead Verifier.

Lead Verifier is the central entity in communication with the Attester (directly in passport model or indirectly via the Relying Party in background-check model). 
It receives Attestation Evidence from a Composite Attester. If the Composite Attestation Evidence is signed, then it validates the integrity of the Evidence by validating the signature. If signature verification fails, the Verification is terminated. Otherwise it performs the following steps.

* Lead Verifier has the knowledge about the overall structure of the Composite Evidence so it decodes the Evidence to extract the sub-Attester Evidence. This may lead to "N" sub-Evidence, one for each sub-Attester.


* Lead Verifier delegates each sub-Attester Evidence to its own Verifier and receives sub-Attester Attestation Results after successful Appraisal of Evidence.

* Once the Lead Verifier receives Attestation Results from all the Verifiers then it constructs a combined Attestation Results.

* Lead Verifier combines the Results from each Verifier and conveys the combined Attestation Results to the Attester
(in Passport model) or to the Relying Party (in background check model)

The overall Verdict may be dependent on the Appraisal Policy of the Lead Verifier.

In certain topologies, it is possible that only the Composite Evidence is Signed to provide the overall integrity, while the individual sub-Attester Evidence (example Evidence 1) is not protected. In such cases, the Lead Verifer upon processing of Composite Evidence produce an Attestation Result first, stash the sub-Attester Evidence (example Evidence 1) as unprocessed Evidence, sign the AR and send it to its Verifier (example Verifier 1).

### Verifier for sub-Attester

The role of a sub-Attester Verifier is to receive sub-Attester Evidence from the lead Verifier and produce Attestation Results to the Lead Verifier.

### Trust Relationships

It is important that the individual sub-Attester Attestation Results needs to have security to prevent any man in the middle attacks. Hence it is important that each sub-Attester Attestation Results MUST be signed by the individual Verifiers.

The Trust Anchors for the sub-Attester Verifiers MUST be present in the Lead Verifier Database. The Lead Verifier MUST verify the signature on the sub-Attester Attestation Results prior to processing it.

Once all sub-Attester Appraisal process is complete, the overall Attestation Results are produced. The Lead verifier may apply its own policies and also add extra claims as part of its appraisal.

In a particular deployment scenario it is possible that the received sub-Attester Results are just appended and the entire envelope signed by the Lead Verifier. A Relying Party may then choose to verify individual Verifier results, based on its Appraisal Policy for Attestation Results.



## Cascaded Pattern {: #sec-verifier-cascade }

Figure below shows the block diagram of a Cascaded Pattern.

                                       +-----------+          +-----------+                         +------------+
        +--------+                     |           |          |           |                         |            |
        |        |  Composite Evidence |           |  (CE)    |           |       (CE)              |            |
        |        +-------------------->+           +--------->+           +------------------------>+            |
        |        |     (CE)            |           |Partial AR|           |     Partial AR          |            |
        |Attester|                     | Verifier 1|          | Verifier 2|                         | Verifier N |
        |  or    |  Composite          |           |          |           |                         |            |
        | RP     +<--------------------+           +<---------+           +<------------------------+            |
        +--------+ Attestation Results |           |  (CAR)   |           |      (CAR)              |            |
                      (CAR)            |           |          |           |                         |            |
                                       +-----------+          +-----------+                         +------------+


In this topological pattern, the Attestation Verification happens in sequence. Verifiers are cascaded to perform the Attestation Appraisal. Each Verifier in the chain possess the knowledge of the entire Composite Attester topology.

Attester may send the Composite Evidence(CE) to any of the Verifier (directly in the passport model, or indirectly via the Relying Party in the background-check model). The Verifier which processes the Composite Evidence, Verifies the signature on the Evidence, if present. It decodes the Composite Evidence performs Appraisal of the sub-Attester whose Reference Values and Endorsements are in its database. Once the appraisal is complete,
it forwards the Composite Evidence and partial Attestation Results to the subsequent Verifier.

The process is repeated, until the entire appraisal is complete. The last Verifier, i.e. Verifier-N, completes its Appraisal of the sub-Attester Evidence and returns the Complete Attestation Results to the N-1 Verifier, which passed Evidence to it. The N-1 Verifier then simply passes the Combined Attestation Results(CAR) from where it received its Combined Evidence. Alternatively, it may also modify the CAR based on the inspection of received CAR. For example, it may add its own Verifier Added Claims (policy claims) and produce a new CAR. The process is repeated, until the Verifier, which recieved the initial Evidence is reached. At this point in time the Combined Attestation Results are signed and the combined Attestation Results are sent to the Attester (in Passport Model) or Relying Party (in background check model).

As shown in the picture, the partial results and Combined Evidence is transmitted to a chain of Verifier, till the Appraisal is complete.
The Verifier combines the incoming partial results, combines the results from it own Evidence Appraisal and passes the Combined Attestation Results to the Verifier from which it receives Combined Evidence.


## Trust Relationships

### Verifiers
In the cascaded pattern, the communicating Verifiers fully trust each other. Each Verifier has the trust anchor for the Verifier it is communicating to (i.e. either sending information or receiving information). This prevents man in the middle attack for the partial Attestation Results received by a Verifier or a Combined Attestation Results (CAR) which it receives in the return path.

### Relying Party and Verifiers
In the cascaded pattern, the RP may communicate with any Verifier and thus receive its Attestation Results. Hence RP must have a trust root for the root CA certificate, whose leaf certifcate key is used to sign the Attestation Results.

## Hybrid Pattern

In a particular deployment, there is a possibility that the two models presented above can be combined to produce a hybrid pattern. For example Verifier 2 in the Cascaded Pattern becomes the Lead Verifier for the remaining Verifers from 3, to N.

# Freshness
The Verifier needs to ensure that the claims included in the Evidence reflect the latest state of the Attester. As per RATS Architecture, the recommended freshness is ascertained using either Synchronised Clocks, Epoch IDs, or nonce, embedded in the Evidence.

In the case of Hierarchical Pattern, the Verification of Freshness should be checked by the Lead Verifier. 
 
In the Cascaded Pattern, the freshness is always checked by the first Verifier in communication with either the Attester (Passport Model) or Relying Party (Background Check Model).
# Security Considerations

<cref>TODO</cref>

# IANA Considerations

## CBOR Tag Registrations


# Acknowledgements


<cref>TODO</cref>
