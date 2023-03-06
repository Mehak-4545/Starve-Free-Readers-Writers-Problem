# Starve-Free Readers-Writers Problem
In this pseudocode, a starve-free solution to the classical IPC(Interprocess Communication) Problem, the Readers-Writers Problem, has been proposed.
</br>
</br>


## A brief overview of the Readers-Writers problem
This is a standard problem in process synchronization. It is encountered when a dataset like a file or a database has to be shared between several processes at a time. On the basis of the action they perform, these processes are classified into two types, these being, Reader-processes which can only read from the dataset, cannot modify its contents, and Writer-processes which can both read from the dataset as well as write into it.
Allowing multiple readers access to the dataset concurrently is allowed as the contents of the dataset are in a consistent state and no process is trying to update its contents. Thus, readers do not require exclusive access to the "Critical Section".
However, this is not valid for a writer. If a writer executes concurrently with a reader or another writer, then it may result in redundant/wrong value being read by the reader or in a race condition between writers.

In order to counter these problems, we use Semaphores(Binary Semaphores in specific) in the Readers-Writers construct implementation.
The standard solution of readers-writers problem, using semaphores(with priority given to readers) is as follows.

We use two binary semaphores- 'mutex' and 'write', which control the access to Critical Section and ensure Exclusive access of writer process to the Critical Section respectively.
</br>
</br>


## Pseudocode

We have the following:
</br>semaphores -> write, mutex
</br>integer -> read_count


</br>

#### Writer Process</br>
```

void writer(int processID)
{
    do
    {
        //writer requests for CR
        wait(write);


        ********
        //writer performs write
        ********


        //writer leaves CR
        signal(write);
    }
    while(true)
}
```


</br>

#### Reader Process</br>
```

void reader(int processID){
    do{
        //Mutual Exclusion while Reader Process is entering the Critical Section
        wait(mutex);
        read_count++;
        if(read_count==1){
            //Block write operation even if single reader is present
            wait(write);
        }
        signal(mutex);


        ********
        //reader performs read
        ********


        //Mutual Exclusion while reader leaves the Critical Section
        wait(mutex);
        read_count--;
        if(read_count==0){
            //Unblock write operation as there are no readers requesting for read
            signal(write);
        }
        signal(mutex);

    }while(true)
}
```

</br>

## Starvation of Writers

However, yet another problem arises in this implementation, the problem of starvation of writer process. In the above implementation there is a clear prefrence given to reader processes over the writer processes. A write request is serviced only when there are no other readers present. If new reader processes keep entering while there are other readers present in the critical section, and such a situation arises that the CR is never devoid of readers, then it will lead to starvation of writer processes.
</br>
A solution to eliminate starvation is to implement an FCFS-type ie First-Come-First-Serve type scheme.This can be implemented using an additional semaphore 'read' using which the writer process can prevent the reader process which arrive after it from entering the critical section. 
</br>
</br>

## Pseudocode : Starvation free</br>

semaphores -> write, mutex, read</br>
integer -> read_count</br>


</br>

#### Writer Process</br>
```

void writer(int processID){
    do{
        wait(read);
        /*writer immediately blocks 'read' to prevent more readers from entering the Critical Region, in effect, blocking the queue */

        wait(write);    //if readers are reading, writer has to wait


        ********
        //writer performs write
        ********


        //writer leaves CR
        signal(write);
        signal(read);  //writer unblocks 'read'

    }while(true)
}
```


</br>

#### Reader Process</br>
```

void reader(int processID){
    do{

        wait(read);      //new reader allowed only if 'read' semaphore is unblocked
        
        wait(mutex);
        read_count++;
        if(read_count==1){
            wait(write);
        }
        signal(mutex);
        
        signal(read);


        ********
        //reader performs read
        ********


        wait(mutex);
        read_count--;
        if(read_count==0){
            signal(write);
        }
        signal(mutex);

    }while(true)
}
```


</br>

## References used
1. Abraham Silberschatz, Peter B. Galvin, Greg Gagne - Operating System Concepts
2. https://en.wikipedia.org/wiki/Readers%E2%80%93writers_problem

</br>

## Authored By
Mehak Sharma
</br> 21114060
</br> B.Tech 2nd Year, CSE


