Research Track I - First Assignment 
================================
Python Simulator Robot - Delucchi Manuel (S4803977)
===============================

Project Description
----------------------

The first assignment requires to write a Python script for achieving the robot's behaviour:

* Approach and grab the silver token;
* Move the silver token next to the golden one; 
* Release the silver token next to the golden one;
* Iterate the process for every token in order to pair all of the silver tokens with the golden ones;
* When all the tokens are paired the robot will complete his mission.

Holonomic Robot:

![](sr/robot.png)

Silver Token:

![](sr/token_silver.png)

Golden Token:

![](sr/token.png)

Map:

![](images/Arena.PNG)


Installing and running
----------------------

The simulator requires a Python 2.7 installation, the [pygame](http://pygame.org/) library, [PyPyBox2D](https://pypi.python.org/pypi/pypybox2d/2.1-r331), and [PyYAML](https://pypi.python.org/pypi/PyYAML/).

Pygame, unfortunately, can be tricky (though [not impossible](http://askubuntu.com/q/312767)) to install in virtual environments. If you are using `pip`, you might try `pip install hg+https://bitbucket.org/pygame/pygame`, or you could use your operating system's package manager. Windows users could use [Portable Python](http://portablepython.com/). PyPyBox2D and PyYAML are more forgiving, and should install just fine using `pip` or `easy_install`.

In order to run the project (main code written inside the file assignment1.py) the command below should be written in the terminal:

```bash
python2 run.py assignment1.py
```

Pseudo-Code
----------------------

```
Initialize silver to a true value

While 1:

    If silver is true:
    
        Then find distance, rotation and code of the closest silver token that has not already been paired
        
        If no silver token is detected:
        
            Print "I don't see any token!!"
            
            Then make the robot turn slowly until we find one
            
        Else if the robot is not well aligned with the token:
        
            Print "Left a bit..." or Print "Right a bit..."
            
            Then make the robot rotate to the left or to the right
            
        Else if we are close to the token:
        
             Print "Found it!!"
        
             Then we make the robot grab the silver token
             
             If we happen to grab it:
             
                 Print "Gotcha!!"
                 
                 Update the list that stores the silver tokens already paired
                 
                 Set silver to false
                 
         Else if the robot is well aligned with the token:
         
             Print "Forward!!" 
             
             Then make the robot drive forward    
    
    Else:
    
         Then find distance, rotation and code of the golden token
         
                 If no golden token is detected:
        
            Print "I don't see any token!!"
            
            Then make the robot turn slowly until we find one
            
        Else if the robot is not well aligned with the token:
        
            Print "Left a bit..." or Print "Right a bit..."
            
            Then make the robot rotate to the left or to the right
            
        Else if we are close to the token:
        
             Print "Found it!!"
        
             Then we make the robot release the silver token
             
             If we release it:
             
                 Print "Paired!!"
                 
                 Update the list that stores the golden tokens already paired
                 
                 Set silver to true
                 
                 Then make the robot drive slightly backward
                 
         Else if the robot is well aligned with the token:
         
             Print "Forward!!" 
             
             Then make the robot drive forward  
             
     If the list of the paired golden token is full:
     
         Print "Mission Complete!!"
     
         Then end the program
        
```
Video
----------------------

https://user-images.githubusercontent.com/97695681/201551968-4a472d7b-c090-42b3-adb4-9206dc35b263.mp4

Robot API
----------------------

The API for controlling a simulated robot is designed to be as similar as possible to the [SR API][sr-api].

Features
----------------------

### Motors ###

The simulated robot has two motors configured for skid steering, connected to a two-output [Motor Board](https://studentrobotics.org/docs/kit/motor_board). The left motor is connected to output `0` and the right motor to output `1`.

The Motor Board API is identical to [that of the SR API](https://studentrobotics.org/docs/programming/sr/motors/), except that motor boards cannot be addressed by serial number. So, to turn on the spot at one quarter of full power, one might write the following:

```python
R.motors[0].m0.power = 25
R.motors[0].m1.power = -25
```

Two main functions have been designed to drive straight and to rotate the robot on its axis:

* ```drive(speed, seconds)```: Function for setting a linear velocity. It allows the robot to move into a straight line for a certain time and with a defined speed.
* ```turn(speed, seconds)```: Function for setting an angular velocity. It allows the robot to turn on its axis.

Each function has no ```Returns```.

Also each function hs two ```Arguments```:

* ```speed (int)```: Speed of the motors.
* ```seconds (int)```: Time interval.

### The Grabber ###

The robot is equipped with a grabber, capable of picking up a token which is in front of the robot and within 0.4 metres of the robot's centre. To pick up a token, call the `R.grab` method:

```python
success = R.grab()
```

The `R.grab` function returns `True` if a token was successfully picked up, or `False` otherwise. If the robot is already holding a token, it will throw an `AlreadyHoldingSomethingException`.

To drop the token, call the `R.release` method.

Cable-tie flails are not implemented.

Two functions have been designed to clean the main function: ```MovetoSilver()``` & ```MovetoGolden```. The role of these functions has been described in the pseudo-code section right above.

### Vision ###

To help the robot find tokens and navigate, each token has markers stuck to it, as does each wall. The `R.see` method returns a list of all the markers the robot can see, as `Marker` objects. The robot can only see markers which it is facing towards.

Each `Marker` object has the following attributes:

* `info`: a `MarkerInfo` object describing the marker itself. Has the following attributes:
  * `code`: the numeric code of the marker.
  * `marker_type`: the type of object the marker is attached to (either `MARKER_TOKEN_GOLD`, `MARKER_TOKEN_SILVER` or `MARKER_ARENA`).
  * `offset`: offset of the numeric code of the marker from the lowest numbered marker of its type. For example, token number 3 has the code 43, but offset 3.
  * `size`: the size that the marker would be in the real game, for compatibility with the SR API.
* `centre`: the location of the marker in polar coordinates, as a `PolarCoord` object. Has the following attributes:
  * `length`: the distance from the centre of the robot to the object (in metres).
  * `rot_y`: rotation about the Y axis in degrees.
* `dist`: an alias for `centre.length`
* `res`: the value of the `res` parameter of `R.see`, for compatibility with the SR API.
* `rot_y`: an alias for `centre.rot_y`
* `timestamp`: the time at which the marker was seen (when `R.see` was called).

For example, the following code lists all of the markers the robot can see:

```python
markers = R.see()
print "I can see", len(markers), "markers:"

for m in markers:
    if m.info.marker_type in (MARKER_TOKEN_GOLD, MARKER_TOKEN_SILVER):
        print " - Token {0} is {1} metres away".format( m.info.offset, m.dist )
    elif m.info.marker_type == MARKER_ARENA:
        print " - Arena marker {0} is {1} metres away".format( m.info.offset, m.dist )
```

Two main functions are designed to recognize the Marker object closest to the robot and whether is gold or silver:

* ```find_silver_token(list_silver_token)```: This function detects the closest silver box to the robot that has not already been paired.

  ```Argument```: 

  * ```list_silver_token``` : List of codes (silver tokens) that have already been paired.
  
  ```Returns```: 

  * ```dist (float)```: Distance of the closest silver token, ```-1``` if no silver token is detected.
  * ```rot_y (float)```: Angle between the robot and the silver token, ```-1``` if no silver token is detected.
  * ```code (int)```: Numeric code of the token.

* ```find_golden_token(list_golden_token)```: This function detects the closest golden box to the robot that has not already been paired.

  ```Argument```: 

  * ```list_golden_token``` : List of codes (golden tokens) that have already been paired.

  ```Returns```: 

  * ```dist (float)```: Distance of the closest silver token, ```-1``` if no silver token is detected.
  * ```rot_y (float)```: Angle between the robot and the silver token, ```-1``` if no silver token is detected.
  * ```code (int)```: Numeric code of the token.

[sr-api]: https://studentrobotics.org/docs/programming/sr/

Possible Improvements
----------------------

* Obstacle avoidance can be implemented.
* It could be useful to make the robot choose the closest golden token to pair with the silver one instead of choosing the first golden token in the robot's field of view. This could be probably done by making the robot turn completely on its axis in order to detect all the golden tokens and finally decide what's the closest one.
* Minimize the robot's research time, increasing the robot's efficiency and making it faster.
