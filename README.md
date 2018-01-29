# CarND-Path-Planning-Project
Self-Driving Car Engineer Nanodegree Program


[overview]: ./images/1.png "overview"
[changelane]: ./images/2.png "changelane"

### Goals
In this project your goal is to safely navigate around a virtual highway with other traffic that is driving +-10 MPH of the 50 MPH speed limit. You will be provided the car's localization and sensor fusion data, there is also a sparse map list of waypoints around the highway. The car should try to go as close as possible to the 50 MPH speed limit, which means passing slower traffic when possible, note that other cars will try to change lanes too. The car should avoid hitting other cars at all cost as well as driving inside of the marked road lanes at all times, unless going from one lane to another. The car should be able to make one complete loop around the 6946m highway. Since the car is trying to go 50 MPH, it should take a little over 5 minutes to complete 1 loop. Also the car should not experience total acceleration over 10 m/s^2 and jerk that is greater than 10 m/s^3.

![overview][overview]

#### The map of the highway is in data/highway_map.txt
Each waypoint in the list contains  [x,y,s,dx,dy] values. x and y are the waypoint's map coordinate position, the s value is the distance along the road to get to that waypoint in meters, the dx and dy values define the unit normal vector pointing outward of the highway loop.

The highway's waypoints loop around so the frenet s value, distance along the road, goes from 0 to 6945.554.

## Basic Build Instructions

1. Clone this repo.
2. Make a build directory: `mkdir build && cd build`
3. Compile: `cmake .. && make`
4. Run it: `./path_planning`.

I use XCode, hence run `cmake -G "Xcode" .` in the root of project foler which will generate XCode project.

Here is the data provided from the Simulator to the C++ Program

#### Main car's localization Data (No Noise)

["x"] The car's x position in map coordinates

["y"] The car's y position in map coordinates

["s"] The car's s position in frenet coordinates

["d"] The car's d position in frenet coordinates

["yaw"] The car's yaw angle in the map

["speed"] The car's speed in MPH

#### Previous path data given to the Planner

//Note: Return the previous list but with processed points removed, can be a nice tool to show how far along
the path has processed since last time.

["previous_path_x"] The previous list of x points previously given to the simulator

["previous_path_y"] The previous list of y points previously given to the simulator

#### Previous path's end s and d values

["end_path_s"] The previous list's last point's frenet s value

["end_path_d"] The previous list's last point's frenet d value

#### Sensor Fusion Data, a list of all other car's attributes on the same side of the road. (No Noise)

["sensor_fusion"] A 2d vector of cars and then that car's [car's unique ID, car's x position in map coordinates, car's y position in map coordinates, car's x velocity in m/s, car's y velocity in m/s, car's s position in frenet coordinates, car's d position in frenet coordinates.


## Valid Trajectories

1. The car is able to drive at least 4.32 miles without incident

During test, we can see the car at least can run 25+ MPH without incident.

2. The car drives according to the speed limit.

We set speed limit to 49.5 MPH, in the simulator tests, we never see LIMIT violation RED signal in the screen.

3. Max Acceleration and Jerk are not Exceeded.
Always run within expected range, no Max and Jerk violation in the simulator screen.

4. Car does not have collisions.
No collisions found. But have some rare small incident during lane change. Usually happen every 20-30 miles.

5. The car stays in its lane, except for the time between changing lanes.
Car is always stay in the lane. Only if there are some lane change needed, car will move out current lane to new lane.

6. The car is able to change lanes
Base on the simulation's sensor_fusion data, if there is no car on neighbor lane, and there is slow car before, car will do lane change.

![changelane][changelane]

## Reflection

All the main code is in main.cpp, and spline.h is download and imported for interpolating points. This application is a standalone C++ app which uses uWebSockets to commucate with simulator. The data sent form simulator include car's location, speed and sensor fusion data.

        // Main car's localization Data
        double car_x = j[1]["x"];
        double car_y = j[1]["y"];
        double car_s = j[1]["s"];
        double car_d = j[1]["d"];
        double car_yaw = j[1]["yaw"];
        double car_speed = j[1]["speed"];

        // Previous path data given to the Planner
        auto previous_path_x = j[1]["previous_path_x"];
        auto previous_path_y = j[1]["previous_path_y"];
        // Previous path's end s and d values
        double end_path_s = j[1]["end_path_s"];
        double end_path_d = j[1]["end_path_d"];

        // Sensor Fusion Data, a list of all other cars on the same side of the road.
        auto sensor_fusion = j[1]["sensor_fusion"];


When application receive above data, it will analysis and predict the lanes usage, e.g. which lanes are occupied by other cars, and cars' speed. main.cpp L-254-290.

Then car will decide what is the behavior for next step. If there is other car ahead, it will check whether left and right has empty lane, if yes, then switch lane, other wise slow down the speed.

        if ( car_ahead ) { // Car ahead
              if ( !car_left && lane > 0 ) {
                  // if there is no car left and there is a left lane.
                  lane--; // Change lane left.
              } else if ( !car_righ && lane != 2 ){
                  // if there is no car right and there is a right lane.
                  lane++; // Change lane right.
              } else {
                  speed_diff -= MAX_ACC;
              }
          }

If there is car ahead, keep the lane and accelerate to max speed (configurable).

Then prepare the trajectory waypoints. Previous last 2 points are used if there are more than two previous way points otherwise will use car's current position.
When interplotting points, Spline toolkit is used:

        // Create the spline.
        tk::spline s;
        s.set_points(ptsx, ptsy);

Here are the algorithms for next points with Spline transform. main.cpp L373 - 415

        double N = target_dist/(0.02*ref_vel/2.24);
        double x_point = x_add_on + target_x/N;
        double y_point = s(x_point);


Finally, will return a json package back to simulator, simulator will refresh the UI based on new trajectory waypoints.

        json msgJson;

        msgJson["next_x"] = next_x_vals;
        msgJson["next_y"] = next_y_vals;

        auto msg = "42[\"control\","+ msgJson.dump()+"]";

        ws.send(msg.data(), msg.length(), uWS::OpCode::TEXT);


---

## Dependencies

* cmake >= 3.5
 * All OSes: [click here for installation instructions](https://cmake.org/install/)
* make >= 4.1
  * Linux: make is installed by default on most Linux distros
  * Mac: [install Xcode command line tools to get make](https://developer.apple.com/xcode/features/)
  * Windows: [Click here for installation instructions](http://gnuwin32.sourceforge.net/packages/make.htm)
* gcc/g++ >= 5.4
  * Linux: gcc / g++ is installed by default on most Linux distros
  * Mac: same deal as make - [install Xcode command line tools]((https://developer.apple.com/xcode/features/)
  * Windows: recommend using [MinGW](http://www.mingw.org/)
* [uWebSockets](https://github.com/uWebSockets/uWebSockets)
  * Run either `install-mac.sh` or `install-ubuntu.sh`.
  * If you install from source, checkout to commit `e94b6e1`, i.e.
    ```
    git clone https://github.com/uWebSockets/uWebSockets
    cd uWebSockets
    git checkout e94b6e1
    ```

## Editor Settings

We've purposefully kept editor configuration files out of this repo in order to
keep it as simple and environment agnostic as possible. However, we recommend
using the following settings:

* indent using spaces
* set tab width to 2 spaces (keeps the matrices in source code aligned)
