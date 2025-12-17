---
layout: page
title: OzStar – Loop Script
permalink: /ozstar-loop/
---

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
    JOB_OUTPUT=$(sbatch launch_your_job)
    
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
done

