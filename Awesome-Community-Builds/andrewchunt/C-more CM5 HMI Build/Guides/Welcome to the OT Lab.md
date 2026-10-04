# Welcome to the OT Lab

Understanding Industrial Control Systems Through Hands-On Learning

## Introduction

Welcome to our Operational Technology (OT) Lab! This interactive demonstration provides an opportunity to explore how industrial control systems operate in a manufacturing environment.

OT systems play an essential role in plant and manufacturing operations. These systems monitor and control equipment such as motors, conveyors, pumps, and other machinery that support production processes.

Our OT Lab represents a simplified Industrial Control System (ICS) commonly found in a manufacturing facility. It includes a Programmable Logic Controller (PLC), Human-Machine Interface (HMI), electrical components, physical control buttons, and a fan/motor.

The goal is simple: see how the physical equipment, control logic, and operator controls work together by operating the lab yourself.

## Interactive Lab: Give It a Try!

Follow the steps below to start the fan/motor, stop it, activate the emergency stop, and reset the system.

## Hands-on walkthrough

### 1   Start the Fan / Motor

Your action: Press the green START button.

What to observe: The PLC receives the start command and activates the fan/motor. Observe the fan begin spinning.

What happens when you press START?

[ ] I've completed this step


### 2   Stop the Fan / Motor

Your action: While the fan is running, press the STOP button.

What to observe: The fan/motor stops as the control system responds to the stop command.

How does the system respond to a normal stop?

[ ] I've completed this step


### 3   Restart and Activate the Emergency Stop

Your action: Press START again. Once the fan is running, press the red EMERGENCY STOP button.

What to observe: The fan/motor stops, and the emergency stop button remains physically latched in its pressed position.

What happens if you press START while the emergency stop is still engaged?

[ ] I've completed this step


### 4   Unlatch the Emergency Stop

Your action: Turn the emergency stop button in the indicated direction to release the latch.

What to observe: The button returns to its normal position. The emergency stop is physically released, but the system still requires a reset.

Does releasing the emergency stop automatically restore operation?

[ ] I've completed this step


### 5   Reset the System

Your action: Press the RESET button after releasing the emergency stop.

What to observe: The control system clears the emergency stop condition and returns the lab to its previous operating state, as configured in this demonstration.

Why must you release the emergency stop before RESET can take action?

[ ] I've completed this step


## Understanding the Emergency Stop

Unlike the normal STOP button, the emergency stop is a latching button. Once pressed, it remains engaged until manually released.

The correct sequence is:

1. Press Emergency Stop
The fan stops and the button latches.

2. Turn to Unlatch
Physically release the emergency stop button.

3. Press Reset
Clear the emergency stop condition and restore the lab's configured operating state.


Important: Pressing RESET while the emergency stop remains latched will not clear the condition. You must first turn the emergency stop button to unlatch it.

In this demonstration, RESET returns the lab to its previous state. Actual industrial machinery may require a separate START command after an emergency stop is reset. Never assume that releasing or resetting an emergency stop makes machinery safe to restart.

## Why Does This Matter?

Industrial control systems connect the digital and physical worlds. A command issued through a button, HMI, or PLC can directly affect equipment operating in our plants.

By interacting with the OT Lab, you can see how these systems respond to operator commands, why safety controls are important, and how control logic influences physical machinery.

Remember: OT cybersecurity is not just about protecting computers and networks. It is also about protecting the equipment, processes, and people who depend on them.