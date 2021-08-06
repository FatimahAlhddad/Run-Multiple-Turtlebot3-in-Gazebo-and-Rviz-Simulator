# Run-Multiple-Turtlebot3-in-Gazebo-and-Rviz-Simulator
This repository shows the steps taken to run more than one of the Turtlebot3 on the Gazebo and Rviz using navigation

###### Requirements:
1. Ubuntu version 18.04
2. ROS melodic
3. Saved SLAM map

## (1) Make one_robot.launch file and paste the code below:
~~~
<launch>
    <arg name="robot_name"/>
    <arg name="init_pose"/>

    <node name="spawn_minibot_model" pkg="gazebo_ros" type="spawn_model"
     args="$(arg init_pose) -urdf -param /robot_description -model $(arg robot_name)"
     respawn="false" output="screen" />

    <node pkg="robot_state_publisher" type="state_publisher"
          name="robot_state_publisher" output="screen"/>
</launch>
~~~

## (2) Make robots.launch file and paste the code below:
```
<launch>
  <!-- No namespace here as we will share this description.
       Access with slash at the beginning -->
  <param name="robot_description"
    command="$(find xacro)/xacro.py $(find turtlebot_description)/robots/kobuki_hexagons_asus_xtion_pro.urdf.xacro" />

  <!-- BEGIN ROBOT 1-->
  <group ns="robot1">
    <param name="tf_prefix" value="robot1_tf" />
    <include file="$(find multiple_turtlebots_sim)/launch/one_robot.launch" >
      <arg name="init_pose" value="-x 3 -y 1 -z 0" />
      <arg name="robot_name"  value="Robot1" />
    </include>
  </group>

  <!-- BEGIN ROBOT 2-->
  <group ns="robot2">
    <param name="tf_prefix" value="robot2_tf" />
    <include file="$(find multiple_turtlebots_sim)/launch/one_robot.launch" >
      <arg name="init_pose" value="-x -4 -y 1 -z 0" />
      <arg name="robot_name"  value="Robot2" />
    </include>
  </group>

</launch>
```

## (3) Make main.launch file and paste the code below:
```
<launch>
  <param name="/use_sim_time" value="true" />

  <!-- start world -->
  <node name="gazebo" pkg="gazebo_ros" type="gazebo" 
   args="$(find turtlebot3_gazebo)/worlds/turtlebot3_world.world" respawn="false" output="screen" />

  <!-- start gui -->
  <!-- <node name="gazebo_gui" pkg="gazebo" type="gui" respawn="false" output="screen"/> -->

  <!-- include our robots -->
  <include file="$(find multiple_turtlebots_sim)/launch/robots.launch"/>
</launch>
```
## (4) Make navigation.launch file and paste the code below:
(map_server)
```
<launch>
  <param name="/use_sim_time" value="true"/>

  <!-- Run the map server -->
  <node name="map_server" pkg="map_server" type="map_server" args="/home/codebind/map.yaml" >
    <param name="frame_id" value="/map" />
  </node>

  <node type="rviz" name="rviz" pkg="rviz"  />

</launch>
```

## (5) Now launch turtlebots navigation file:
```
$ roslaunch multiple_turtlebots_sim navigation.launch
```
## (5-1) Open the map:
1. Click ***Add*** button 
2. Select ***map*** then add it
3. go to map and in the ***topic*** section write ***/map***


