# Notes on *Operating Systems : Three Easy Pieces*

Author: Akiyama

Create : 13 Feb 2022

Update: 13 Feb 2022

## Process need go over

TO DO





## Scheduling

### Metric:

* $ T_{turnaround} = T_{complection} - T_{arrival} $
* $T_{response}  = T_{firstrun} - T_{arrival}$

### FIFO (First In, First Out) / FCFS (First Come, First Serve)

* Bad in both Response Time and Turnaround TIme
* Non-preemptive scheduler

### Shortest Job First (SJF)

* Non-preemptive scheduler
* Bad in Response Time
* Improved in Turnaround Time

### Shortest Time-to-Complection First (STCF)

* Preemptive scheduler 
* Much improved in Turnaround Time
* Bad in Response Time

### Round Robin

* Good in Reponse Time. 
* Less the __Time Slice__, better the Response Time
* Bad in Turnaroung Time

### Multiple Level Feedback Queue

* Good Turnaround Time
* Good Response Time

### Lottery Scheduling

__Unfairness Metric : __ $ U =  Time_{A} / Time_{B}$ , where $ U = 1 $ will get the best fairness

* Only be precise in large job scale, because we use randomness
* Use List structure for implementation

### Stride Scheduleing

* Avoid randomly unfairness even in small number of jobs

* Pick the process with lowest __pass value__. , run it with given __stride__ of time, and increase its __pass value__, and push it back to queue finally
* It have __global state__ , so it's worse than __Stride Scheduling__ in some way

### Linux Complete Fairness Scheduler (CFS, Only Real-world Scheduler)

* Pick process with lowest __vruntime__, CFS use `sched_latency` / number of process to determine the time of the process to run, but it cannot be lower than `min_granularity`
* Introduced Weighting to calculate __vruntime__
* Use RB-Tree to store running processes

