Vehicle Model:
	The model used for this project is a bicycle model which only takes into account the position and heading direction, velocity into account and does not take into account dynamic forces like inertia, friction etc. 

Polynomial Fitting:
	The co-ordinates that are obtained from the simulator are in map's co-ordinates. They need to be converted to car's co-ordinates for further processing using affine transform. Once the co-ordinates are obtained in car's reference, a 3rd degree polynomial is fitted with those co-ordinates since many of the roads can be modeled using a 3rd degree polynomial. Polyfit function is used to fit the polynomial along the co-ordinates. 

Cross Track Error, Orientation Error:
	Since the car is always at 0,0 in its frame of reference, the cte of the car can be calcualted by plugging 0,0 into the 3rd degree polynomial equation at any given instant. Polyeval is used to calculate this value. The angle at which the car is heading can also be calculated by calculating the slope of its tangent at the car's relative position. 

A state vector is created which contains the position (0,0), orientation (0), v, cte, and epsi. This state vector is passed to the MPC solver along wit the coefficients of the 3rd degree polynomial. MPC Solver uses these to calculate the next steering angle and the throttle. These values are passed to the simulator for moving the car.


MPC Solver:
	Most of the code for the solver has been used from the lectures and Q&A session that was held for helping out students solve this project. My contributions were in tuning the parameters to make the car go on the track without losing the lane. 

	Solver takes into account various parameters for calculating the speed and steering angle that needs to be passed to the simulator. The parameters that I tuned were reference velocity (ref_v), simulation time (N, dt), and the cross track error dependencies (multipliers for fg[0]). 

	The state that is input to the MPC solver is used to predict the path on which the car would travel for the N*dt seconds. Even though the path trajectory for all time between now and N*dt is solved, only the first actuations are sent to the simulator for processing. The model is again recalculated by the state vector that the simulator returns. This is because the output of out model is only an approximation and errors in approximations can get accumulated very quickly and lead to a wrong path. 

	The time between simulator's inputs and MPC's outputs need to be a small as possible to avoid stale situations. This is why the number of computations that need to be done should be carefully chosen. (N, dt)

Parameters that were tweaked in my model: ref_v, fg[0]'s multipliers, N, dt
	fg[0]. ref_v:
		fg[0] is the cost function that is used to calculate the cte. The aim is to minimize the cte to keep the car in the lane. fg[0] is made to depend on the car speed, acceleration, change of acceleration and the position of the vehicle. The multiplying co-efficients dictate how much affect those parameters should have on fg[0]. Having a higher multiplier means that the model would try its best to minimize that parameter. 

		ref_v is the reference velocity which is included in the calculation of fg[0]. This value is there to prevent the car from going at a speed higher than this specified velocity. I started at ref_v of 30 (the minimum for the project).

		I started with modifying the parameters that would make the feel smooth like penalizing the model for sudden change in acceleration and sudden change in speeds or direction of travel. Started with a small penalty value for both and started increasing them to keep the car in lane. At the same time, I started increasing the speed which would make the car go out of lane sometimes. After modifying the parameters few times, I started noticing a pattern that penalizing the car for sudden changes to acceleration and speed are reasonable enough parameters to keep the car in the lane for smaller speeds. However, if the speed of the car is further increased, other parameters like multipliers for position and angle also need to be tweaked. 

		During my tweaking, I reached the values for 10 for a multiplier to acceleration and 5e5 as a multiplier as a penalizing factor for sudden change to velocities between sequential acutations. These values at a speed of 40 kept the car in the lane.

	N, dt:
		These indicate the number of time stamps that need to be taken into consideration for calculating the path and motion. The higher N indicates that the car would predict the path for a longer duration and hence more computation power. Smaller N means that car only predicts its path for a short time. It does not take into account the future and might lead to stronger corrections if path changes suddently. Higher dt means that car would calculate the path at spaced out intervals and might not be good if the path changes quite frequently (think curvy and windy road). Smaller dt would lead to more computation.

		I started with N = 10 and dt = 0.05 (from the lectures) and it felt like the car was little wobbly at the corners. I reduced the dt to 0.01 to take more frequent measurements so that the car can predict better. From then, I keep increasing N to predict further into the path. 

		I stopped varying these N, dt after the car decently kept in lane even at the corners. The values came out to be N = 50 and dt = 0.01.


Latency:
	There would be some delay between the time we get the inputs into the simulator and the our MPC Solver processes those and sends those back. This latency needs to be taken into consideration and might play a higher role especially at higher speeds. I assumed that the car continues at current speed and heading direction as it was, for the entire duration of this latency. This assumption works reasonably okay for slower speeds but for rapdily changing path at higher speeds, this model does not work and need to be modified. 


		