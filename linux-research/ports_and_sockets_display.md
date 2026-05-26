shows all listening ports with their porccess PID: <br>
`netstat -tunlp`<br>
- -t : TPC
- -u : UDP
- -n : numeric form instead of resolved names (displays IP)
- -l : listening ports
- -p :  Show the PID and name of the listener’s process.( this information is shown only if you run as root or sudo)

newer and similar command that functions the same:<br>
`ss -tunlp`

display all TCP + UDP ports with any state: <br>
`ss -tuna`
> -TCP -UDP -numeric -all

display all listening Unix Sockets:<br>
`ss -xl`
> -Unixsockets -listening

display all existing Unix Sockets:<br>
`ss -xa`