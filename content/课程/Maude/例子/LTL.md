# LTL Exercises

## Exercise 4 (Using Until)
**Atomic propositions:**
* `init`: the system is in initialization mode.
* `configured`: the system has finished configuration.
* `running`: the system is executing its normal workload.

**Property:**
The system stays in initialization mode until it becomes configured, and while it is in initialization mode it must not be running.

**LTL Formula:**
```maude
(init /\ ~ running) U configured
```

## Exercise 5 (Using Next and Until)
**Atomic propositions:**
* `maintenanceRequest`: a maintenance request is issued.
* `maintenance`: the system is in maintenance mode.
* `done`: maintenance has been completed.

**Property:**
Whenever a maintenance request occurs, then at the next step the system must enter maintenance mode and remain in maintenance mode until maintenance is done.

**LTL Formula:**
```maude
[] (maintenanceRequest -> O (maintenance U done))
```
