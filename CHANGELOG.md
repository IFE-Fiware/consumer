## 1.3.1 (2025-05-14)

No changes.


# Changelog
All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.3.2] - 2025-05-16
- Updated default value for Vault service.
- Hotfix for new naming of advsearch.

## [1.3.1] - 2025-05-14
- Updated readme regarding secret creation.

## [1.3.0] - 2025-04-18
- Updated many components to implement Consumer version 1.3.0.
- Moved Redis and PostgreSQL components to Common Agent
- Added Notification component to Common Agent


### Simpl Cloud gateway (Tier 1)

#### 1.3.1 (2025-04-11)
No changes.

#### 1.3.0 (2025-03-31)

#### Fixed
- SIMPL-9234 BUGFIX boolean methods names


### Users Roles

#### 1.3.1 (2025-04-11)
No changes.

#### 1.3.0 (2025-03-31)

#### Added
- Event management for deleted onboarding requests

#### Changed
- Replaced StreamingResponseBody with Resource

#### Fixed
- SIMPL-10198 Bugfix


### SIMPL FE

#### 1.3.5 (2025-04-16)

#### Fixed
- [SIMPL-10718] - Fixed Request revision visibility for Notary
- fixed checkbox behavior, now you cannot select an identity attribute if it cannot be assigned to a role

#### 1.3.4 (2025-04-08)

#### Fixed
- fix api v1 with prefix

#### 1.3.3 (2025-04-07)

#### Fixed
- fixed rejection cause message and update info

#### 1.3.2 (2025-04-02)

#### Fixed
- fixed /authApi/v1/credentials API for multipart

#### 1.3.1 (2025-04-01)

#### Added
- added loading in echo button

#### Changed
- removed old alert banner with new EUI growl

#### Fixed
- fix endpoint API

#### 1.3.0 (2025-03-31)

#### Added
- [SIMPL-9111] - EUI Init for SAP MFE
- added auth guard
- added and configured eui for onboarding micro frontend
- [SIMPL-9111] - Refactoring Participant Type Page


#### Changed
- onboarding refactoring request status page and comment component in library
- onboarding request added eui component and restyling page
- Show and hidden password
- edit pages info page and request credentials by replacing material with eui design system (merge request)

#### Fixed
- [SIMPL-9111] - Fixed Identifier Column in Identity Attributes Page. See [SIMPL-918]


### TLS Gateway (Tier 2)

#### 1.3.1 (2025-04-11)
No changes.

#### 1.3.0 (2025-03-31)

#### Changed
- AuthenticationProviderClient now use Exchanges to make http requests

#### Fixed
- SIMPL-9510 Bugfix on boolean method name


### Authentication Provider

#### 1.3.4 (2025-04-11)
No changes.

#### 1.3.3 (2025-04-08)

#### Fixed
- Revert CredentialId fix on MTLSController

#### 1.3.2 (2025-04-07)

#### Fixed
- Update http client version

#### 1.3.1 (2025-03-31)

#### Fixed
- CredentialId decode

#### 1.3.0 (2025-03-31)

#### Changed
- Replaced StreamingResponseBody with Resource


### xsfc-advsearch-be

#### 1.6.0 (2025-03-07)

#### Added
- SIMPL-10614 Creation of status API endpoint
- added new charts values.yaml key 'openapiConfig.servers'

#### Changed
- simpl-http-client-feign updated to 1.0.1
- code review: removed unused API and relative methods

#### Fixed
- Enabled PVC
- SIMPL-8413 Use meaningful names for methods in CatalogueAdapterTier2Client


### simpl-edc

#### 1.0.3 (2025-03-07)

#### Changed
- SIMPL-9498 - Clean up the development cluster
- SIMPL-9494 - Fix requesting resource from one edc to another one on dev env
- SIMPL-9106 - PSO | Code verification: Determine the layers of the code correctly and put classes to the corresponding packages


### contract-consumption-be

#### 1.4.0 (2025-02-17)

#### Changed
- changed deprecated @MockBean with @MockitoBean in test classes
- docker file: updated entrypoint in order to run application with user simpluser


### Filebeat

#### 0.1.12 (2025-03-07)

#### Added
- Added logs parsing for new containers
- Added end user manual for dashboards

#### Changed
- Changed default values for kibana resources

#### Fixed
- Moved changelog file to correct directory
- Upgrade ELK to 8.16.0
