# Pancake Robot Transportation Node (iCreate 3)

## Overview

This Python script controls an iRobot Create 3 robot acting as a transporter within an automated pancake cooking system. Its primary job is to fetch pancake orders from a central system (using Airtable), navigate between different processing stations (Cooking, Toppings, Pickup) following a line on the floor, and communicate its status and the order's progress back to Airtable.

Think of this robot as the "waiter" or "delivery driver" in your pancake factory. It picks up an order, takes the pancake base to the required cooking and topping stations in the correct sequence, and finally delivers it to the pickup point.

## Functionality in Simple Terms

1.  **Gets Orders:** The robot constantly checks an online spreadsheet (Airtable) to see if there are any new pancake orders waiting.
2.  **Plans the Trip:** Once it gets an order, it looks at which steps are needed (e.g., Cook 1, Whipped Cream, Pickup). It figures out the route it needs to take based on the physical layout of the stations.
3.  **Follows the Line:** The robot uses two infrared (IR) sensors underneath it to detect and follow a dark line on the floor. It adjusts its steering (turning slightly left or right) to stay centered on the line. If it loses the line, it spins in place to try and find it again.
4.  **Finds Stations:** Each station has a green marker. The robot uses its camera to constantly look for this green color while following the line.
5.  **Knows Which Station is Which:** Since all station markers are green, the robot cleverly figures out which station it's at by *counting* how many green markers it has passed on its current trip. For example, the *first* green marker it sees is Cooking Station 1, the *second* is Cooking Station 2, the *third* is Whipped Cream, and so on, according to a predefined physical sequence.
6.  **Checks if Station is Needed:** When it arrives at a station (detects green), it checks its planned trip: "Does this specific pancake order need something done at *this* station?"
7.  **Updates Status & Waits:**
    * If the station *is* needed for the current order, the robot stops and updates the Airtable spreadsheet to say "Robot Arrived".
    * It then waits patiently, periodically checking Airtable until that station's team updates the status to "Done".
    * If the station *is not* needed for the order, or if it was already marked "Done" when the robot arrived, it skips the waiting step.
8.  **Moves to Next Station:** After a station is completed (or skipped), the robot drives forward a little bit to clear the station marker and then continues following the line, looking for the *next* green marker.
9.  **Completes Order:** Once the robot has visited all the required stations for an order (ending at the Pickup station), it updates Airtable one last time to mark the entire order as finished.
10. **Repeats:** The robot then goes back to looking for new orders in Airtable.

## Key Components

* **Robot:** iRobot Create 3.
* **Control System:** Raspberry Pi running this Python script.
* **Navigation:**
    * **Line Following:** Two IR sensors connected to Raspberry Pi GPIO pins.
    * **Station Detection:** Raspberry Pi Camera V2/V3 using OpenCV (`cv2`) library to detect green color patches.
* **Communication:**
    * **Internal (Robot):** ROS2 (Robot Operating System 2) framework (`rclpy`) to send movement commands (`/cmd_vel`) and play sounds (`/cmd_audio`).
    * **External (System Coordination):** Airtable API (using `requests` library) to fetch orders and update station statuses.
* **State Management:** The script uses a state machine (`RobotState` enum) to keep track of what the robot is currently doing (e.g., `IDLE`, `MOVING_TO_STATION`, `WAITING_FOR_STATION_COMPLETION`).

## How it Works (Simplified Flow)
