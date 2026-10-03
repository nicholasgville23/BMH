# BMH

This repository contains the BMH project, a Java-based broadcast messaging and emergency alert system used for message validation, scheduling, delivery, and audio generation.

This version is maintained collaboratively by @warrickmoran and @nicholasgville23.

## Overview

BMH is designed to handle weather and emergency message workflows across multiple components:

- message intake and validation
- SAME/MRD message parsing and rule enforcement
- scheduling and replacement logic
- transmitter and suite configuration
- text-to-speech / voice dictionary support
- broadcast playback and communications handling
- EDEX and client-side integration points

At a high level, the project combines shared Java model code, Eclipse plugin/UI modules, server-side components, and deployment/configuration assets.

## Repository structure

- `cave/` - CAVE client-side Eclipse/RCP modules and GUI components
- `common/` - shared Java library projects and data model packages
- `edex/` - EDEX server-side integration, handlers, DAQ/comms, and processing
- `features/` - Eclipse feature metadata
- `foss/` - third-party/embedded library dependencies
- `rpms-BMH/` - RPM packaging and deployment assets
- `deltaScripts/` - schema and deployment update scripts
- `test/bmh.testSuite/` - scenario-based validation tests for messaging behavior

## Key technology areas

- Java-based service and model layers
- Eclipse/OSGi plugin structure
- Spring XML configuration for service wiring
- SQL schema and migration/update scripts
- Python helper scripts for operational tasks and test support
- RPM-based installation and runtime packaging

## Typical usage

This repository is not a standalone desktop app in the usual sense. It is organized as a multi-module system intended to run within a larger operational software environment, with client, server, database, and deployment artifacts working together.

The project is most useful when deployed in a Raytheon/eDEX-style environment where message configuration, playlists, transmitters, DACS, and operational services are available.

## Notes

- The codebase is primarily Java and is structured around Eclipse projects.
- Build/deployment behavior depends on the surrounding environment and operational configuration.
- This repository includes extensive test data and scheduling scenarios to validate message processing rules.

## Project status

The repository appears to be an active, multi-module Java codebase for operational broadcast management and alert dissemination. It contains both source code and deployment assets used to support message services in production-like environments.

## Repository origin

This README was added for the collaborative BMH repositories maintained by @warrickmoran and @nicholasgville23.

