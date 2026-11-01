

# Rover Propulsion System Pseudocode


## 1. System Overview

The propulsion system controls six independently driven
brushless DC motors using Electronic Speed Controllers
(ESCs). The propulsion software will communicate with the Radio Frequency (RF) communication subsystem and the LiDAR navigation subsystem through the Raspberry Pi.

Four steering servo motors, two on the front wheels and two on the rear wheels. The middle wheels remain fixed in their steering orientation. These servos will control the wheel steering angles to allow the rover to perform different turning maneuvers.

### 1.1 Initial Movement Commands
The first version of the propulsion program will implement three basic movement commands:

    FORWARD(speed): Command all six propulsion motors to move the rover forward at the requested throttle.

    REVERSE(speed): Command all six propulsion motors to move the rover backward at the requested throttle, assuming the selected ESCs support reverse operation.

    STOP(): Command all six propulsion motors to stop.

During initial testing  the four steering servos will remain at their calibrated straight ahead positions.

### 1.2 Future Steering Functions
Once the steering configuration is finalized by the mechanical team, the software will be expanded to support

    TURN_LEFT(speed, angle): Calculate individual motor commands and steering angles for a left turn.

    TURN_RIGHT(speed, angle): Calculate individual motor commands and steering angles for a right turn.

    SHARP_LEFT(speed, angle): Coordinate the front and rear steering servos and individual wheel speeds to achieve a tighter left turning radius.

    SHARP_RIGHT(speed, angle): Coordinate the front and rear steering servos and individual wheel speeds to achieve a tighter right turning radius.

    SPIN_LEFT(speed): Rotate the rover counterclockwise around its center with minimal translational movement.    

    SPIN_RIGHT(speed): Rotate the rover clockwise around its center with minimal translational movement.

    CRAB_LEFT(speed, angle): Move the rover diagonally toward the left while maintaining approximately the same chassis orientation.

    CRAB_RIGHT(speed, angle): Move the rover diagonally toward the right while maintaining approximately the same chassis orientation.
    
### 1.3 Control Sequence
    1. The Raspberry Pi receives operator commands through the RF communication subsystem.
    2. Pi receives obstacle and distance information through the LiDAR subsystem.
    3. Software evaluates the requested movement and applicable safety conditions.
    4. Calculates the required commands for the six propulsion motors and, when necessary the four steering servos.(To do this mechanical prototype must be assembled)
    5. Raspberry Pi sends the calculated propulsion commands to the ESCs through the selected control interface.

## 2. Motor Initialization
The propulsion system will initialize six BLDC motor controllers and four steering servos before beginning normal operation. Each ESC will be configured independently, allowing individual control of the corresponding propulsion motor.

During initialization all propulsion motors will remain in a  neutral state.

### 2.1 Propulsion Motor Initialization

FUNCTION Initialize_All_Motors()

    // Define the six propulsion motors by wheel position
    motors = [FL, ML, RL, FR, MR, RR]

    // Initialize each motor controller independently
    FOR each motor IN motors

        // Configure the communication interface for the ESC
        status = Configure_ESC(motor)

        // Check if the ESC configuration was successful

    
        // Check if the ESC configuration was successful
        IF status == FAILED
            Stop_All_Motors()
            RETURN FAILED
        END IF

        // Set motor output to neutral to prevent movement
        Set_Motor_Neutral(motor)

    END FOR

    // Motors remain neutral until movement is authorized
    RETURN SUCCESS

END FUNCTION   

### 2.2 Steering Servo Initialization

FUNCTION Initialize_Steering_Servos()

    // Define the four steering servos
    // Middle wheels do not have steering actuators
    servos = [FL, RL, FR, RR]

    // Initialize each steering servo independently
    FOR each servo IN servos

    // Configure the servo control interface
    status = Configure_Servo(servo)

    // Check if the servo was configured successfully
    IF status == FAILED
        RETURN FAILED
    END IF    

    // Each servo may have a different mounting offset
    angle = Center_Angle(servo)

    // Command the servo to its neutral steering position
    Set_Steering_Angle(servo, angle)

    END FOR

    // All steering servos have been initialized
    RETURN SUCCESS

