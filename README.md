# AI-Based Safe and Sustainable Traffic Signal System (DDQN + PER)

**1. The Design Methodology**
  The design methodology outlines the comprehensive plan used to develop, train, and validate the adaptive traffic control system.

**A. Sequence of Steps**
```The project execution is organized into three distinct, sequential phases:
Phase 1: Environment Setup
  Defining the road network geometry and traffic demand using SUMO's XML files (.net.xml, .rou.xml).Establishing the connection and command link via the TraCI (Traffic Control Interface) to Python.
Phase 2:
   AI Model TrainingThe core iterative cycle where the DDQN+PER Agent observes the environment state, chooses an action, receives the multi-objective reward, and updates its Policy Network weights.Integrating and testing the Emergency Vehicle Priority override module.
Phase 3:
  Evaluation & DeploymentTesting the final, converged model against baselines (Fixed-Time, standard DQN).Analyzing key performance metrics (bar{W}, Q, E) and preparing the optimal signal strategy for deployment.```
**B. Architecture and Diagrams**
The system architecture is based on a standard Reinforcement Learning framework, ensuring a clear flow of data between the environment and the agent.
**System Architecture Flowchart:**
The flowchart visually represents the State - arrow - Action - Reward - Update loop central to the DDQN+PER training process, including the external intervention path for emergency handling.

Sequence Diagram: Emergency Vehicle Priority
  This sequence illustrates the immediate, safety-critical actions taken when an emergency vehicle is detected:
    Emergency Detector identifies the vehicle's proximity.
    Traffic Controller receives the alert and immediately halts the adaptive control process.        The Traffic Controller uses TraCI to send a direct command to the SUMO Environment.
    The SUMO Environment executes the immediate green phase for the EV's path.
    After the EV clears, the Traffic Controller resumes the normal DDQN+PER adaptive control cycle.

**C. Hardware and Software Selection**
The system utilizes industry-standard tools for simulation and artificial intelligence:

**Simulation Environment: SUMO** (Simulation of Urban Mobility) for realistic traffic modeling.
**AI Frameworks: **Python 3.x environment leveraging TensorFlow/PyTorch for neural network implementation.
**Interface:** TraCI for communication between the Python agent and SUMO.

```**2. Project Implementation**
Project implementation details the mathematical foundation, the custom reward function, and the necessary code structure.

**A. Algorithm and Formulae**
DDQN Target Q-Value Formula
The Double Deep Q-Network (DDQN) utilizes two separate networks to stabilize the learning process by decoupling action selection and value evaluation.

<img width="663" height="72" alt="image" src="https://github.com/user-attachments/assets/573e2c40-b955-4183-9e25-3e1d6930442e" />


CO2/Fuel-Aware Reward FormulaThis custom reward function promotes environmental sustainability by penalizing the combined total of traffic congestion (Q_t) and CO2 emissions (E_t). alpha and 
beta are weight coefficients.

<img width="313" height="84" alt="image" src="https://github.com/user-attachments/assets/5df41d23-1103-4853-9895-189ec45e8a76" />

**B. Code Segments**
The following segments illustrate the implementation of the reward logic and the agent's interaction with the simulation environment.
**Python Function for CO2-Fuel-Aware Reward**

```import traci

def calculate_eco_reward(intersection_lanes, alpha=0.6, beta=0.4):
    total_queue = sum(traci.lane.getQueueLength(lane_id) for lane_id in intersection_lanes) 
    total_co2 = traci.simulation.getCO2Emission()
    reward = -(alpha * total_queue + beta * total_co2)
    return reward
```
**Python Function for Agent Step and TraCI Interaction**
```
def take_step(agent, current_state, TLS_ID, next_phase, env):
    action = agent.choose_action(current_state)
    
    if action == 1:
        traci.trafficlight.setPhase(TLS_ID, next_phase)
    
    traci.simulationStep()
    
    next_state = env.get_state()
    reward = env.calculate_eco_reward()
    
    return next_state, reward
```
**3. Output, Results, Testcases**

A. Output and Results

<img width="1232" height="719" alt="Screenshot 2025-11-28 135731" src="https://github.com/user-attachments/assets/955ff5c4-ae24-4a20-9a38-ed03eef5b624" />

B. Test Cases and Validation

<img width="1442" height="220" alt="Screenshot 2025-11-28 135739" src="https://github.com/user-attachments/assets/a2fbe868-1d26-4bfe-a73f-a5bab5ea359d" />

The system's functionality and robustness were validated through specific test scenarios:

TC-1: Balanced Peak Hour Load: Tested the core adaptive efficiency and environmental sustainability under heavy, uniform traffic conditions. Validation: Successfully demonstrated the lowest values for congestion and emissions metrics.

TC-2: Emergency Vehicle Protocol: Tested the critical safety feature's immediate response. Validation: Confirmed near-zero delay for the priority vehicle, proving the successful implementation of the override logic.

TC-3: Highly Unbalanced Flow: Tested the agent's robustness and dynamic decision-making under severe traffic asymmetry. Validation: The agent successfully adapted by allocating proportionally more green time to the congested roads, preventing gridlock.

**C. References**
1 . K. Gupta, R. Jain, V. Kumar, “Multi-Agent DDQN with Prioritized Experience Replay for Traffic Signal Optimization,” IET Intelligent Transport Systems, vol. 18, no. 1, pp. 77–89, 2024.
2 . J. Chen, H. Wei, Z. Li, “HiLight: Hierarchically and Cooperatively Learning Traffic Signal Control,” AAAI Conference on Artificial Intelligence, pp. 612–620, 2021.
3 . B. Xu, K. Li, P. Wang, “HumanLight: RL-Based Person-Throughput Optimization for Urban Traffic Signals,” Berkeley Press, vol. 7, 2024.
4 . R. Zhou, X. Yu, C. Chen, “Constrained Traffic Signal Control Under Competing Public Transport Priorities Using Safe RL,” Transportation Research Part C, vol. 168, 2025.
5 . G. Lee, D. Jung, M. Park, “RUTGe: Realistic Urban Traffic Generator for RL-Based Control,” IEEE Trans. Intelligent Transport Systems, vol. 26, 2025.
