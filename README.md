## FCFS Readers-Writers

The easiest (and probably the most inefficient way) to achieve a starve-free solution to the readers-writers problem is to perform **both** reading and writing exclusively. This basically boils down to a first-come-first-serve scheme for providing exclusive access to the file/database and hence is starve-free.

**Initial Conditions**:

* `mutex = 1`

#### Writer Process

```c
wait(mutex);
    // writing is performed
signal(mutex);
```

#### Reader Process

```c
wait(mutex);
    // reading is performed
signal(mutex);
```

## An Improved Solution

The solution presented above is obviously inefficient because it does not make use of the fact that multiple readers may read simultaneously. In this solution, we try to address this shortcoming. We notice that, if at any given instant, only readers have been waiting for access, we can grant access to all of them simultaneously. But as soon as a writer comes up, we must make it wait for all the readers which were granted access earlier to finish their work. 

Although we have overlapped the readers with this solution, it is still, in a sense, a first-come-first-serve scheme. If a writer arrives before a reader, it is given access before the reader (and vice-versa). Thus, the solution remains starve-free.

**Initial Conditions**:

*  `rw_mutex = 1, mutex = 1, reads_completed = 1`

* `readcount = 0`

#### Writer Process

```c
wait(rw_mutex);
    wait(reads_completed);
        // writing is performed
    signal(reads_completed);
signal(rw_mutex);
```

#### Reader Process

```c
wait(rw_mutex);
    wait(mutex); // used for accessing the readcount variable
        readcount++;
        if (readcount == 1) {
            wait(reads_completed); // this will always return immediately
        }    
    signal(mutex);
signal(rw_mutex);
// reading is performed
wait(mutex);
    readcount--;
    if (readcount == 0) {
        signal(reads_completed);
    }
signal(mutex);
```
