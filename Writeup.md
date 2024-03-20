# Building an Estimator
### Michael Walden

This project consisted of building a 3D state estimator for the quadrotor simulation that was used in the last project for the control system.  

First, we calculated the standard deviation from the simulated measurements.  After running the 06_NoisySensors scenario, the CSV files that were output were processed in Excel in order to calculate the standard deviations and these values were entered into the appropriate fields of the 6_Sensornoise.txt configuration file.  After plugging in these values, the test showed that the standard deviation could correctly capture ~68% of the sensor measurements indicating an accurate estimate for the standard deviation. 

Second, an improved integration scheme was implemented for the Update from the IMU function which takes the delta time and gyroscope measurements and integrates them in the body frame, converting the current roll, pitch, and yaw estimates to a quaternion before multiplying the two together to obtain the predicted attitude.  The quaternion is converted to the predicted pitch, roll and yaw values before being normalized.  The roll and pitch acceleration values are then calculated from the accelerometer measurements.  Everything is finally input into the new roll and pitch estimates using a complementary filter.  After implementing the above changes, the quadrotor was able to maintain < 0.1 for each of the Euler angles for a duration of 3 seconds to pass the scenario.

Third, the PredictState function was implemented which carries out the non-linear state transition function g.  This takes the current state, accelerometer, and gyro measurements as input and calculates the predicted state.  First the current roll, pitch and yaw estimates are converted from Euler angles into a quaternion which represents the attitude of the vehicle.   The accelerometer measurement is used to convert this attitude into the inertial frame and then subtract the gravity vector.  These values are then used as the control input for the integration calculation of the new predicted state.  R<sup>'</sup><sub>bg</sub> was then implemented in the GetRbgPrime function which is the partial derivative of the R<sub>bg</sub> rotation matrix with respect to yaw.  Next, the Predict function was implemented that calculates the new covariance matrix estimate from the Jacobian of the non-linear state transition function and the current covariance matrix estimate with the process noise.  After making the above changes, the Predict state scenario showed that the estimator was able to track the true state of the vehicle reasonably well.  The Predict Covariance scenario also demonstrated that the predicted covariance tracks reasonably well with how the covariance in the real data grows over time.

Fourth, the Update from the magnetometer function was implemented which takes the current estimated yaw and calculates the difference between the current measurement and then normalizes this value.  The $h'$ matrix is initialized and everything is passed to the update function.

Finally, the Update from the GPS function was implemented and this just grabs the current position and velocity of the current estimated state and initializes the $h'$ matrix before passing these values to the update function. 

After making the above changes for the update functions for the GPS and magnetometer, both scenarios for testing the GPS and magnetometer passed with the estimated yaw value tracking very well over time with the true yaw state and doing an acceptable job of filtering the high frequency noise from the true magnetometer measurements.  The yaw error stayed within the acceptable range to pass the scenario's test.  The GPS scenario also demonstrated that the estimator was able to track the true position and velocity over time, maintaining an error sufficiently low enough to pass the test.

The controller that was implemented in the Controller project was then merged into the project in order to have a full quadrotor controller and state estimator implementation.  The proportional gains for the yaw controller had to be lowered slightly along with the R value for the PQR body rate controller in order to pass the Yaw and GPS scenarios again.