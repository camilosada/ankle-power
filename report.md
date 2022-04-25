# Ankle Power

- [Ankle Power](#ankle-power)
  - [Contexte and objective](#contexte-and-objective)
  - [Power](#power)
  - [Coordinate system](#coordinate-system)
  - [The data](#the-data)
  - [Procedure](#procedure)
  - [Results](#results)
  - [Discussion and conclusion](#discussion-and-conclusion)
  - [Bibliography](#bibliography)
## Contexte and objective
The physical variables associated with sensor measurements can be used to find a clinical variable.
Using the recordings of the movements of a patient’s ankle, the objective of this work was to find a clinical variable that allows to represent the clinical state of the patient, and thus be exploitable by a physician.

## Power
In activities of daily living the rate at which muscles can produce work, referred as power production, is the critical performance variable. Power production is measured as the amount of work done per unit time (Eq. 1).

$\bar{𝑃}$ = 	$\frac{𝑈}{∆𝑡}$ = $\bar{𝐹}$·$\frac{∆𝑟}{∆𝑡}$  = $\bar{𝐹}$ · $\bar{v}$ = 𝑚 $\bar{a}$ · $\bar{v}$  (Eq. 1)

Power is computed as the work done (U) divided by the amount of time (t) it took to perform
the work, or as the product of force (F) and velocity (v). Because acceleration and force are
proportional to each other, we can obtain the power associated with the movement by
multiplying the velocity and the acceleration (a).


If the power is positive indicates that the energy flows from the muscle, and if the power is
negative indicates that the energy flows to the muscle (figure 1). Since what interests us is to
know the power that the patient is capable of delivering, we will only analyze the positive
power values.

![power](img\ankle_power.jpg)
Figure 1. Power of the ankle joint during a normal gait cycle.

## Coordinate system
The readings of sensors such as the accelerometer, gyroscope, and magnetometer are in
the sensor coordinate system. The world coordinate system (WCS) is a fixed universal
system that works as an absolute reference for all sensors. A rotation matrix (Eq. 2) is used
to translate the sensor's coordinates into a single coordinate system, the WCS. Spatial
rotations in three dimensions can be parametrized using both Euler angles and quaternions.

![euler](img\euler.jpg) (Eq. 2)<br/>
XYZ are the rotated body coordinates, xyz the original lab coordinates and ψ, θ, φ Euler angles.

Quaternions provide a convenient mathematical notation for representing spatial orientations
and rotations. Compared to rotation matrices they are more compact, more numerically
stable, and more efficient. A unit quaternion can be described as:

![u_q](img\unit_q.jpg) (Eq. 3)

The Euler angles can be obtained from the quaternions via the relations:

![quaternions](img\quaternions.jpg) (Eq. 4)

## The data
3D trajectory of the ankle of a patient reconstructed from measurements taken by a magneto inertial sensor. 
The patient wore the device every day for 15 days. For each day we have a file containing the 
time (in seconds), the speed (in metres per second) in the inertial frame of reference and the 
quaternion representing the orientation of the ankle in the inertial frame.
Each recording is assumed to start at 8:00 AM but each has a different duration.

## Procedure
The first step was to use the quaternions to obtain the speed in the world coordinate system.
As seen before (Eq. 4), It is possible to compute the euler angles from the quaternions.

`[roll_phi, pitch_theta, yaw_psi] = quartion_to_euler_angles(quaternion)`

Once the Euler angles were obtained, the rotation matrix was constructed.

`rotation_matrix = rotation_matrix_from_euler_angles(roll_phi, pitch_theta, yaw_psi)`

Finally using the rotation matrix, it was possible to obtain the speed in the world coordinate system (eq. 2).

`speed_rot = speed_rotation(rotation_matrix,speed)`

Before calculating the acceleration as $\frac{v_{i+1}-v_i}{t_{i+1}-t_i}$
it was necessary to filter the speed signal.

Using the FFT (Fast Fourier transform) it was observed the frequencies went from 0 Hz to more than 60 Hz, so a low-pass butterworth filter, with a cutoff frequency of 10 Hz, was used for that purpose.

`speed_rot_filt = butterworth_filter(day_time,fc,degree,speed_rot,type='lowpass')`

`acceleration_filt = acceleration_from_speed(speed_rot_filt,day_time)`

Once the acceleration was obtained, the power was calculated.

`power_filt = power_from_speed_acceleration(speed_rot_filt,acceleration_filt)`

In order to keep only relevant values, negative values were set to zero, and values that were not representative of the whole were also set to zero.

`power_quantile = np.where(power_filt < quantile_max, power_filt,0)`

`power_quantile = np.where(power_quantile > 0, power_quantile, 0)`

## Results
In figure 2 it is possible to observe the power obtained from the first 30 seconds of the first day. This short time frame was traced to make it readable.

![power1](img\power1.jpg)<br/>
Figure 2. Power obtained from the first 30 seconds of the first day

In figure 3 it is possible to see the mean power of each hour of the first, fourth and fourteenth day. The fourth day is the day with the most amount of time with information, and the fourteenth, the day with less.

![power3](img\power3.jpg)<br/>
Figure 3. Mean power of each hour of the first, fourth and fourteenth day.

To compare the power between the days, the mean power generated on each day was calculated (Figure 4).
Because each day has a different number of hours with information, to make it comparable, the days that had less than 7 hours of information were removed (days 5,6,10 and 14). From the rest of the days, the time interval between 8 am and 3 pm was selected, and the mean value of power was calculated (Figure 5).

![meanday](img\meanday.jpg)<br/>
Figure 4. Mean power generated on each day.

![meanday2](img\meanday2.jpg)<br/>
Figure 5. Mean power from 8 am to 3pm.

In the last figure (figure 6), it is possible to see the mean value for each week at each hour between 8 am and 3 pm. Only the days with information in that period were used. To have the same number of days in each week, the first 5 were selected for week 1 and the last 5 for week 2.The middle day was not used (day 8). This decision was based on separating the two weeks as much as possible to see if there was a difference between the first and last days.

![meanweek](img\meanweek.jpg)<br/>
Figure 6. Mean value for each week at each hour between 8 am and 3 pm.

## Discussion and conclusion
Seeing the evolution of the power over the time in figure 2 it is possible to observe that the signal varies greatly from one second to another. These variations are normal, they correspond to the natural fluctuations in the patient's data.

If we compare figure 2 with figure 3, where it was taken the mean value during each hour, it is possible to see the values are more stable and in a range of values comparable to those found in different sources, as in figure 1. If we observe the variations between the hours on the different days, it is not possible to determine one specific period of time where the patient is more or less active.

Comparing figure 4 with figure 5, we can see that the patient increased the amount of activity through the days. Although the mean value of each day is taken, the days with only four hours of information are not comparable with days with more hours. Therefore, figure 5 is more appropriate to compare between days, even though only the first half of the day is taken.

Figure 6 compares the power generated by the patient in each week. It can be observed that in week 2 the patient has generated more power than in week 1. While during week 2 there are not too high variations every hour, in week 1 it can be seen that there is a peak at 1 pm.

Because the days in each week are not consecutive (for example, days 1,2,3,4 and 7 in week 1), and because an increase in the power generated by the patient is seen over the course of the days, the values can be influenced by the days furthest away (for example by day 7 in week 1).

After evaluating the three periods of time on which to compute the power, it could be seen that the comparison between hours does not give so much information. The values between each week can vary greatly depending on the number of days available. And finally, if only the days that provide a sufficient amount of information are taken into account, the representation per day, allows monitoring and evaluation of the patient's progress over time (in days).

## Bibliography
[1] Roger M. Enoka. Neuromechanics of human movement<br/>
[2] https://en.wikipedia.org/wiki/Conversion_between_quaternions_and_Euler_angles<br/>
[3] https://en.wikipedia.org/wiki/Quaternion<br/>
[4] https://ieeexplore.ieee.org/document/7487531