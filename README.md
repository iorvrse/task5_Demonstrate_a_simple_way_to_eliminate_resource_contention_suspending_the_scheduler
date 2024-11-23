
# Demonstration of Eliminating Resource Contention Using Critical Sections in FreeRTOS

This project demonstrates how to safely access shared resources in a FreeRTOS-based system by implementing **critical sections**. The program manages LED controls using tasks and ensures resource contention is avoided by temporarily suspending the scheduler during critical code execution.

---

## Overview

The system consists of three tasks, each controlling a specific LED (Red, Green, and Blue) while interacting with a shared resource. A custom millisecond delay function is used for precise timing, enhancing system responsiveness.

### Functionality of Each Task and LED

1. **RelLedFlash Task** (Red LED):
   - Toggles the **Red LED** ON and OFF.
   - Accesses the shared resource `lampu` through the `accessData()` function.
   - Represents periodic system actions requiring resource access.

2. **GreenLedFlash Task** (Green LED):
   - Toggles the **Green LED** ON and OFF.
   - Accesses the shared resource `lampu` through the `accessData()` function.
   - Simulates an additional periodic system process with different timing.

3. **Blue LED**:
   - Controlled within the `accessData()` function.
   - *Collision Behavior*: If `accessData()` is interrupted by another task, the shared resource (Blue LED) unintentionally turns ON.

4. **lampu LED**:
   - Flashes to indicate the execution of critical operations.

---

## Key Features

### Critical Section for Resource Access
- The **critical section** is implemented using `taskENTER_CRITICAL()` and `taskEXIT_CRITICAL()` macros.
- The `accessData()` function ensures shared resource access is protected against task preemption.

#### Example: Critical Section Code
```c
void accessData() {
    taskENTER_CRITICAL();  // Begin critical section
    HAL_GPIO_WritePin(lampu_GPIO_Port, lampu_Pin, 1);
    if (flagg == 1) 
        flagg = 0;
    else {
        HAL_GPIO_WritePin(Blue_LED_GPIO_Port, Blue_LED_Pin, 0);
        Delay_ms(100);
        HAL_GPIO_WritePin(Blue_LED_GPIO_Port, Blue_LED_Pin, 1);
    }
    Delay_ms(500);
    flagg = 1;
    HAL_GPIO_WritePin(lampu_GPIO_Port, lampu_Pin, 0);
    taskEXIT_CRITICAL();  // End critical section
}
```

### Custom Millisecond Delay Function
- Replaces `HAL_Delay()` to avoid blocking other operations.
- Utilizes the `DWT` hardware cycle counter for precise timing.

#### Example: Custom Delay Code
```c
void Delay_ms(uint32_t milliseconds) {
    start = millis();  // Record the current time
    while (millis() - start < milliseconds) {
        __NOP();  // Wait until the delay period is over
    }
}
```

---

## Advantages and Disadvantages

### Advantages:
1. **Simplicity**:  
   - Easy to implement using `taskENTER_CRITICAL()` and `taskEXIT_CRITICAL()`.
   - Avoids the need for more complex synchronization mechanisms like semaphores or mutexes.

2. **Guaranteed Data Consistency**:  
   - Prevents race conditions by ensuring exclusive access to shared resources.

3. **Efficiency**:  
   - Suitable for short critical sections where suspending the scheduler is less overhead than using other methods.

4. **Deterministic Behavior**:  
   - Ensures predictable execution of the critical section, especially in real-time systems.

### Disadvantages:
1. **Task Blocking**:  
   - Suspending the scheduler prevents all other tasks from running, which may lead to missed deadlines in time-critical tasks.

2. **Reduced System Responsiveness**:  
   - Extended critical sections can delay task scheduling, potentially affecting real-time performance.

3. **Not Scalable**:  
   - Becomes inefficient as the system grows with more tasks and shared resources.

---

## Task Functions and LED Roles

| Task Name         | LED Controlled | LED Role                               | Task Function                                     |
|--------------------|----------------|----------------------------------------|--------------------------------------------------|
| **RelLedFlash**    | Red LED        | Signals periodic task operation        | Toggles Red LED and accesses `accessData()`.            |
| **GreenLedFlash**  | Green LED      | Indicates another system process       | Toggles Green LED and accesses `accessData()`.          |
| **accessData**     | Blue LED       | Indicates critical section execution   | Flashes briefly during resource access.          |

---

## Example Task Implementation

### RelLedFlash Task (Red LED)
```c
for(;;) {
    HAL_GPIO_WritePin(Red_LED_GPIO_Port, Red_LED_Pin, 1);
    accessData();  // Critical section
    HAL_GPIO_WritePin(Red_LED_GPIO_Port, Red_LED_Pin, 0);
    osDelay(100);  // Wait before the next iteration
}
```

### GreenLedFlash Task (Green LED)
```c
for(;;) {
    HAL_GPIO_WritePin(Green_LED_GPIO_Port, Green_LED_Pin, 1);
    accessData();  // Critical section
    HAL_GPIO_WritePin(Green_LED_GPIO_Port, Green_LED_Pin, 0);
    osDelay(500);  // Wait before the next iteration
}
```

---

## To-Do for Final Report

- [ ] **Photos of the Hardware Setup**:
  ![IMG-20241123-WA0006 1](https://github.com/user-attachments/assets/559649af-bc77-478b-aa78-45e25e944587)


- [ ] **Video of the Experiment**:

https://github.com/user-attachments/assets/2b6bfcc4-d5e7-4f9c-b0a5-ecf1e36ad8bc

---

## Summary

This project demonstrates an effective way to handle resource contention in FreeRTOS by using critical sections. While this method is simple and ensures data consistency, it may not be ideal for larger systems with multiple time-critical tasks. Custom timing functions and structured task design further enhance the system’s reliability and maintainability.

This program is well-suited for embedded systems with limited shared resources and where real-time performance is critical for smaller task loads.
