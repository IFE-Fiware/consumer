# Changelog
All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.2.0] - 2025-03-13
- Updated many components to implement Consumer version 1.2.0.
- Implemented automatic onboarding.

## [1.1.3] - 2025-02-24

### Changed
- Added denied-urls of mtls paths in simpl-cloud-gateway

## [1.1.2] - 2025-02-13

### Changed
- Fixed Filebeat deployment

## [1.1.1] - 2025-12-02

### Changed
- Readme cleanup and fix of TOC

## [1.1.0] - 2025-01-30

### Changed
- Installation of the Consumer agent
- Consumer secured access to Governance Authority to request security credentials (certificates)
- API gateway, secured with certificates, for Consumer communications with the Governance Authority for catalogue search
- Central Governance Authority catalogue search functionality, including a UI
- Basic service usage contract set-up with a Data provider DataSpace participant, after request by the Consumer of the selected service offering from the central catalogue, including functionality available in the UI
- Consumer access to a contracted dataset (direct download) via notification
- Consumer access to a contracted VM infrastructure + dataset bundle (indirect access) via email notification
