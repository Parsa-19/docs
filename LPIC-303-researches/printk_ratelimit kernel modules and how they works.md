# overview of kernel.printk_ratelimit

despite the security risk of exposer of kernel ring buffer logs or kernel module logs if a situation happens that for example a bad kernel module continiously prink() errors and generate logs, disk can fill up and log messages can be overwritten or lost.<br>
like when the kernel ring buffer overflows and overwrites the initial logs which could be caused by a buggy kernel module.

you can control this buy two kernel parameters:
 - `kernel.printk_ratelimit` 
 - `kernel.printk_ratelimit_burst`

## kernel.printk_ratelimit
this specifies the time interval in seconds between messages which are ratelimited.
during the logging procedure if the rate limiter (which is controled by kernel) deciedes its time to stop logging for e.g. 5 seconds and limit the messages, then the messages which had generated durring this time is not stored then and so less logs are generated.

default is set to 5 seconds:
```
sysctl kernel.printk_ratelimit
```

## kernel.printk_ratelimit_burst
between each limit time interval, it is allowed to generate a specific number of logs which is determined by printk_ratelimit_burst parameter.<br>
for example if it is set to 10. when logs are limited, durring that time it is allowd to generate at the maximum number of 10 logs and then stop storing rest of logs till the limit finishes.

default is set to 10 times:
```
sysctl kernel.printk_ratelimit_burst
```

## limitation of the parameters
these sysctl parameters do not affect every kernel messages and only those modules which are coded like it meant to use the kernel's rate-limiting mechanism. for example:
```
if (printk_ratelimit())
    printk(KERN_WARNING "Error\n");
```
this checks the global rate-limiting state. the output of printk_ratelimit() could be wheather zero or not zeor. if zero it means it is under the limit and doesnt execute the printk() and if it is not zero, it is not under limit and printk() could be executed.

or in one line without checking with a if, you can use macros like:
```
pr_warn_ratelimited("Error\n");
# or
dev_warn_ratelimited("Warning\n");
``` 
they will also obey the sysctl parameter values.

## when the rate limiter decides to limit messages?
this is handled by linux kernel side. kernel uses an algorithm called **token bucket algorithm**.

it works based on rate limit state;<br>
for example the parameters are configured as:
```
kernel.printk_ratelimit = 5
kernel.printk_ratelimit_burst = 10
```
they will create a state like:
```
interval = 5 seconds
burst = 10 tokens

current tokens = 10
window starts = now
```

every time the dunder function `__ratelimit()` is called:
1. check if the rate limit interval (5 seconds) is expired
2. if yes:
    - refill the bucket to 10 tokens again
    - start a new 5 seconds interval
3. else if tokens still remain (5 seconds still not finished):
    - consume and decrease one token
    - and allow the message
4. else if no tokens remain (5 seconds still not finished):
    - do not allow messages and suppress messages