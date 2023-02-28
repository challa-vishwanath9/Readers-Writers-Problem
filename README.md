# Readers-Writers-Problem
## Description of the problem
The Readers-Writers Problem is a Synchronization problem in which there are 2 types of users.

- **READERS** - They can read data from a shared file/shared memory location, Multiple of them can access this file/memory location and read at a time.
- **WRITERS** - They can edit the file /data in memory location.

The problem of synchronization occurs when if when writer is in the file and either of the READER or another WRITER accesses the file then this  will cause caution.

## SOLVING SYNCHRONIZATION

We can solve the basic issue of synchronization using semaphores keeping in mind the conditions for properly working .

The semaphores used  are(integers)
```cpp
// global variables 
int writermutex=1; // Semaphore for M.E between Writers
int mutex=1;       // Semaphore for M.E for readercount variable
int readercount=0;// Counter if Readers there or not;

```
WRITERS PROCESS
```cpp
while(true){

P(writemutex);
/**
  writing to file 
**/
V(writemutex);

}
```
READERS PROCESS
```cpp
while(true){
P(mutex);
readercount=readercount+1;
if(readercount==1) P(writemutex);
V(mutex);
/*
 read from file
*/
P(mutex);
readercount=readercount-1;
if(readercount==0) V(writemutex);
V(mutex);
}
```
## The problem in this solution

 In this process function the basic idea is whenever readercount is not equal to zero 
 the signal for process execution will not be given and when a writer is writing the file no other writer can write to the file because writemutex is in wait state . So the condition for the synchronizaton is fulfilled.
 But the writer can't write if readers keep on coming because readercount will never go to 0.
 This problem is called "Starvation for Writers ".

 We will try to solve this problem by using Queue Implementation if the Semaphore.


## SOLUTION

### SEMAPHORE IMPLEMENTATION

The problem in the Synchronization is writers not getting a chance to write while readers are coming .
 
if Readers know that a writer is present and is waiting to write if they signal him to write after completion of present readers the problem will be solved as the readers completed will give chance to writer who is waiting to write and leave.

For this type of implementaion we need a data structure which keeps the order in which persons enter and as soon as the person completes his job he should exit.
 > THE BEST DATA STRUCTURE FOR THIS IS A "QUEQE"

Because queue has the property of FIFO
 we will try to implement semaphore with queue using linked list implementation

PROCESS
```cpp
struct Process{
    Process *next;
    // Contains all details of a Process example Process ID 

}
```

QUEUE-USING LINKED LIST
```cpp
struct queuell{
 process *start,*end;
 queuell(){
    start=nullptr;//empty queue both pointing to null
    end=nullptr;
//constructor of struct
 }
 // a method to add a process 'a' to the queue
 void add( int processid){  
    Process *a=new Process();
    a->ID=processid;
    // In if case we are making end process as the last process
    if(end!=nullptr){
        end->next=a;
end =a;
    }
    // Both start and end will be same in else case
    else{
        start=a;
        end=a;
    }
 }
 // A method for removing the first process in the queue
 Process delete(){
if(start!=nullptr){
    Process *a= start;
    start=start->next;
    // if case is only one process is there in queue we have to make rear =null after removing
    if(start==nullptr){
        end=nullptr;
    }else{
        return a;
    }
}
//Else case is no process ia vailable
else{
    return nullptr;
}
 }
};
```
Finally implementation of SEMAPHORE
```cpp
struct Semaphore{
    int semaphore=1;//intialization
    Semaphore(){
        semaphore=1 //constructor
    }
    Semaphore(int s){
        semaphore=s; 
    }
    
// "Queue to store process 
queuell *Queue =new queuell();
// method to implement P(s) for semaphore i.e blocking the function when all resources are being used
void P(int processid){
semaphore--;
if(semaphore<0){
    Queue->add(processid);
    Process *a=Queue->end;
    block(a);
}
}
// method to implement V(S) for semaphore i.e waking up the process when resources are free
void V(){
semaphore++;
if(semaphore<=0){
    Process a = Queue->delete();
    wakeup(a)
}
}
};
```
## SOLUTION FOR STARVE FREE
The idea we are using is 
lets say there are some readers doing their job if at one time a writer comes in he signals to readers that he is waiting so when the reader is leaving then he signals to writer that you can use File for writing and then no reader will be allowed to enter when writer leaves he signals present readers to check out the file now.
This and using the queue format removes the starvation problem because writers will get a chance to write the file. 

**Variables** we are taking are 
```cpp
Semaphore *semain= new Semaphore(1);//Semaphore for readerscoming in variable
Semaphore *semaout= new Semaphore(1);// semaphore for readersleaving in variable
Semaphore *writesema=new Semaphore();// semaphore for permission to write 

int readercoming =0; // Number of readers reading
int readergoing=0; // Number of readers completed and leaving
bool writerwaiting=false;//says if writeris waiting or not
```

**METHOD FOR READERS**
```cpp
while(true){
   semain->P(processid)
    readercoming++;
    semain->V();
    /*
    *FILE READING
    */
    semaout->P(processid);
    readergoing++;
    if(readercoming==readergoing && writerwaiting) writesema->V();
    semaout->V();

}
```
This gives the signal to writers if writer is and readerscoming and readers going are equal i.e all readers who came before writers completed thier job

## METHOD FOR WRITERS
```cpp
while(true){
    semain->P(processid);
    semaout->P(processid);
    if(readercoming==readergoing) {
    semaout->V();}
else{
   writerwaiting =true;
   semaout->V();
   writesema->P(processid);
   writerwaiting=false;
}
/*
FILE can be written NOW
*/

semain->V();
}
```

 *Finally, This logic makes us sure that there is MUTUAL EXCLUSION and STARVING.*



 ### REFERENCES USED
 - *CONCURRENT CONTROL WITH 'READERS' AND 'WRITERS'*  by Courtios et al.1974
 - Article used [ARTICLE](https://arxiv.org/ftp/arxiv/papers/1309/1309.4507.pdf)
 




