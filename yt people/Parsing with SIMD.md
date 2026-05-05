This is actually about the type-1 optimization. So there's gonna be some algorythmic improvement too.

The bottle-neck is dumping the TB of data from the command/kernel, that data bufferization cannot be circumvented, it is under much bigger stress than the rest of the system that only shows a slither of the data on the screen.
And the problem on our end is the parser that is in front of the line-buffer.

What to look for: theroetical throughput.
modern-day bandwidth: 10 - 100 GB/sec, so that should be about what the code should be able to do. So turn off your code to see the throughput of the input pipe. With casey's code turned on the throughput was about 0.5 GB/sec.

> 1.8 GB/sec was roughly the throughtput ceiling imposed by the input pipe. Some bits of what he said might be involved:
> - the buffer size
> - the setup of the pipe (fast pipe)

The first-look idea is it takes too much effort for a CPU to run a command, so you might as well apply SIMD to it. So parsing a byte at a time is sad.
Look at as many bytes as you can at a time.

Win-machine had 16-byte registers. The paralellizable things to be doing in SIMD are:
- look for the escape code
- look for the end of the line `\n`
If you happen to get a control character - call the parser and update the pen. Otherwise - skip to the next 16 bytes.
Casey claims it accounts for 10times increase in performance.


stuff:
- https://uica.uops.info/