**Scenario**: ["Saint John": what is writing to this log file?](https://sadservers.com/newserver/saint-john#)

**Level**: Easy

**Description**: A developer created a testing program that is continuously writing to a log file /var/log/bad.log and filling up disk. You can check for example with tail -f /var/log/bad.log.
This program is no longer needed. Find it and terminate it.

**Test**: The log file hasn't changed in the last 6 seconds: find /var/log/bad.log -mmin -0.1 (You don't need to know the details of this command).

**Time to Solve**: 10 minutes.

-------------------------------------
-------------------------------------

Let's start with listing all the processes using `ps` and try to find the testing program using `grep` with related keywords.

```bash
ps auxf | grep log
```

Found the offending process, a python program called `badlog.py` with a `PID 624`, let's terminate it using `kill` as per the requirement.

![](../assets/saint_john_1.jpg)

```bash
kill -9 624
```

An **alternative yet better approach** is to use the command to list open files: `lsof`

1. Find the name (first column) and Process ID (PID, second column) of the process related to `/var/log/bad.log` by running lsof and filtering the rows to the one(s) containing `bad.log`.

2. You can also use the `fuser` command to quickly find the offending process: `fuser /var/log/bad.log`

3. Run: `lsof |grep bad.log` and get the PID (second column).

4. With the PID of the process, it's not necessary but we can find its current working directory (program location) by doing `pwdx PID` or for more detail: `lsof -p PID` and check the `cwd row`. This will allow us to check its ownership and perhaps inspect its offending code if it's a script (not a binary).

5. Inspecting the offending code, though not required for this problem statement.
    ```python
    #! /usr/bin/python3
    # test python writing to a file

    import random
    import time
    from datetime import datetime

    f = open('/var/log/bad.log', 'a')
    while True:
      r = random.randrange(2147483647)
      f.write(str(datetime.now()) + ' token: ' + str(r) + '\n')
      f.flush()
      time.sleep(1)
    ```
6. Solution: Using the PID found, terminate (kill) the process with `kill -9 PID`.

