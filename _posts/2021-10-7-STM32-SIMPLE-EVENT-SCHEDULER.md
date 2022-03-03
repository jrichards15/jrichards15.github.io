---
title: STM32 Simple Event Scheduler
date: 2021-09-31 08:00:00
layout: page
published: true
---

## Scheduling Important I/O with SysTick

Given the previous two blog posts on using the blocking I/O HAL drivers for I2C to control a screen, and my lack of success in implementing interrupt-driven or DMA-based I/O, I needed to implement a scheme for asynchronous transmission of commands to the HD4770-based LCD. Since I've integrated the UI into a real-time audio project, the vast majority of processing time must be utilized to generate signed 16-bit samples. As such, using a screen with semi-strict timing requirements via an I2C I/O expander can cause a lot of wasted CPU cycles, which is definitely not what we want.

### Using SysTick Interrupts

One of the most straightforward ways of timing operations on an STM32 chip is to use the SysTick interrupt, as it fires every millisecond to update the HAL time-tracking fields. In addition, it's automatically configured by the STM32CubeIDE device configuration tool, which makes this much easier to implement.

An alternate option is to use one of the hardware timers, though the prescaler registers must be set correctly to get the desired time resolution, which was slightly more difficult, though still achieveable. This would allow for finer control over the update rate of the LCD and other peripherals, though for our purposes, millisecond-resolution will be just fine. In the future, modifying the frequency of the interrupts might provide for better functionality, though the blocking nature of the I2C might causes some slight issues and occupy CPU cycles needed for time-critical operations.  

### Implementing the SysTick Scheduler

Fortunately, SysTick gives us some simple tools for implementing time-based scheduling! I didn't want to configure FreeRTOS given the nature of this project, so a simple, custom scheduler was better suited to my purposes. This brings us to the SysTick scheduler!

For our notion of tasks, we will define it fairly simply: a task is an operation that must be run once, at a predetermined time specified by its sleep time. Once run, it can be removed from the execution queue and deleted. There needs to be some way to collect all these tasks in a list somewhere, so they can be executed in order. For my implementation, I represented these requirements in a structure.

![scheduler-struct](/assets/img/scheduler_struct.PNG){:class="img-responsive"}

The struct contains a few vital pieces of information: a function pointer for the operation to be performed, the amount of time that the scheduler should wait before executing the operation, the amount of time the task has already been sleeping, and a small 4-byte buffer containing the arguments to be passed to the function pointer. In the vast majority of cases, the function pointer passed in is a lambda, which will utilize the data in the buffer to form the parameters to the underlying operations. This standardizes the scheduler, allowing it to perform a wide variety of functions without any major adjustments to its underlying code, which facilitates future expansion. 

The scheduler operation is fairly simple. Since there can be an arbitrary number of tasks that must execute, using a fixed-sized array will simply not cut it, so there are a few more things to do before we can schedule our I2C I/O operations. Since my project is using dynamic memory allocation (which can sometimes cause issues on embedded systems), the most straightforward approach is to create a standard linked list structure, where each node holds the required information for each task. Each node in the linked list contains a head pointer, a tail pointer, and a scheduler block struct (shown in the screenshot above). This implementation acts as a queue, where elements are removed from the head of the list and added to the tail, ensuring the desired first-in, first-out execution of tasks. When a task can execute, it will be dequeued, executed, and then deleted to prevent memory issues. For each SysTick interrupt, the scheduler will check the head of the list and increment the amount of time slept. If this time exceeds the specified sleep duration, then the function pointer will be invoked with the arguments in the struct's buffer, and then the node will be removed from the list and destroyed. Before exiting, the scheduler will check the next task to ensure that it is not ready for execution. For tasks with a non-zero sleep duration, the scheduler will exit, since the task has been sleeping for 0 milliseconds. For a task with a sleep duration of zero, it will immediately be dequeued, executed, and destroyed. For the LCD I am using, this is desireable, as the enable pin must be pulsed for the command to take effect. Thus, most I2C operations will be followed with a pair of tasks to pulse the enable pin, which is enqueued as two tasks (one to set the pin high, and another to set it low 1ms later).

As a minor implementation note, the need for some synchronization tools are required. Since the SysTick interrupt can occur at any time, it is critical that the enqueuing and dequeuing of all operations is protected. For this, I utilized a very simple mutex scheme that I hand-implemented as well. It has a simple increment/decrement operation that will prevent an interrupted enqueue operation from corrupting the integrity of the linked list and scheduler. This will delay tasks slightly, though the millisecond-resolution of the interrupts mean that longer waits won't necessarily destroy normal operations for a user.

### Wrapping Up

Given the length of this description, "simple" might have been a little misleading. It works very well in practice, and I have not needed to substantially modify any piece of it while I work on the rest of the project. On the bright side, it does resolve many of the issues I had with writing long strings of text to the LCD while attempting to synthesize realtime audio, which means it completes the goal I needed it to. Overall, it was a worthwile little experiment that I may utilize in future projects.