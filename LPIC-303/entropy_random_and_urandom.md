Generating random numbers in Linux is important due to **Security Concerns**. Using random data helps in cryptography and tools that use these data as cryptographic key.<br>
So you need to ensure that the key is unpredictable and safe.

## Enropy
Entroy in linux refers to the random data gathered by kernel from the unpredictable events in hardware; like keystrokes, mouse movments, disk I/O.<br>
The random number generator will gather random input from external sources including device drivers into the **Entropy Pool**.<br>
Actually they are entropies that goes into entropy pool which is used by /dev/random and /dev/urandom to provide random data.<br>

## /dev/random & /dev/urandom
Both /dev/random and /dev/urnadom provides cryptographically secure random output generated from interrupt timings or input written to the devices.<br>
More specific they provide an interface to kernel random number generator.

- /dev/random : a block device whcih gives you a high quiality secure random data only when there is enough amount of random input available from entropy pool and if not it blocks all reads to /dev/random till the entropy pool is filled with new random data or enough randomness which then ensures the unpredictability and randomness of data.

- /dev/urandom : urandom(unlimited random) also provides good random data from entropy pool same as the /dev/random. but the only difference here is if there is an insufficient amount of random inputs(entropy pool is empty), it doesn't block reading to /dev/urandom, instead it takes the data from random number generator and generate data with algorithms like MD5, SHA to provide you the needed random data at the time.