## (6)Make amcl launch files and paste the code below:
## (6-1) amcl_robot1.launch
```
<launch>

  <arg name="use_map_topic"   default="false"/>
  <arg name="scan_topic"      default="/robot1/kobuki/laser/scan"/>
  <arg name="initial_pose_x"  default="3.0"/>
  <arg name="initial_pose_y"  default="1.0"/>
  <arg name="initial_pose_a"  default="0.0"/>
  <arg name="odom_frame_id"   default="/robot1_tf/odom"/>
  <arg name="base_frame_id"   default="/robot1_tf/base_footprint"/>
  <arg name="global_frame_id" default="/map"/>

  <node pkg="amcl" type="amcl" name="robot1_amcl">
    <param name="use_map_topic"             value="$(arg use_map_topic)"/>
    <!-- Publish scans from best pose at a max of 10 Hz -->
    <param name="odom_model_type"           value="diff"/>
    <param name="odom_alpha5"               value="0.1"/>
    <param name="gui_publish_rate"          value="10.0"/>
    <param name="laser_max_beams"             value="60"/>
    <param name="laser_max_range"           value="12.0"/>
    <param name="min_particles"             value="500"/>
    <param name="max_particles"             value="2000"/>
    <param name="kld_err"                   value="0.05"/>
    <param name="kld_z"                     value="0.99"/>
    <param name="odom_alpha1"               value="0.2"/>
    <param name="odom_alpha2"               value="0.2"/>
    <!-- translation std dev, m -->
    <param name="odom_alpha3"               value="0.2"/>
    <param name="odom_alpha4"               value="0.2"/>
    <param name="laser_z_hit"               value="0.5"/>
    <param name="laser_z_short"             value="0.05"/>
    <param name="laser_z_max"               value="0.05"/>
    <param name="laser_z_rand"              value="0.5"/>
    <param name="laser_sigma_hit"           value="0.2"/>
    <param name="laser_lambda_short"        value="0.1"/>
    <param name="laser_model_type"          value="likelihood_field"/>
    <!-- <param name="laser_model_type" value="beam"/> -->
    <param name="laser_likelihood_max_dist" value="2.0"/>
    <param name="update_min_d"              value="0.25"/>
    <param name="update_min_a"              value="0.2"/>
    <param name="odom_frame_id"             value="$(arg odom_frame_id)"/>
    <param name="base_frame_id"             value="$(arg base_frame_id)"/>
    <param name="global_frame_id"           value="$(arg global_frame_id)"/>
    <param name="resample_interval"         value="1"/>
    <!-- Increase tolerance because the computer can get quite busy -->
    <param name="transform_tolerance"       value="1.0"/>
    <param name="recovery_alpha_slow"       value="0.0"/>
    <param name="recovery_alpha_fast"       value="0.0"/>
    <param name="initial_pose_x"            value="$(arg initial_pose_x)"/>
    <param name="initial_pose_y"            value="$(arg initial_pose_y)"/>
    <param name="initial_pose_a"            value="$(arg initial_pose_a)"/>
    <remap from="scan"                      to="$(arg scan_topic)"/>
    <remap from="initialpose"               to="/robot1/initialpose"/>
    <remap from="amcl_pose"                 to="/robot1/amcl_pose"/>
    <remap from="particlecloud"             to="/robot1/particlecloud"/>

  </node>

</launch>
```
## (6-2) amcl_robot2.launch
```
<launch>

  <arg name="use_map_topic"   default="false"/>
  <arg name="scan_topic"      default="/robot2/kobuki/laser/scan"/>
  <arg name="initial_pose_x"  default="-4.0"/>
  <arg name="initial_pose_y"  default="1.0"/>
  <arg name="initial_pose_a"  default="0.0"/>
  <arg name="odom_frame_id"   default="robot2_tf/odom"/>
  <arg name="base_frame_id"   default="robot2_tf/base_footprint"/>
  <arg name="global_frame_id" default="/map"/>

  <node pkg="amcl" type="amcl" name="robot2_amcl">
    <param name="use_map_topic"             value="$(arg use_map_topic)"/>
    <!-- Publish scans from best pose at a max of 10 Hz -->
    <param name="odom_model_type"           value="diff"/>
    <param name="odom_alpha5"               value="0.1"/>
    <param name="gui_publish_rate"          value="10.0"/>
    <param name="laser_max_beams"             value="60"/>
    <param name="laser_max_range"           value="12.0"/>
    <param name="min_particles"             value="500"/>
    <param name="max_particles"             value="2000"/>
    <param name="kld_err"                   value="0.05"/>
    <param name="kld_z"                     value="0.99"/>
    <param name="odom_alpha1"               value="0.2"/>
    <param name="odom_alpha2"               value="0.2"/>
    <!-- translation std dev, m -->
    <param name="odom_alpha3"               value="0.2"/>
    <param name="odom_alpha4"               value="0.2"/>
    <param name="laser_z_hit"               value="0.5"/>
    <param name="laser_z_short"             value="0.05"/>
    <param name="laser_z_max"               value="0.05"/>
    <param name="laser_z_rand"              value="0.5"/>
    <param name="laser_sigma_hit"           value="0.2"/>
    <param name="laser_lambda_short"        value="0.1"/>
    <param name="laser_model_type"          value="likelihood_field"/>
    <!-- <param name="laser_model_type" value="beam"/> -->
    <param name="laser_likelihood_max_dist" value="2.0"/>
    <param name="update_min_d"              value="0.25"/>
    <param name="update_min_a"              value="0.2"/>
    <param name="odom_frame_id"             value="$(arg odom_frame_id)"/>
    <param name="base_frame_id"             value="$(arg base_frame_id)"/>
    <param name="global_frame_id"           value="$(arg global_frame_id)"/>
    <param name="resample_interval"         value="1"/>
    <!-- Increase tolerance because the computer can get quite busy -->
    <param name="transform_tolerance"       value="1.0"/>
    <param name="recovery_alpha_slow"       value="0.0"/>
    <param name="recovery_alpha_fast"       value="0.0"/>
    <param name="initial_pose_x"            value="$(arg initial_pose_x)"/>
    <param name="initial_pose_y"            value="$(arg initial_pose_y)"/>
    <param name="initial_pose_a"            value="$(arg initial_pose_a)"/>
    <remap from="scan"                      to="$(arg scan_topic)"/>
    <remap from="initialpose"               to="/robot2/initialpose"/>
    <remap from="amcl_pose"                 to="/robot2/amcl_pose"/>
    <remap from="particlecloud"             to="/robot2/particlecloud"/>

  </node>

</launch>
```

