# SuChron
Unix cron is lame and we can do better! This is a proposal of a uri-ish syntax that not only is more readable but also expands upon the feature-set available in `cron`.

## Context/Protocol

The first portion of a SuChron uri indicates how it should be interpreted with the "suchr" shorthand:  
`suchr:...`  

A full version of the string should also be supported:  
`suchron:...`

## Frequency: Describes a reliable time reference and algorithm. SuChron includes the following:

`suchr:tai:...` - Atomic* time format.  
`suchr:gps:...` - GPS* time format.  
`suchr:utc<+/-><#>:...` - Constant UTC Offset in # hours.  
`suchr:_:...` - "Inherit" i.e. let the system decide.  
`suchr:tz<timezone>:...` - Timezone abbreviation. Implementation specific.  

All of these times are assumed to be zero at the unix epoch unless their standard has a separate definition.

* GPS and Atomic time units have their own standards that should be followed in SuChron implementation.

## Unit:

The third section includes the unit that should be referred to for this descriptor.  
- 0 : ZERO (No time. Multiplication ignored.)
- fs : femptoseconds (1E-15 seconds)  
- ps : picoseconds (1E-12 seconds)  
- ns : nanoseconds (1E-9 seconds)  
- us : microseconds (1E-6 seconds)  
- ms : milliseconds (1E-3 seconds)  
- cs : centi-seconds (1/100 seconds)  
- ds : deci-seconds (1/10 seconds)  
- s : seconds  
- m : minutes  
- h : hours  
- d : days  
- wk : weeks (w also works but wk is preferred)  
- mo : months  
- yr : years (y also works but yr is preferred)  

`suchr:tai:ms...` would describe one millisecond in atomic time.  

These times can be expanded by multiplying with the `*` operator. `suchr:tai:ms*2348`, for example, would describe an event that occurs every 2348 milliseconds.

It is even possible to combine units with the `+` operator! `suchr:_:yr*2022+mo*9+d*27` would be 27 days + 9 months + 2022 years.

## Quantity

A SuChron definition followed by `#` and then an integer describes the number of times that the event should occur:

`suchr:gps:ms*230+yr+h*23#2` describes an event that is based on gps time and occurs every 1 year, 23 hours, and 230 milliseconds. Once it has executed twice it will automatically close.

If `#` is not included then a chron is assumed to be infinite. It must have #1 in order to only occur once.

## Referring to other chrons:  
SuChron instances can be based on other SuChron instances! SuChron assumes 3 primary objects:  

Chron `Class` - Enables grouping  
Chron `Gen` (Generator) - Spawns chron instances based on its parameters  
Chron `Inst` (Instance) - A singular chron  

### Chron Instance - `ci`
This is a single instance of something happening. 

A chron instance may have the following events:
- Chron Instance Spawn (cis) -Begins to exist but isn't necessarily open/active
- Chron Instance Open (cio) - Open/active (Is relevant)
- Chron Instance Modified (cim) - Modified (e.g. postponed, description changed, un-opened, etc.)
- Chron Instance Closed (cic) - Is complete, canceled or otherwise no longer relevant
- Chron Instance Despawn (cid) - Regardless of current state, chron has been eliminated

A chron generator or class may have the following events:
- Chron Generator Open (cgo/cco) - Open/active (Is relevant)
- Chron Generator Modified (cgm/ccm) - Modified (It is up to monitoring functions to figure out what the nature of the modification was)
- Chron Generator Closed (cgc/ccc) - Is complete, canceled or otherwise no longer relevant
- Chron Listener (kinda like chron gen? but not triggered based on time?) - TODO

### Referencing Chrons:  
`suchr:cgm:<ID>*2` would refer to every second instance of a chron generator being modified that has ID "ID"
  
`suchr:ccs:<ID>*3+ms*30` would refer to 30ms after every 3rd chron instance of class ID spawns

`suchr:tai:ms*99#2@ccc:<ID>#22` is a chron that executes twice (separated by 99ms) every time that a chron class "ID" event closes until 22 chron class "ID" events have closed
  
# Shifting the base time reference:
  
The `@` operator shifts the base time to the timepoint defined by what is to the right. The text to the right may re-define the time-base and allows for combining time-bases. If no time-base is defined then it should be assumed to be the same. If it is a chron reference then it should be based on the time that the chron was defined in:

`suchr:gps:h*2@cic:<ID>` : 2 gps-time hours after a chron instance closed event.
  
# Minus ("-") operator. 

This operator works much like the plus "+" operator but when referring to other chrons it will only operator on assumed values as it can (obviously!!) not predict the future.

TODO:
- Allow text month descriptions (JAN, FEB, March)
- Allow text day-of-week descriptions (MON, T, W, Thursday)
- technically @ creates a hidden Chron Gen that fires the chron to the right! does that mean that the command can be on the far left?! `suchr:<cmd>@chron@chron`
- get rid of * operator. just use # for chron references and then + or @. then 2h, 25m, 0, etc.
- use ~ to force a specific time resolution for execution.
- flag if new recurrences should despawn old or leave them
