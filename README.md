# af-base-template
LabVIEW Template based on the actor framework for data logging and control applications

# Required VIPM Packages
* JSONtext v1.6.10.113 by JDP Science
* OpenG File Library v4.0.1.22 by OpenG.org

# Definitions
## Data
Data is stored in memory as byte arrays  
They are referenced by a location in memory through a reference and what's shared across the application is not the data itself but a DVR to that data.

The data content is defined as follows:
- Header: array of 145 bytes comprised of
  - Priority: 1 byte (0: Act immediately and report errors, 1: Act immediately but disregard errors, 2: Do not process)
  - Timestamp: 16 bytes
 - Metadata: Reserved for specific use, 128 bytes
- Data: Variable size bite array
- Footer: Reserved for specific use, 128 bytes

Data is also copied to a UI DVR instance that holds only the latest point received in the "send data.vi". This can be accessed by the nested actor's User Interface override to display the latest piece of data made available by the application.  
This data is overwritten whenever a new data packet is ingested.

## Errors
Errors can be only handled by the root actor, which can either send it to any actor registered with an \<\*error\*\> as its name or shut down the application if it is not found anywhere.

The error flow is described below:

1. Error happens in the "Actor Core.vi" override during a message processing task.  
↓  
2. "Actor Core.vi" executes "Handle Error.vi" override.  
↓  
3. In the error case, "fire error event.vi" triggers the "New Error" user event  
↓  
4. New Error user event triggers the New Error event in the "loop helper.vi"  
↓  
5. New Error event case executes the "error intake.vi"  
↓  
6. a. If this actor is the root actor:  
* Searches in the nested actors map for any actor containing __'error'__ in its name.
  * If error actor is found, sends "process error.vi" interface override message to it.
  * if error actpr is **not** found, sends "process error.vi" interface override message to **itself**   
6. b. If this actor is **not** the root actor:  
* Executes "share error upstream.vi" that reads the caller enqueue and sends the message: "fire error object.vi"
* "fire error object.vi" triggers the "New Error" user event in the caller's actor core.
* Everything is repeated from step 4. onwards until it reaches the root actor