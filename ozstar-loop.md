---
layout: page
title: OzStar – Loop Script
permalink: /ozstar-loop/
---

##Why do we need eternal loops

On OzSTAR, jobs requesting long wall times (e.g. 24 hours) often sit in the queue for a long time before starting.

To avoid this, we can request shorter wall times (e.g. 1–2 hours) and allow the loop to resubmit jobs immediately after completion.
This can significantly reduce queue wait times.

## Overview

This page documents a simple “eternal loop” wrapper for running jobs on OzStar (Slurm).

It repeatedly:

1. submits a Slurm job
2. captures the job ID
3. waits until the job is no longer in the queue
4. submits again

---

## Script: `eternal_run.sh` 

Save the following as `eternal_run_.sh`:

```bash
#!/bin/bash

cd /path/to/your/directory

# Define a log file
LOG_FILE="your_loop_log.txt"

echo "=== Starting your job loop at $(date) ===" >> "$LOG_FILE"
echo "Log file: $LOG_FILE"

while true; do
    # Submit the job
    JOB_OUTPUT=$(sbatch your_job.sh)
    
    # Extract job ID
    JOB_ID=$(echo "$JOB_OUTPUT" | awk '{print $4}')

    # Log the submission
    echo "$(date): Submitted your job array with Job ID $JOB_ID" >> "$LOG_FILE"
    echo "Submitted your job array with Job ID $JOB_ID"

    # Wait until all matching jobs are finished
    echo "Waiting for your jobs to finish..."
    while squeue --me | grep -q "$JOB_ID"; do
        sleep 10m
    done

    # Log completion
    echo "$(date): your job array $JOB_ID completed" >> "$LOG_FILE"
    echo "your job array $JOB_ID completed at $(date)"

    # Optional notification (can be replaced with a mail command)
    echo " [Notification] Job $JOB_ID finished at $(date)"
```

In the above script `your_job.sh` is your bash script containing the slurm directives.

##How to run and stop the loop

Make it executable (only once)
```bash
chmod +x eternal_run.sh
```
Run it in the background
```bash
nohup ./eternal_run.sh > loop_output.log 2>&1 &
```
Check if it is running
```bash
ps aux | grep eternal_run
```
Stop it with
```bash
kill <PID>
```
or
```bash
pkill -f eternal_run.sh
```
Check Slurm jobs
```bash
squeue -u <your user name>
```


##Notes on Responsible Use

This script submits jobs in an infinite loop.
It is powerful and very useful, but must be used carefully on a shared system.

Important cautions:

- Run only one instance at a time. Multiple loops will submit duplicate jobs.
- Do not forget it is running!
- The script will not stop on its own.
- Avoid unstable jobs. If jobs fail immediately, the loop may resubmit continuously.



done

