# HASSRobot

This program was written entirely by me as a Capstone Design Project for Microprocessors course at Delaware Technical Community College.

The Capstone Design Project was entirely my creation. The idea, planning, writing, debugging, and building of the robot were all executed by me.

The program ran on a PIC32MX board and imported the pic32 library for port assignments. The rest of the program initially imported headers of my
own design, but they were all compiled into one program as seen in this repository for the purposes of submission. Each header was written to provide
functionality to the peripherals on the board, including an ultrasonic sensor, line trackers, and servor motors.

# Robot Functionality

The Home Autonomous Security System, or HASS, was designed to operate as a miniature security system that would autonomously operate within its environment. HASS
was given a "safe zone" that was marked with a black line that it would be able to operate within. HASS was designed with two different modes which it could 
alternate between at the push of a button from the user, and upon transition, HASS was capable of traveling between the necessary locations to perform its
operations specific to its operating mode.

## Mode 1: Perimeter Mode

This mode utilized the line trackers onboard HASS to travel autonomously from within the safe zone to the perimeter of the safe zone (marked by a black line). 
Once it reached the perimeter, the line trackers would allow HASS to travel along the perimeter while scanning for intruders using its ultrasonic sensor. If an 
intruder was detected, HASS would alert the user to the presence of the intruder and would not resume operation until the threat was removed from the boundary.

## Mode 2: Scanning Mode

Switching to this mode would prompt HASS to leave the perimeter of the and travel to the center of the safe zone. Here, it would rotate 360 degrees while scanning
for intruders to the safe zone using its ultrasonic sensor. If an intruder was detected by HASS, HASS would halt scanning and would proceed to travel to the
location of the intruder and physically push the intruder outside the boundary of the safe zone. When the threat had been neutralized, HASS would then 
autonomously travel back to the center of the safe zone and resume scanning operations.

