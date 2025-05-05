# 🥞🤖 Pancake Robot Transportation Using a iRobot Create 3 Robot

## 📝 Overview

This Python script controls an iRobot Create 3 robot, acting as a transporter 🚚 within an automated pancake cooking system. Its primary job is to fetch pancake orders from a central system (using Airtable ☁️), navigate between different processing stations (🍳 Cooking, 🍦 Toppings, 🏁 Pickup) following a line on the floor 〰️, and communicate its status and the order's progress back to Airtable.

Think of this robot as the "waiter" or "delivery driver" 🤖 in your pancake factory. It picks up an order, takes the pancake base to the required cooking and topping stations in the correct sequence, and finally delivers it to the pickup point.

## ⚙️ How it Works

1.  **📥 Gets Orders:** The robot constantly checks an online spreadsheet (Airtable) to see if there are any new pancake orders waiting.
2.  **🗺️ Plans the Trip:** Once it gets an order, it looks at which steps are needed (e.g., Cook 1, Whipped Cream, Pickup). It figures out the route it needs to take based on the physical layout of the stations.
3.  **〰️ Follows the Line:** The robot uses two infrared (IR) sensors underneath it to detect and follow a dark line on the floor. It adjusts its steering (turning slightly left or right) to stay centered on the line. If it loses the line, it spins 🔄 in place to try and find it again.
4.  **🟩 Finds Stations:** Each station has a green marker. The robot uses its camera 📷 to constantly look for this green color while following the line.
5.  **🔢 Knows Which Station is Which:** Since all station markers are green, the robot cleverly figures out which station it's at by *counting* how many green markers it has passed on its current trip. For example, the *first* green marker it sees is Cooking Station 1, the *second* is Cooking Station 2, the *third* is Whipped Cream, and so on, according to a predefined physical sequence.
6.  **✅ Checks if Station is Needed:** When it arrives at a station (detects green), it checks its planned trip: "Does this specific pancake order need something done at *this* station?"
7.  **⏳ Updates Status & Waits:**
    * If the station *is* needed for the current order, the robot stops 🛑 and updates the Airtable spreadsheet to say "Robot Arrived".
    * It then waits patiently, periodically checking Airtable until that station's team updates the status to "Done" 👍.
    * If the station *is not* needed for the order, or if it was already marked "Done" when the robot arrived, it skips the waiting step.
8.  **➡️ Moves to Next Station:** After a station is completed (or skipped), the robot drives forward a little bit to clear the station marker and then continues following the line, looking for the *next* green marker.
9.  **🏁 Completes Order:** Once the robot has visited all the required stations for an order (ending at the Pickup station), it updates Airtable one last time to mark the entire order as finished 🎉.
10. **🔄 Repeats:** The robot then goes back to looking for new orders in Airtable.

## 🧩 Key Components

* **Robot 🤖:** iRobot Create 3.
* **Control System 💻:** Raspberry Pi running this Python script.
* **Navigation 🗺️:**
    * **Line Following 〰️:** Two IR sensors connected to Raspberry Pi GPIO pins.
    * **Station Detection 🟩📷:** Raspberry Pi Camera V2/V3 using OpenCV (`cv2`) library to detect green color patches.
* **Communication 📡:**
    * **Internal (Robot) 🤖↔️💻:** ROS2 (Robot Operating System 2) framework (`rclpy`) to send movement commands (`/cmd_vel`) and play sounds (`/cmd_audio`).
    * **External (System Coordination) ☁️📊:** Airtable API (using `requests` library) to fetch orders and update station statuses.
* **State Management 🚦:** The script uses a state machine (`RobotState` enum) to keep track of what the robot is currently doing (e.g., `IDLE`, `MOVING_TO_STATION`, `WAITING_FOR_STATION_COMPLETION`).

## ⚙️ Configuration

* **Airtable Credentials 🔑:** API Token, Base ID, and Table Name **must** be set in a `.env` file in the same directory as the script. The script uses the `python-dotenv` library to load these.
    ```dotenv
    AIRTABLE_API_TOKEN=your_api_token_here
    AIRTABLE_BASE_ID=your_base_id_here
    AIRTABLE_TABLE_NAME=your_table_name_here
    ```
* **Hardware Pins 📌:** GPIO pins for the Left and Right IR sensors are defined near the top (`LEFT_IR_PIN`, `RIGHT_IR_PIN`).
* **Navigation Parameters 🎛️:** Speeds, turning factors, color detection sensitivity, and timeouts are configured as constants near the top of the script.
* **Station Sequence 📍🗺️:** The exact physical order the robot encounters the stations *must* be correctly defined in `PHYSICAL_STATION_SEQUENCE_INDICES`. The mapping between Airtable field names and these indices is defined in `STATION_FIELD_TO_INDEX`.

## 🛠️ Hardware Setup

* An iRobot Create 3 🤖 connected to a Raspberry Pi 🔌.
* Two IR line-following sensors 👀 connected to the specified GPIO pins on the Raspberry Pi.
* A Raspberry Pi Camera 📷 connected and configured, pointing forward/down to see the line and station markers.
* A floor surface 🗺️ with a clearly marked dark line ⚫ and distinct green patches 🟩 at each station location.

## 📦🐍 Software Dependencies

Make sure you have the necessary Python libraries installed:

* `rclpy` (Part of your ROS2 installation)
* `irobot_create_msgs` (Part of Create 3 ROS2 packages)
* `opencv-python` (`pip install opencv-python`)
* `numpy` (`pip install numpy`)
* `python-dotenv` (`pip install python-dotenv`)
* `requests` (`pip install requests`)
* `RPi.GPIO` (`pip install RPi.GPIO`)
* `picamera2` (`pip install picamera2`) - May require additional system setup on Raspberry Pi OS.
* `libcamera` (Usually installed alongside `picamera2`/system libraries)

## ▶️ Running the Code

1.  **📁 Set up Environment:** Create the `.env` file with your Airtable credentials.
2.  **🌐 Source ROS2:** Ensure your ROS2 environment is sourced (e.g., `source /opt/ros/<your_ros_distro>/setup.bash`, `source install/setup.bash` if in a workspace).
3.  **🚀 Run the Node:** Execute the script directly using Python:
    ```bash
    python3 path/to/your/pancake_robot_node.py
    ```
    (Replace `path/to/your/` with the actual path).

## ⚠️ Important Notes

* **Single Color Detection 🎨:** The system relies heavily on *counting* the green markers it passes. If the robot gets lost or misses a marker count, it might go to the wrong station.
* **Line Following 〰️:** Assumes a relatively simple path without sharp intersections that might confuse the IR sensors.
* **Error Handling ❌:** Includes basic error states for hardware failures (GPIO, Camera) or communication issues (Airtable timeouts), which will typically stop the robot.