## (7) Modify navigation.launch by substituting this code:
```
<launch>
  <param name="/use_sim_time" value="true"/>

  <!-- Run the map server -->
  <node name="map_server" pkg="map_server" type="map_server" args="/home/codebind/map.yaml" >
    <param name="frame_id" value="/map" />
  </node>



 <!-- Properties of each robot      -->
    
   <!-- AMCL      -->

    <include file="$(find multiple_turtlebots_sim)/launch/amcl_robot1.launch" />
    <include file="$(find multiple_turtlebots_sim)/launch/amcl_robot2.launch" />

    

<!-- Launching Rviz      -->

  <node name="rviz" pkg="rviz" type="rviz" args="-d $/opt/ros/melodic/share/rviz/default.rviz" />

</launch>
```
## (8) Launch the move base for each robot
## (8-1) Make move_base1.launch and paste the code below:
```
<!--
    ROS navigation stack with velocity smoother and safety (reactive) controller
-->
<launch>


  <include file="$(find turtlebot_navigation)/launch/includes/velocity_smoother.launch.xml"/>
  <include file="$(find turtlebot_navigation)/launch/includes/safety_controller.launch.xml"/>

  <arg name="odom_frame_id"   default="robot1_tf/odom"/>
  <arg name="base_frame_id"   default="robot1_tf/base_footprint"/>
  <arg name="global_frame_id" default="/map"/>
  <arg name="odom_topic" default="/robot1/odom" />
  <arg name="laser_topic" default="/robot1/kobuki/laser/scan" />
  <arg name="custom_param_file" default="$(find turtlebot_navigation)/param/dummy.yaml"/>

  <node pkg="move_base" type="move_base" respawn="false" name="move_base" output="screen">
    <rosparam file="$(find turtlebot_navigation)/param/costmap_common_params.yaml" command="load" ns="global_costmap" />
    <rosparam file="$(find turtlebot_navigation)/param/costmap_common_params.yaml" command="load" ns="local_costmap" />
    <rosparam file="$(find turtlebot_navigation)/param/local_costmap_params.yaml" command="load" />
    <rosparam file="$(find turtlebot_navigation)/param/global_costmap_params.yaml" command="load" />
    <rosparam file="$(find turtlebot_navigation)/param/dwa_local_planner_params.yaml" command="load" />
    <rosparam file="$(find turtlebot_navigation)/param/move_base_params.yaml" command="load" />
    <rosparam file="$(find turtlebot_navigation)/param/global_planner_params.yaml" command="load" />
    <rosparam file="$(find turtlebot_navigation)/param/navfn_global_planner_params.yaml" command="load" />
    <!-- external params file that could be loaded into the move_base namespace -->
    <rosparam file="$(arg custom_param_file)" command="load" />

    <!-- reset frame_id parameters using user input data -->
    <param name="global_costmap/global_frame" value="$(arg global_frame_id)"/>
    <param name="global_costmap/robot_base_frame" value="$(arg base_frame_id)"/>
    <param name="local_costmap/global_frame" value="$(arg odom_frame_id)"/>
    <param name="local_costmap/robot_base_frame" value="$(arg base_frame_id)"/>
    <param name="DWAPlannerROS/global_frame_id" value="$(arg odom_frame_id)"/>

    <remap from="cmd_vel" to="/robot1/cmd_vel"/>
    <remap from="odom" to="$(arg odom_topic)"/>
    <remap from="scan" to="$(arg laser_topic)"/>
    <remap from="map" to="/map" />
    <remap from="/move_base_simple/goal"                                           to="/robot1/move_base_simple/goal" />
    <remap from="/move_base/TebLocalPlannerROS/global_plan"                        to="/robot1/move_base/TebLocalPlannerROS/global_plan" />
    <remap from="/move_base/TebLocalPlannerROS/local_plan"                         to="/robot1/move_base/TebLocalPlannerROS/local_plan" />
    <remap from="/move_base/TebLocalPlannerROS/teb_markers"                        to="/robot1/move_base/TebLocalPlannerROS/teb_markers" />
    <remap from="/move_base/TebLocalPlannerROS/teb_markers_array"                  to="/robot1/move_base/TebLocalPlannerROS/teb_markers_array" />
    <remap from="/move_base/TebLocalPlannerROS/teb_poses"                          to="/robot1/move_base/TebLocalPlannerROS/teb_poses" />
    <remap from="/move_base/global_costmap/costmap"                                to="/robot1/move_base/global_costmap/costmap" />
    <remap from="/move_base/global_costmap/costmap_updates"                        to="/robot1/move_base/global_costmap/costmap_updates" />
    <remap from="/move_base/local_costmap/costmap"                                 to="/robot1/move_base/local_costmap/costmap" />
    <remap from="/move_base/local_costmap/costmap_updates"                         to="/robot1/move_base/local_costmap/costmap_updates" />
    <remap from="/move_base/local_costmap/footprint"                               to="/robot1/move_base/local_costmap/footprint" />

    <remap from="/move_base/GlobalPlanner/parameter_descriptions"                  to="/robot1/move_base/GlobalPlanner/parameter_descriptions" />
    <remap from="/move_base/GlobalPlanner/parameter_updates"                       to="/robot1/move_base/GlobalPlanner/parameter_updates" />
    <remap from="/move_base/GlobalPlanner/plan"                                    to="/robot1/move_base/GlobalPlanner/plan" />
    <remap from="/move_base/GlobalPlanner/potential"                               to="/robot1/move_base/GlobalPlanner/potential" />
    <remap from="/move_base/TebLocalPlannerROS/obstacles"                          to="/robot1/move_base/TebLocalPlannerROS/obstacles" />
    <remap from="/move_base/TebLocalPlannerROS/parameter_descriptions"             to="/robot1/move_base/TebLocalPlannerROS/parameter_descriptions" />
    <remap from="/move_base/TebLocalPlannerROS/parameter_updates"                  to="/robot1/move_base/TebLocalPlannerROS/parameter_updates" />
    <remap from="/move_base/cancel"                                                to="/robot1/move_base/cancel" />
    <remap from="/move_base/current_goal"                                          to="/robot1/move_base/current_goal" />
    <remap from="/move_base/feedback"                                              to="/robot1/move_base/feedback" />
    <remap from="/move_base/global_costmap/footprint"                              to="/robot1/move_base/global_costmap/footprint" />
    <remap from="/move_base/global_costmap/inflation_layer/parameter_descriptions" to="/robot1/move_base/global_costmap/inflation_layer/parameter_descriptions" />
    <remap from="/move_base/global_costmap/inflation_layer/parameter_updates"      to="/robot1/move_base/global_costmap/inflation_layer/parameter_updates" />
    <remap from="/move_base/global_costmap/obstacle_layer/clearing_endpoints"      to="/robot1/move_base/global_costmap/obstacle_layer/clearing_endpoints" />
    <remap from="/move_base/global_costmap/obstacle_layer/parameter_descriptions"  to="/robot1/move_base/global_costmap/obstacle_layer/parameter_descriptions" />
    <remap from="/move_base/global_costmap/obstacle_layer/parameter_updates"       to="/robot1/move_base/global_costmap/obstacle_layer/parameter_updates" />
    <remap from="/move_base/global_costmap/parameter_descriptions"                 to="/robot1/move_base/global_costmap/parameter_descriptions" />
    <remap from="/move_base/global_costmap/parameter_updates"                      to="/robot1/move_base/global_costmap/parameter_updates" />
    <remap from="/move_base/global_costmap/static_layer/parameter_descriptions"    to="/robot1/move_base/global_costmap/static_layer/parameter_descriptions" />
    <remap from="/move_base/global_costmap/static_layer/parameter_updates"         to="/robot1/move_base/global_costmap/static_layer/parameter_updates" />
    <remap from="/move_base/goal"                                                  to="/robot1/move_base/goal" />
    <remap from="/move_base/local_costmap/obstacle_layer/parameter_descriptions"   to="/robot1/move_base/local_costmap/obstacle_layer/parameter_descriptions" />
    <remap from="/move_base/local_costmap/obstacle_layer/parameter_updates"        to="/robot1/move_base/local_costmap/obstacle_layer/parameter_updates" />
    <remap from="/move_base/local_costmap/parameter_descriptions"                  to="/robot1/move_base/local_costmap/parameter_descriptions" />
    <remap from="/move_base/local_costmap/parameter_updates"                       to="/robot1/move_base/local_costmap/parameter_updates" />
    <remap from="/move_base/local_costmap/static_layer/parameter_descriptions"     to="/robot1/move_base/local_costmap/static_layer/parameter_descriptions" />
    <remap from="/move_base/local_costmap/static_layer/parameter_updates"          to="/robot1/move_base/local_costmap/static_layer/parameter_updates" />
    <remap from="/move_base/parameter_descriptions"                                to="/robot1/move_base/parameter_descriptions" />
    <remap from="/move_base/parameter_updates"                                     to="/robot1/move_base/parameter_updates" />
    <remap from="/move_base/result"                                                to="/robot1/move_base/result" />
    <remap from="/move_base/status"                                                to="/robot1/move_base/status" />
    <remap from="/move_base_simple/goal"                                           to="/robot1/move_base_simple/goal" />
  </node>


</launch>
```
## (8-2) Make move_base2.launch and paste the code below:
```
<!--
    ROS navigation stack with velocity smoother and safety (reactive) controller
-->
<launch>


  <arg name="odom_frame_id"   default="robot2_tf/odom"/>
  <arg name="base_frame_id"   default="robot2_tf/base_footprint"/>
  <arg name="global_frame_id" default="map"/>
  <arg name="odom_topic" default="/robot2_tf/odom" />
  <arg name="laser_topic" default="/robot2/kobuki/laser/scan" />
  <arg name="custom_param_file" default="$(find turtlebot_navigation)/param/dummy.yaml"/>

  <node pkg="move_base" type="move_base" respawn="false" name="move_base2" output="screen">
    <rosparam file="$(find turtlebot_navigation)/param/costmap_common_params.yaml" command="load" ns="global_costmap" />
    <rosparam file="$(find turtlebot_navigation)/param/costmap_common_params.yaml" command="load" ns="local_costmap" />
    <rosparam file="$(find turtlebot_navigation)/param/local_costmap_params.yaml" command="load" />
    <rosparam file="$(find turtlebot_navigation)/param/global_costmap_params.yaml" command="load" />
    <rosparam file="$(find turtlebot_navigation)/param/dwa_local_planner_params.yaml" command="load" />
    <rosparam file="$(find turtlebot_navigation)/param/move_base_params.yaml" command="load" />
    <rosparam file="$(find turtlebot_navigation)/param/global_planner_params.yaml" command="load" />
    <rosparam file="$(find turtlebot_navigation)/param/navfn_global_planner_params.yaml" command="load" />
    <!-- external params file that could be loaded into the move_base namespace -->
    <rosparam file="$(arg custom_param_file)" command="load" />

    <!-- reset frame_id parameters using user input data -->
    <param name="global_costmap/global_frame" value="$(arg global_frame_id)"/>
    <param name="global_costmap/robot_base_frame" value="$(arg base_frame_id)"/>
    <param name="local_costmap/global_frame" value="$(arg odom_frame_id)"/>
    <param name="local_costmap/robot_base_frame" value="$(arg base_frame_id)"/>
    <param name="DWAPlannerROS/global_frame_id" value="$(arg odom_frame_id)"/>

    <remap from="cmd_vel" to="/robot2/cmd_vel"/>
    <remap from="odom" to="$(arg odom_topic)"/>
    <remap from="scan" to="$(arg laser_topic)"/>
    <remap from="map" to="/map" />
    <remap from="/move_base_simple/goal"                                           to="/robot2/move_base_simple/goal" />
    <remap from="/move_base/TebLocalPlannerROS/global_plan"                        to="/robot2/move_base/TebLocalPlannerROS/global_plan" />
    <remap from="/move_base/TebLocalPlannerROS/local_plan"                         to="/robot2/move_base/TebLocalPlannerROS/local_plan" />
    <remap from="/move_base/TebLocalPlannerROS/teb_markers"                        to="/robot2/move_base/TebLocalPlannerROS/teb_markers" />
    <remap from="/move_base/TebLocalPlannerROS/teb_markers_array"                  to="/robot2/move_base/TebLocalPlannerROS/teb_markers_array" />
    <remap from="/move_base/TebLocalPlannerROS/teb_poses"                          to="/robot2/move_base/TebLocalPlannerROS/teb_poses" />
    <remap from="/move_base/global_costmap/costmap"                                to="/robot2/move_base/global_costmap/costmap" />
    <remap from="/move_base/global_costmap/costmap_updates"                        to="/robot2/move_base/global_costmap/costmap_updates" />
    <remap from="/move_base/local_costmap/costmap"                                 to="/robot2/move_base/local_costmap/costmap" />
    <remap from="/move_base/local_costmap/costmap_updates"                         to="/robot2/move_base/local_costmap/costmap_updates" />
    <remap from="/move_base/local_costmap/footprint"                               to="/robot2/move_base/local_costmap/footprint" />

    <remap from="/move_base/GlobalPlanner/parameter_descriptions"                  to="/robot2/move_base/GlobalPlanner/parameter_descriptions" />
    <remap from="/move_base/GlobalPlanner/parameter_updates"                       to="/robot2/move_base/GlobalPlanner/parameter_updates" />
    <remap from="/move_base/GlobalPlanner/plan"                                    to="/robot2/move_base/GlobalPlanner/plan" />
    <remap from="/move_base/GlobalPlanner/potential"                               to="/robot2/move_base/GlobalPlanner/potential" />
    <remap from="/move_base/TebLocalPlannerROS/obstacles"                          to="/robot2/move_base/TebLocalPlannerROS/obstacles" />
    <remap from="/move_base/TebLocalPlannerROS/parameter_descriptions"             to="/robot2/move_base/TebLocalPlannerROS/parameter_descriptions" />
    <remap from="/move_base/TebLocalPlannerROS/parameter_updates"                  to="/robot2/move_base/TebLocalPlannerROS/parameter_updates" />
    <remap from="/move_base/cancel"                                                to="/robot2/move_base/cancel" />
    <remap from="/move_base/current_goal"                                          to="/robot2/move_base/current_goal" />
    <remap from="/move_base/feedback"                                              to="/robot2/move_base/feedback" />
    <remap from="/move_base/global_costmap/footprint"                              to="/robot2/move_base/global_costmap/footprint" />
    <remap from="/move_base/global_costmap/inflation_layer/parameter_descriptions" to="/robot2/move_base/global_costmap/inflation_layer/parameter_descriptions" />
    <remap from="/move_base/global_costmap/inflation_layer/parameter_updates"      to="/robot2/move_base/global_costmap/inflation_layer/parameter_updates" />
    <remap from="/move_base/global_costmap/obstacle_layer/clearing_endpoints"      to="/robot2/move_base/global_costmap/obstacle_layer/clearing_endpoints" />
    <remap from="/move_base/global_costmap/obstacle_layer/parameter_descriptions"  to="/robot2/move_base/global_costmap/obstacle_layer/parameter_descriptions" />
    <remap from="/move_base/global_costmap/obstacle_layer/parameter_updates"       to="/robot2/move_base/global_costmap/obstacle_layer/parameter_updates" />
    <remap from="/move_base/global_costmap/parameter_descriptions"                 to="/robot2/move_base/global_costmap/parameter_descriptions" />
    <remap from="/move_base/global_costmap/parameter_updates"                      to="/robot2/move_base/global_costmap/parameter_updates" />
    <remap from="/move_base/global_costmap/static_layer/parameter_descriptions"    to="/robot2/move_base/global_costmap/static_layer/parameter_descriptions" />
    <remap from="/move_base/global_costmap/static_layer/parameter_updates"         to="/robot2/move_base/global_costmap/static_layer/parameter_updates" />
    <remap from="/move_base/goal"                                                  to="/robot2/move_base/goal" />
    <remap from="/move_base/local_costmap/obstacle_layer/parameter_descriptions"   to="/robot2/move_base/local_costmap/obstacle_layer/parameter_descriptions" />
    <remap from="/move_base/local_costmap/obstacle_layer/parameter_updates"        to="/robot2/move_base/local_costmap/obstacle_layer/parameter_updates" />
    <remap from="/move_base/local_costmap/parameter_descriptions"                  to="/robot2/move_base/local_costmap/parameter_descriptions" />
    <remap from="/move_base/local_costmap/parameter_updates"                       to="/robot2/move_base/local_costmap/parameter_updates" />
    <remap from="/move_base/local_costmap/static_layer/parameter_descriptions"     to="/robot2/move_base/local_costmap/static_layer/parameter_descriptions" />
    <remap from="/move_base/local_costmap/static_layer/parameter_updates"          to="/robot2/move_base/local_costmap/static_layer/parameter_updates" />
    <remap from="/move_base/parameter_descriptions"                                to="/robot2/move_base/parameter_descriptions" />
    <remap from="/move_base/parameter_updates"                                     to="/robot2/move_base/parameter_updates" />
    <remap from="/move_base/result"                                                to="/robot2/move_base/result" />
    <remap from="/move_base/status"                                                to="/robot2/move_base/status" />
    <remap from="/move_base_simple/goal"                                           to="/robot2/move_base_simple/goal" />
  </node>


</launch>
```

## (10) Modify navigation file by substituting this code:
```
<launch>
  <param name="/use_sim_time" value="true"/>

  <!-- Run the map server -->
  <node name="map_server" pkg="map_server" type="map_server" args="/home/codebind/map.yaml" >
    <param name="frame_id" value="/map" />
  </node>



 <!-- Properties of each robot      -->
    
   <!-- AMCL      -->

    <include file="$(find multiple_turtlebots_sim)/launch/amcl_robot1.launch" />
    <include file="$(find multiple_turtlebots_sim)/launch/amcl_robot2.launch" />

   <!-- MOVE_BASE-->

    <include file="$(find multiple_turtlebots_sim)/launch/move_base1.launch" />
    <include file="$(find multiple_turtlebots_sim)/launch/move_base2.launch" />
    

<!-- Launching Rviz      -->

  <node name="rviz" pkg="rviz" type="rviz" args="-d $/opt/ros/melodic/share/rviz/default.rviz" />

</launch>

```
