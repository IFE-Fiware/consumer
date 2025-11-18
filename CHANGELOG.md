# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [2.4.0] - 2025-11-15
- Updated many components to implement Consumer version 2.4.0.

### Simpl Cloud gateway (Tier 1)

#### 2.5.0 (2025-09-29)

#### Added
- Added new routes for Security Attributes Provider

#### Fixed
- Https constraints applied in Content Security Policy only when https origins are present


### Users Roles

#### 2.5.1 (2025-10-16)

#### Fixed
- Identity Attributes validation now handles correctly identity attributes not assigned to participant, not assignable to roles and disabled.

#### 2.5.0 (2025-09-29)

#### Fixed
- SIMPL-12860
- SIMPL-16081


### SIMPL FE

#### 2.5.0 (2025-09-29)

#### Added
- SIMPL-14573
- SIMPL-16741
- SIMPL-16738
- SIMPL-16739
- SIMPL-16740

#### Fixed
- SIMPL-16738

### TLS Gateway (Tier 2)

#### 2.5.0 (2025-09-29)

#### Added
- Added new routes for Security Attributes Provider

#### Fixed
- SIMPL-14604


### Tier 2 Proxy

#### 1.0.1 (2025-08-06)

#### Fixed

- Fixed base docker image


### Authentication Provider

#### 2.5.2 (2025-10-17)

#### Fixed
- Removed bitnami legacy image from helm chart

#### 2.5.1 (2025-10-07)

#### Fixed
- Attempt identity attributes update after storing the ephemeral proof
- Avoid storing already expired ephemeral proofs

#### 2.5.0 (2025-09-29)

#### Added
- SIMPL-17522
- SIMPL-17529
- SIMPL-17530
- SIMPL-17492
- SIMPL-17517
- SIMPL-17516

#### Fixed
- SIMPL-16621

### xsfc-advsearch-be

#### 1.15.0 (2025-09-26)

#### Added
- SIMPL-14978

#### Fixed
- fixed request logging issue

#### Changed
- simpl-data1-common updated to 1.5.0
- SIMPL-17497


### edc connector adapter

#### 1.7.0 (2025-09-26)
No changes.


### contract-consumption-be

#### 1.12.0 (2025-09-26)

#### Fixed
- SIMPL-13435


### simpl-edc

#### 1.0.11 (2025-09-05)

#### Changed

- SIMPL-14812 fix sonar issues


### simpl-catalogue-client

#### 2.0.0 (2025-09-29)
No changes.


### Filebeat

#### 0.1.19 (2025-09-26)

#### Fixed
- SIMPL-18667 Fix cluster health alert

#### Changed
- SIMPL-18665 Create ILM policy for filebeat


### Contract Manager

#### 2.0.9 (2025-10-02)

No changes.


### Signer (Stubs)

#### 2.0.2 (2025-07-23)
No changes.
