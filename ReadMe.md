This repo is for the 4YP project reading the navtech radar data using ROS2 jazzy.

Build the container

Inside the container inside /ament_ws run

    colcon build --symlink-install

After building successfully source the workspace with 

    source install/setup.bash

Then run the radar node with 

    ros2 run nav_radar colossus_publisher --ros-args --params-file /ament_ws/src/iasdk-public/ros/ros2/src/nav_radar/config/colossus_publisher.yaml

For recording data, just run the node above, check if the topics are outputted by running 
    
    ros2 topic list -v
    cd /ament_ws/src 
and start a new recording with 

    ros2 bag record -a

When you want to replay it just run

    ros2 bag play /ament_ws/src/rosbag2_2024_12_05-18_19_42/
    
To look at the data, open a new terminal and run

    ros2 topic echo /radar_data/fft_data

To see the possible ros2 bag play command, type

    ros2 bag play -h

For example, to continuously play the data,

    ros bag play /ament_ws/src/rosbag2_2024_12_05-18_19_42/ -l
    


