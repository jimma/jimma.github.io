---
layout:     post
title:      "Golang misc part2"
subtitle:   "mutex and channel"
date:       2018-01-28 15:11:20
author:     "Jim Ma"
header-img: "img/bg1.jpg"
---
Goroutine is Golang's concurrency programming thread or lightweight thread to run function concurrently. 
From Golang's documentation : 
```
They're called goroutines because the existing terms—threads, coroutines, processes, and so on—convey inaccurate 
connotations. A goroutine has a simple model: it is a function executing concurrently with other goroutines in 
the same address space. It is lightweight, costing little more than the allocation of stack space. And the stacks 
start small, so they are cheap, and grow by allocating (and freeing) heap storage as required.
Goroutines are multiplexed onto multiple OS threads so if one should block, such as while waiting for I/O, others 
continue to run. Their design hides many of the complexities of thread creation and management.

Prefix a function or method call with the go keyword to run the call in a new goroutine. When the call completes, 
the goroutine exits, silently. (The effect is similar to the Unix shell's & notation for running a command in the background.)
```
Unlike the java's thread model based on OS thread, golang has its own lightwieght/costing little thing and can run more threads than java language. Write concurrent programming is simple , the only thing you need to do is add "go" before you function:
```
 go myBalance.Expense(10)
 go myBalacne.Income(10)
```
Simple enough, but you have to think about the thread.join, lock/synchronized thing like other language. Golang can't simpify it for you. In the following content, we'll go through a versy simple example to demonstrate what does Golang's goroutine, mutex and channel look like.
The bank account is always created to talk about mutex, lock and synchronized thing. Here we create the same balance type : 
```
type balanceType struct {
	AccountNo string
	Balance   int
	Mutex     sync.Mutex
}

func NewBalance(accountNo string, balance int) balanceType {
	var mutex = sync.Mutex{}
	myBalance := balanceType{accountNo, balance, mutex}
	return myBalance
}

func (b *balanceType) InitBalance(amount int) int {
	fmt.Println("Init Balance")
	b.Mutex.Lock()
	defer b.Mutex.Unlock()
	b.Balance = amount
	fmt.Println(b.Balance)
	return b.Balance
}

func (b *balanceType) Income(amount int) int {
	b.Mutex.Lock()
	defer b.Mutex.Unlock()
	time.Sleep(700)
	b.Balance = b.Balance + amount
	return b.Balance
}

func (b *balanceType) Expense(amount int) int {
	b.Mutex.Lock()
	defer b.Mutex.Unlock()
	time.Sleep(300)
	b.Balance = b.Balance - amount
	return b.Balance
}

func (b *balanceType) Get() int {
	return b.Balance
}

```
Golang struct type doesn't support default value. So we crate a constructor to add a mutex to each balance type. balance type is minuscule and note exported, it has to be created by NewBalance() function and added with a mutex to lock the balance field.  time.sleep() is added here to 
simulate the account processing time elapsed. This can help test the result without the mutex lock. 
```
    var wg sync.WaitGroup
	myBalance := concurrency.NewBalance("0001", 0)

	myBalance.InitBalance(30)
	fmt.Printf("init balance is : %d \n", myBalance.Get())
	wg.Add(100)

	for i := 0; i < 50; i++ {
		go func() {
			defer wg.Done()
			myBalance.Expense(10)
		}()
		go func() {
			defer wg.Done()
			myBalance.Income(10)
		}()

	}

	wg.Wait()
	fmt.Printf("Current balance is : %d \n", myBalance.Get())
```
When we create balance struct, expense and income same amount money for same times like the above lines code, the final result of balance should be the same with intial value 30. Without mutex, this isn't. We ran these lines code for couple of times, each time you'll get different balance value if we commment the b.Mutex.Lock() and unLock() in balance's fucntion:
```
init balance is : 30
Current balance is : 50
init balance is : 30
Current balance is : 40
init balance is : 30
Current balance is : 70
```
This is not correct to calculate the expense and income. Add back the lock thing, balance will be synchrnozied and the amount of money or points in the balance type won't be lost.  You'll always get the correct result and current balance is 30 after multiple times expense(10) and income(10) invocation.
You'll find we add the sync.WaitGroup to wait the other threads to start and finish, other wise the main finish imediately without wait any of the thread execution is finished. This can be implemented with a channel:
channel is another thing we can use to wait the thread finished , but it needs writing more lines code :
```
    myBalance2 := concurrency.NewBalance("0002", 0)
	myBalance2.InitBalance(100)
	fmt.Printf("init Balance2 is : %d \n", myBalance2.Get())
	done := make(chan bool)
	for i := 0; i < 50; i++ {
		go func() {
			myBalance2.Expense(10)
			done <- true
		}()
		go func() {
			myBalance2.Income(10)
			done <- true
		}()

	}
	for j := 0; j < 100; j++ {
		<-done
	}
	fmt.Printf("Current Balancer2 is : %d \n", myBalance2.Get())
```
channel is created by a make statement and "channel <- sendvalue", the left side value after minus sign will be sent to channel varible. Read the channel value with  " <-channel", it is easy to understand from the arrow direction, the channel value will be sent to ther receiver. 
Drain the channel with  multiple "<-channel" invocation and this can wait all threads are finished.
Both sync.waitGroup and channel can be used to wait the goroutine ending, if I choose I'll use waitgroup to do this job. It's easy to understand and channel more likes a hack to do this thing. 

  

