# Laravel-Queue-Workers-Management-with-Supervisor

This README provides a comprehensive guide for setting up and managing Laravel queue workers using Supervisor. It includes configuration for workers, `numprocs` (number of processes), and how to start, stop, and manage jobs in high- and low-priority queues.

## Table of Contents

1. [Overview of Laravel Queue Workers](#overview-of-laravel-queue-workers)
2. [Setting Up Supervisor for Laravel Workers](#setting-up-supervisor-for-laravel-workers)
   - [Installing Supervisor](#installing-supervisor)
   - [Configuring Workers](#configuring-workers)
   - [Setting `numprocs`](#setting-numprocs)
3. [Stopping All Workers for a Queue](#stopping-all-workers-for-a-queue)
4. [Stopping Specific Workers for a Queue](#stopping-specific-workers-for-a-queue)
5. [Starting Workers for a Queue](#starting-workers-for-a-queue)
6. [Checking the Status of Workers](#checking-the-status-of-workers)
7. [Graceful Shutdown and Process Management](#graceful-shutdown-and-process-management)

---

## 1. Overview of Laravel Queue Workers

In this Laravel application, we use the **database driver** for queue management, while Supervisor handles the workers that process jobs. This README focuses on managing high and low-priority queues with multiple worker processes for each.

---

## 2. Setting Up Supervisor for Laravel Workers

### Installing Supervisor

To install Supervisor:

```bash
sudo apt-get install supervisor
```

Ensure it is running:

```bash
sudo service supervisor start
```

### Configuring Workers

#### High and Low priority queue configuration:

Create a configuration file for the queue:

```bash
sudo nano /etc/supervisor/conf.d/laravel-worker.conf
```

Add this content for high priority queue:

```ini
[program:prod-laravel-worker-high]
process_name=%(program_name)s_%(process_num)02d
command=php /path-to-your-laravel-project/artisan queue:work database --queue=high --sleep=3 --tries=3
autostart=true
autorestart=true
user=root
numprocs=3
redirect_stderr=true
stdout_logfile=/path-to-your-log-directory/worker-high.log
```

Add this content for low priority queue:

```ini
[program:prod-laravel-worker-low]
process_name=%(program_name)s_%(process_num)02d
command=php /path-to-your-laravel-project/artisan queue:work database --queue=low --sleep=3 --tries=3
autostart=true
autorestart=true
user=root
numprocs=3
redirect_stderr=true
stdout_logfile=/path-to-your-log-directory/worker-low.log
```

### Setting `numprocs`

- `numprocs` specifies the number of worker processes for each queue. In the examples above:
  - `prod-laravel-worker-high` runs 3 processes for the high-priority queue.
  - `prod-laravel-worker-low` runs 3 processes for the low-priority queue.

To adjust the number of workers, modify the `numprocs` value in each configuration.

After updating the configurations, apply the changes:

```bash
sudo supervisorctl reread
sudo supervisorctl update
```

---

## 3. Stopping All Workers for a Queue

To stop all workers for a specific queue, run:

```bash
sudo supervisorctl stop prod-laravel-worker-high:
```

For the low-priority queue:

```bash
sudo supervisorctl stop prod-laravel-worker-low:
```

---

## 4. Stopping Specific Workers for a Queue

To stop a specific worker, use the process name and index. For example:

```bash
sudo supervisorctl stop prod-laravel-worker-high:prod-laravel-worker-high_01
```

---

## 5. Starting Workers for a Queue

Start all workers for the `high` priority queue:

```bash
sudo supervisorctl start prod-laravel-worker-high:
```

Start all workers for the `low` priority queue:

```bash
sudo supervisorctl start prod-laravel-worker-low:
```

### Starting Specific Workers:

To start only one specific worker:

```bash
sudo supervisorctl start prod-laravel-worker-high:prod-laravel-worker-high_01
```

---

## 6. Checking the Status of Workers

To check the status of all your workers:

```bash
sudo supervisorctl status
```

Example output:

```bash
prod-laravel-worker-high:prod-laravel-worker-high_00   RUNNING   pid 600**, uptime --:--:--
prod-laravel-worker-high:prod-laravel-worker-high_01   STOPPED   pid 601**, uptime --:--:--
prod-laravel-worker-high:prod-laravel-worker-high_02   RUNNING   pid 600**, uptime --:--:--
prod-laravel-worker-low:prod-laravel-worker-low_00     RUNNING   pid 600**, uptime --:--:--
prod-laravel-worker-low:prod-laravel-worker-low_01     STOPPED   pid 600**, uptime --:--:--
prod-laravel-worker-low:prod-laravel-worker-low_02     RUNNING   pid 600**, uptime --:--:--
```

---

## 7. Graceful Shutdown and Process Management

When stopping workers, Laravel workers can be shut down gracefully, meaning they will complete their current job before stopping.

To gracefully stop all workers for a queue:

```bash
sudo supervisorctl stop prod-laravel-worker-high:
```

---

## Summary of Commands:

| Action                           | Command                                                                 |
|-----------------------------------|-------------------------------------------------------------------------|
| Stop all workers for a queue      | `sudo supervisorctl stop prod-laravel-worker-high:`                |
| Stop specific worker              | `sudo supervisorctl stop prod-laravel-worker-high:prod-laravel-worker-high_01` |
| Start all workers for a queue     | `sudo supervisorctl start prod-laravel-worker-high:`               |
| Start specific worker             | `sudo supervisorctl start prod-laravel-worker-low:prod-laravel-worker-low_00`  |
| Check the status of workers       | `sudo supervisorctl status`                                             |

