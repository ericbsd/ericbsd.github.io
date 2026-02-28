Title: How I Used SIGUSR1 To Avoid Python Process Conflicts
Date: 2026-02-28
Category: Tutorial
Tags: Python, IPC, SIGUSR1, Update-Station
Slug: how-i-used-sigusr1-to-avoid-python-process-conflicts

Lately I was looking at a solution to avoid `update-station check-now` starting a new process that could potentially clash with the tray process. I learned how to use SIGUSR1 for IPC to avoid starting a second instance of Update Station when doing a check-now for updates.

## The Problem

Update Station runs as a system tray daemon. It also accepts a `check-now` argument to check if updates are available and open the update window. The problem is that if the tray was already running, `check-now` would start a second instance of update-station unnecessarily. The tray was already there and capable of opening the update window itself.

```python
# Old approach, always spawns a second process even if tray is running
if arg[1] == "check-now":
    Data.close_session = True
    StartCheckUpdate()
```

## The Solution: IPC via SIGUSR1

Instead of spawning a second process, I wanted the `check-now` invocation to find the running tray and just tell it to open the window. That meant I needed three things: a way to find the running process, a way to talk to it, and a way for it to listen.

### Finding the running process

The first thing I needed was a way to identify the running tray process reliably. I did that by giving it a recognizable name using `setproctitle`, which is a libc function on FreeBSD that changes the process name visible in `ps` and `/proc`. Python does not expose it directly but `ctypes` makes it easy to call:

```python
import ctypes
import ctypes.util

libc = ctypes.CDLL(ctypes.util.find_library('c'))
libc.setproctitle(b'update-station')
```

After this call the process cmdline becomes `['python: update-station']`, something unique I could search for. Then I used `psutil` to scan running processes and find it:

```python
import os
import psutil

def find_running_instance_by_name() -> int:
    my_pid = os.getpid()
    for proc in psutil.process_iter(['pid', 'cmdline']):
        try:
            cmdline = proc.info['cmdline']
            if (cmdline and len(cmdline) == 1
                    and 'update-station' in cmdline[0]
                    and proc.info['pid'] != my_pid
                    and proc.is_running()):
                return proc.info['pid']
        except (psutil.NoSuchProcess, psutil.AccessDenied, psutil.ZombieProcess):
            pass
    return 0
```

### Talking to it

Once I had the PID, I needed a way to tell the running tray to open the update window. I used SIGUSR1 for that. It is a user-defined signal that does nothing by default, which makes it perfect for this kind of custom IPC. Sending it is just one line:

```python
import signal

def send_signal_to_instance(pid: int) -> bool:
    try:
        os.kill(pid, signal.SIGUSR1)
        return True
    except OSError:
        return False
```

### Making the tray listen

The last piece was making the running tray respond to the signal. The tricky part here is that signals arrive outside the GTK main loop, so you cannot touch any widgets directly from a signal handler or you will get crashes. The solution is `GLib.idle_add()`, which schedules the work to run safely on the main thread:

```python
from gi.repository import GLib

def signal_check_now(signum, frame):
    def start_check():
        StartCheckUpdate()
        Data.system_tray.tray_icon().set_visible(False)
        return False

    Data.stop_pkg_refreshing = True
    GLib.idle_add(start_check)

signal.signal(signal.SIGUSR1, signal_check_now)
```

### Putting it all together

With all three pieces in place, the `check-now` logic became simple. If a running instance is found, signal it and exit. If not, open the window ourselves:

```python
existing_pid = find_running_instance_by_name()

if arg[1] == "check-now":
    if existing_pid > 0:
        if send_signal_to_instance(existing_pid):
            sys.exit(0)
    # No existing instance, open the window ourselves
    Data.close_session = True
    StartCheckUpdate()
```

The key insight is that the `check-now` invocation never needs to create a window itself. It just tells the process that is already running to open it.

The result is a simpler `check-now` flow. Before, it had to go through a lot of verification logic to avoid clashing with the running tray. Now it does not need any of that, because it just signals the process that is already running to do the work.
