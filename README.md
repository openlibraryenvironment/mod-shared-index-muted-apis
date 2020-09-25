# Shared Index: Muted APIs


## Purpose
The purpose of this module is to reduce the installation footprint of the shared index by setting up faux APIs to satisfy certain expressed requirements in some of the core shared index modules, namely requirements for secondary modules that actually aren't necessary for the shared index to function. 

The faux APIs act as stand-ins for unneeded APIs that are requested in the `requires` sections of the essential modules. This way it is not necessary to actually install those secondary modules or any other dependencies that they may in turn pull in. 

The motivation is to simplify the installation of a shared index for development purposes. It's probably only useful in scenarios where the developer installs everything by hand - rather than using a prebuilt back-end box with the whole FOLIO back-end ecosystem in it. 

Though something along these lines _could_ be considered for a shared demo or test service as well, such blunt tactics may not be appropriate for a production installation.  
## Rationale
The Shared Index only needs a few FOLIO modules to serve its purposes. Besides `Okapi` and `Stripes` the essential modules are:

Backend 
- `Inventory Storage Module`
- `Inventory Module` 
- `MARC Record Storge Module`
- `Inventory Update Module`
- `Users`, `users business logic`
- `login`, `authtoken`, `Password Validator Module` (?), `permissions`
- `Configuration` (?)

Frontend
- `ui-inventory`
- `ui-users`
- ?

However, some of these essential modules declare dependencies on secondary modules that in turn declare dependencies on even more modules, greatly expanding the dependency tree beyond what's needed for the shared index.

At this point the shared index does not need or use FOLIO circulation capability, for example, but `ui-inventory` requests the `circulation` API in its 'requires' section, which in turn requires a number of other modules to be present.

The `Inventory Module` requests `source-record-storage` and Pub-Sub to be present, but the shared index has its own source storage and doesn't need pub-sub functionality at this point. 

The `users` module requires `notify`, which in turn pulls in a number of other modules for sending out notifications. It's muted here because it's not needed for developing the shared index but it may or may not be required for user management in a production installation of the shared index.  

Muting those dependencies reduces the footprint of the overall FOLIO install significantly for the shared index, in terms of:
  - the number/length of scripts etc needed to install the shared index for development purposes
  - the time it takes to complete the installation process 
  - the complexity of upgrading modules when new FOLIO releases are out
  - the memory required on a developer box to run a full install of the shared index


## Implementation
The Shared Index Muted APIs module `provides` following faux APIs:

Requested by `Inventory Module`
- `pubsub-event-types 0.1`
- `pubsub-publishers 0.1`
- `pubsub-subscribers 0.1`
- `pubsub-publish 0.1`
- `request-storage 3.3`
- `source-record-storage 2.0`

Requested by `users business logic`
- `notify 2.0`

Requested by `UI Inventory` 
- `circulation 9.0`

## Prerequisites

- Java 11 JDK
- Maven 3.3.9

## Building

run `mvn install` from the root directory.

## Further developments

- The `circulation` API must return a dummy response to prevent error modals in the UI. 
- The `request-storage` API should return a dummy response to prevent errors in the browser's JavaScript console.
- All the other APIs could return an HTTP error status in the event that they are unexpectedly invoked after all.
