---
title: Sharing I2C with non-preemptive tasks with improved thread scheduling
author: Rohin Razdan
categories:
  - How-To
tags:
  - Creativity
  - Inspiration
  - Habits
---
In this project, the accelerator and the LEDs were peripherals that shared the same I2C bus for communication. The Accelerator movement in x-y-z directions lit rgb respectively.

Initially the code used Busy-Wait for the accelerator which prevents the CPU from doing other tasks, thus the efficiency of operation is compromised initially.Initial code uses Busy-Wait which prevents the CPU from doing other tasks, thus the efficiency of operation is compromised initially.
Why dit we use RTOS?

- RTOS uses more memory
- We may need a lightweight implementation
- This could be transferred to a cheaper board
- May need to add other parts of code in
- RTOS may delay response, But this may not matter based on human eye
How did we fix it: Compare performance between Blocking and FSM
- First: Mode 1 - blocking (checking initial responsiveness, with Busy-Wait)
- Set debug bits and connect logic analyzers to collect data on runtime of each task.
- Second: Mode 2 - FSM w/ run-to-completion scheduler (Without any Busy-Wait OR RTOS)
- Write/configure a FSM mode
- Create CFG of task to plan FSM conversion
- Configure tasks to run periodically
- Create Handshake w/ I2C Msg: (init g_I2C_Msg.Status = IDLE)
- if Status is IDLE return unless Command is READ or WRITE. 
- Update Status to READING or WRITING if needed. 
- Begin I2C & begin FSM.
- Update Status to READ_COMPLETE or WRITE_COMPLETE

FSM Implementation
- Breaks down the TMS into small pieces that can communicate within each other using the shared data structure (g_I2C_Msg). In Busy wait we have to run unnecessary functionality too. But in an FSM functionality which requires the processor is given the priority.

Message format -- 7-bit address followed by data: 

|start|----Slave Device Address--|rd/wr|ack|-------Data Byte------|ack|stop|

|Start|AD7 AD6 AD5 AD4 AD3 AD2 AD1|R/W|ACK|D7 D6 D5 D4 D3 D2 D1 D0|ACK|Stop|


