# Create A Cron-job That Executes A Python Script In Background

### Overview
It allows you to run shell script or a compiled executable as cron-job.<br>
There is a deamon named **cron** or **crond** which you can check the service by:<br>
`sudo systemctl status cron`

And there is `crontab` that is the command which let you edit the configuration file to add/remove cron-jobs. Open it like:<br>
`crontab -e`<br>

List cronjobs:<br>
`crontab -l`<br>

before that you need to check if there is cron/crond up:<br>
`ps aux | grep cron` <br>
you should see sth like: <br>
`root     	617  0.0  0.0   9420  2800 ?    	Ss   17:00   0:00 /usr/sbin/cron -f`

If there is no cron, install it like: <br>
`sudo apt update && sudo apt install cron`<br>
And enable it: <br>
`sudo systemctl enable cron`

### Syntax
you schadule your cron-jobs using this syntax in config file:
```
*    *    *    *    *   /home/user/bin/somecommand.sh
|    |    |    |    |            |
|    |    |    |    |    Command or Script to execute
|    |    |    |    |
|    |    |    | Day of week(0-6 | Sun-Sat)
|    |    |    |
|    |    |  Month(1-12)
|    |    |
|    |  Day of Month(1-31)
|    |
|   Hour(0-23)
|
Min(0-59)
```

### Implementation of cron-job as py-script
generally to schadule python crontab entry the format is : <br>
`* * * * * /usr/bin/python /path/to/script.py`<br>
e.g. for 5:30 of every day in every month:<br>
`30 5 * * * /usr/bin/python /path/to/script.py`<br>

first check out your VM time:<br> 
`timedatectl`<br> 
here I set Etc/UTC as my linux time.<br>
search for it:<br>
`timedatectl list-timezones | grep -i utc`<br>
grab "Etc/UTC" and run this command to set the time:<br>
`timedatectl set-timezone Etc/UTC`<br>

to schadule open crontab:<br>
`crontab -e`<br>
add this line:<br>
`*/1 * * * * /usr/bin/python3 /home/user/somepy.py >> /home/user/cron_output.log 2>&1`<br>
it'll run the job every minute so you test the py output quickly (change it later).<br>
it redirects the standard output to the file (whenever pyhon prints) and also redirects the standard input to that file so you can always check it.
save and exit.<br>
check out the file to see if it effects evert minute.