# af-base-template
LabVIEW Template based on the actor framework for datalogging and control applications

## Required VIPM Packages
* JSONtext v1.6.10.113 by JDP Science
* OpenG File Library v4.0.1.22 by OpenG.org

## Definitions
# Data
Data is stored in memory as byte arrays
They are referenced by a location in memory through a reference and what's shared accross the application is not the data itself but a DVR to that data.

The data content is defined as follows:
- Header: array of 145 bytes comprised of
 - Timestamp: 16 bytes
 - Priority: 1 byte (0: Act immediately and report errors, 1: Act immediately but disregard errors, 2: Do not process)
 - Metadata: Reserved for specific use, 128 bytes
- Data: Variable size bite array
- Footer: Reserved for specific use, 128 bytes

Data is also copied to an UI DVR instance that holds only the latest point received in the "send data.vi". This can be accessed by the nested actor's User Interface override to display the latest piece of data made available by the application.
This data is overwritten whenever a new data packet is ingested.