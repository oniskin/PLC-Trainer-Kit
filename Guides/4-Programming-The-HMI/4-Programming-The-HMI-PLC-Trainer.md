# Introduction.

You have your kit all wired and your PLC is programmed!
You have a working OT system.

*Great job!*

THIS is where things get interesting.

Programming the Ignition HMI is where industrial visualization comes to life. You'll create a user interface for your motor controller that allows operators to control the motor and monitor its usage over time.

The fundamentals you'll learn here are used in power plants, factories, and critical infrastructure worldwide.

In this guide, you'll learn HMI design and programming using Ignition by Inductive Automation.

Watch the [LIVESTREAM](https://www.youtube.com/live/xun7izINi8M) on YouTube!

<img src="thumbnail4.png" width="400" alt="Programming the HMI!" />

# Objectives. What you'll learn.

By the end of this guide, you should understand:
- Ignition HMI design and configuration
- Connecting Ignition to a PLC
- Creating Tags to track process values
- Creating user interfaces for motor control
- Connecting Ignition to MySQL database
- Data logging and Ignition Historian
- Basic SCADA concepts
- How to deploy and run HMI applications

# Prerequisites & Software Setup

Before you start programming, you'll need the Ignition software installed and connected to your hardware.

> [!NOTE]
> See the [Bill of Materials](../BOM.md) for software download links.

## Prerequisites:
- Ubuntu 24.04 Virtual Machine
- Ethernet connection from the machine to your PLC and Ignition Gateway
- Installation file for Ignition by Inductive Automation (Free trial or licensed) — Download from [Inductive Automation](https://inductiveautomation.com/)
- Connection to the Internet to download applications like MySQL

# Guide.

## Part 1: Installing and Setting Up Ignition

### Downloading and Installing Ignition

1. Go to the Inductive Automation website and download the Ignition installer.

Choose *Ignition - Linux Installer 64-bit (1.9 GB)*

2. Run the installer and follow the prompts to install the Ignition Gateway.

3. Start the Ignition Gateway service.

### Accessing the Ignition Designer

1. Open a web browser and navigate to `http://localhost:8088` (or the configured port).
2. Log in with the default credentials (admin/admin).
3. Launch the Designer.

## Part 2: Creating Your First Ignition Project

### Setting Up a New Project

1. In the Designer, create a new project.
2. Configure the project settings.

### Connecting to the PLC

1. Add a device connection to your Click PLC.
2. Configure the IP address and communication settings.
3. Test the connection.

## Part 3: Designing the HMI Interface

### Creating Screens

1. Design a main screen with motor control buttons (Start/Stop).
2. Add indicators for motor status.

### Motor Control Logic

1. Create tags for motor start/stop commands.
2. Bind buttons to write to these tags.
3. Read motor status from PLC tags.

## Part 4: Tracking Motor Usage

### Installing MySQL Community Edition

Before setting up data logging, we need a database to store the historical data. We'll use MySQL Community Edition.

#### Downloading and Installing MySQL

1. Go to the [MySQL Community Downloads](https://dev.mysql.com/downloads/mysql/) page.
2. Download the MySQL Installer for Windows.
3. Run the installer and select "Developer Default" or "Server only" setup type.
4. Follow the installation wizard, setting a root password when prompted.

#### Initial Configuration

1. Open the MySQL Command Line Client or MySQL Workbench.
2. Log in as root.
3. Create a new database for Ignition:
   ```
   CREATE DATABASE ignition;
   ```
4. Create a new user for Ignition:
   ```
   CREATE USER 'ignition_user'@'localhost' IDENTIFIED BY 'password';
   ```
   (Replace 'password' with a secure password)
5. Grant privileges to the user on the database:
   ```
   GRANT ALL PRIVILEGES ON ignition.* TO 'ignition_user'@'localhost';
   ```
6. Flush privileges:
   ```
   FLUSH PRIVILEGES;
   ```
7. Exit the MySQL client.

This sets up MySQL with a dedicated database and user for Ignition to use.

### Data Logging

1. Set up a database connection (we'll be using MySQL).
2. Create transaction groups to log motor run time.
3. Configure historical data storage.

### Creating Charts and Reports

1. Add a chart component to display usage over time.
2. Create reports for motor usage statistics.

## Part 5: Testing & Debugging

### Running Your HMI

1. Publish the project.
2. Open the client and test the interface.
3. Verify motor control and data logging.

### Common Issues & Troubleshooting

- Connection issues with PLC
- Tag configuration errors
- Database setup problems

# Conclusion

You've now learned the fundamentals of HMI programming with Ignition. The concepts you've mastered here scale to complex SCADA systems managing industrial processes.

# Next Steps

Ready to take it further?

Explore advanced Ignition features like scripting, alarming, and multi-user access!

