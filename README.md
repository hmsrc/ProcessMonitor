# HMS Process monitor

We use this code to watch running user process on the shell login nodes of our HPC cluster environment.  A cron job runs periodically to make sure users are not running cpu intensive or memory intensive jobs directly on the login servers.

For our use case we run the cron job every 5 minutes and check for processes using more than 80% cpu or 25% of memory.  The bash code is very basic and uses a history file to track pid's which violate the usage policy.  A job which exceeds cpu or memory threshold in 2 successive cron intervals is considered a violation and terminated.  An email is sent to the user indicating the job was terminated.

The ps command is used to gather process information.  The monitor loops over all the process entries and will itself cause some load while running.

The code is modified slightly from our running configuration to make it more easily adapted to other environments.  Otherwise, the code has been stable in our production HPC cluster.

Note: this code uses the kill command and has the potential to make a system unstable or set everything on fire when run as root.
Please test using a non root user and the provided stress_cpu.sh and stress_memory.sh commands first. 

## Download 

Code is available on github in [https://github.com/hmsrc](https://github.com/hmsrc)

## License

Code released under the GNU General Public License.

## Configure

Edit the process_monitor.config file and adjust the contact emails and email message content.  Modify process_monitor.sh or make sure the config file is in the same directory or modify the path to process_monitor.config at the top of process_monitor.sh.

Comment out or delete the first echo "Disabled ..." of process_monitor.sh .  You should reconsider running this program if you don't know how to edit or comment out the line.


## Testing and troubleshooting

* as a non root user, run ./process_monitor.sh and verify it completes successfully
* fix any permissions issues with log, history and temp files
* verify the userid range is valid in process_monitor.config
* un-comment echo "User" line if necessary to confirm processes are being checked
* run stress_cpu.sh, simultaneously in another terminal run process_monitor.sh twice
* verify process detected and killed in the terminal and log file
* check contents of ps output file ${BASE}/process_monitor.out to further troubleshoot
* repeat testing with stress_memory.sh

## Putting into production

The monitor is intended to run as root from cron.  To enable, edit the path in the check_procs cron job and then place it in the /etc/cron.d/ directory.  The default is to run every 5 minutes.  This results in processes being killed after running for anywhere in the range of 5 minutes through 10 minutes depending on what point during the cron interval the process was started.

The script was developed and tested on a debian squeeze environment.  Potential issues might arise with ps output formatting on different linux variants. Email is sent to the user on the system and assumes mail delivery is functioning properly. 

It loops over all users and all processes and should be optimized to instead only check users with running processes.  There is overhead to running the check, but it continues to function in our ~2500 user environment.
 
Enjoy

Gregory Cavanagh  
gregory_cavanagh@hms.harvard.edu  
Research Computing Group, Harvard Medical School  
[http://rc.hms.harvard.edu](http://rc.hms.harvard.edu)  