END FUNCTION


## 3. Movement Functions
The movement control functions will calculate the required throttle and rotational direction for each of the six BLDC propulsion motors based on commands received by the Raspberry Pi.

Motor throttle will be represented using range from -1.0 to +1.0, where positive values represent forward motion, negative values represent reverse motion, and zero represents neutral.

### 3.1 Forward Movement & Reverse Movement

FUNCTION Move_Rover(speed)

    // Limit throttle between -1.0 and +1.0
    // Positive = Forward
    // Negative = Reverse
    // Zero = Stop
    speed = Limit_Throttle(speed, -1.0, 1.0)

    // Create an empty mapping for motor commands
    motor_targets = {}

    // Assign the requested throttle to all six motors
    FOR each motor IN [FL, ML, RL, FR, MR, RR]
        motor_targets[motor] = speed
    END FOR

    // Return motor commands to the main program
    RETURN motor_targets

END FUNCTION

### 3.3 Stop All Motors
FUNCTION Stop()

    // Set the requested throttle of every motor to zero
    FOR each motor IN motors

        motor_targets[motor] = 0

    END FOR

    // Return neutral motor targets
    RETURN motor_targets

END FUNCTION

### 3.4 Future Steering Functions

The following functions will be developed once the mechanical team finalizes the steering geometry and actuator specifications:

    SHARP_LEFT(speed, angle)
    SHARP_RIGHT(speed, angle)

    SPIN_LEFT(speed)
    SPIN_RIGHT(speed)

    CRAB_LEFT(speed, angle)
    CRAB_RIGHT(speed, angle)

## 4. Main Control Loop

The main control loop coordinates the propulsion system by processing movement commands received through the RF subsystem and safety information provided by the LiDAR/navigation subsystem (This is late game shit and the hardest part to implement). Continuously evaluates the latest valid movement command and determines the appropriate motor outputs.

BEGIN Main_Control

    // Initialize motors and steering servos
    status = Initialize_Propulsion_System()

    IF status == FAILED
        EXIT
    END IF

    // Start with all motors stopped
    requested_speed = 0

    WHILE system_running

        // Read latest commands and subsystem status
        Update_RF_Status()
        Update_LiDAR_Status()

        // Check whether propulsion is permitted
        IF Emergency_Stop_Active() OR Communication_Lost() OR Motor_Fault_Active()

            requested_speed = 0

        ELSE IF Obstacle_Blocks_Movement()

            requested_speed = 0

        ELSE IF New_Valid_RF_Command()

            requested_speed = Get_Requested_Speed()

        END IF

        // Calculate requested throttle for all six motors
        // Move_Rover() assigns the same requested throttle to each motor.
        // For example, requested_speed = 0.8 produces six targets of 0.8.
        // This function only calculates values it does not control hardware.
        requested_targets = Move_Rover(requested_speed)

        // TODO: Implement acceleration and direction-change limiting.
        // Apply_Motor_Limits() will prevent sudden changes in motor throttle.
        // It will compare each requested target against the previously
        // commanded throttle and the time elapsed since the last update.
        // For example, if the motors are stopped and the operator requests
        // 80% throttle, the function may initially allow only 10%.
        // Subsequent control cycles will gradually increase the throttle
        // until the requested value is reached.
        // The actual limits will be determined after hardware testing.
        // Emergency stops must bypass normal acceleration limiting.
        safe_targets = Apply_Motor_Limits(requested_targets)

        // Update_All_ESCs() will receive the six limited motor commands,
        // convert each throttle value into the signal required by its ESC,
        // and transmit the commands through the selected hardware interface.
        // The ESC communication protocol has not been selected yet.
        // This function will be developed once the ESC hardware is finalized.
        Update_All_ESCs(safe_targets)

    END WHILE

## 5. Safety and Error Handling

## 6. Pending Hardware Decisions