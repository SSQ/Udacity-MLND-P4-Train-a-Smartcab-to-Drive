# Reinforcement Learning
## Project: Train a Smartcab to Drive
### Files Description
It contains four files:
- `project description.md`: Project overview, highlights, evaluation and software requirement. **Read Firstly**
- `README.md`: this file.
- `smartcab.ipynb`: This is the main file where you will answer questions and provide an analysis for your work.
- `visuals.py`: This Python script provides supplementary visualizations for the analysis. Do not modify.

This project also contains three directories:
- `/logs/`: This folder will contain all log files that are given from the simulation when specific prerequisites are met.
- `/images/`: This folder contains various images of cars to be used in the graphical user interface. You will not need to modify or create any files in this directory.
- `/smartcab/`: This folder contains the Python scripts that create the environment, graphical user interface, the simulation, and the agents. You will not need to modify or create any files in this directory except for `agent.py`.

Finally, in `/smartcab/` are the following four files:
- **My work**
    - `agent.py`: This is the main Python file where you will be performing your work on the project.
- **Not my work**
    - `environment.py`: This Python file will create the *smartcab* environment.
    - `planner.py`: This Python file creates a high-level planner for the agent to follow towards a set goal.
    - `simulation.py`: This Python file creates the simulation and graphical user interface. 

### Run
In a command window (OS: Win7), navigate to the top-level project directory `smartcab/` (that contains the two project directories) and run one of the following commands:

`python smartcab/agent.py` or  
`python -m smartcab.agent`

This will run the `agent.py` file and execute your implemented agent code into the environment. Additionally, use the command `jupyter notebook smartcab.ipynb` from this same directory to open up a browser window or tab to work with your analysis notebook. Alternatively, you can use the command `jupyter notebook` or `ipython notebook` and navigate to the notebook file in the browser window that opens. Follow the instructions in the notebook and answer each question presented to successfully complete the implementation necessary for your `agent.py` agent file. A **README** file has also been provided with the project files which may contain additional necessary information or instruction for the project.
## Project Implementation
### Getting Started
#### Understand the World
Provide a thorough discussion of the driving agent as it interacts with the environment.
- Does the Smartcab move at all during the simulation?
- What kind of rewards is the driving agent receiving?
- How does the light changing color affect the rewards?
```
1. The Smartcab idles all the time no matter the light is red or green.
1. According to red and green, the driving agent receives positive and negative rewards respectly
    - green light with other traffic: small positive rewards
    - green light without other traffic: big negative rewards
    - red light: positive rewards
```
#### Understand the Code
- In the `agent.py` Python file, choose three flags that can be set and explain how they change the simulation.
- In the `environment.py` Python file, what Environment class function is called when an agent performs an action?
- In the `simulator.py` Python file, what is the difference between the `'render_text()'` function and the `'render()'` function?
- In the `planner.py` Python file, will the `next_waypoint()` function consider the North-South or East-West direction first?
```
1. 
    - learning: Whether the agent is expected to learn
    
    - testing: 'testing' is set to True if testing trials are being used once training trials have completed.
2. act
3.
    - render_text(): This is the non-GUI render display of the simulation.Simulated trial data will be rendered in the terminal/command prompt.
    - render(): This is the GUI render display of the simulation. Supplementary trial data can be found from render_text.
4. East-West direction first
```

### Implement a Basic Driving Agent
#### Basic Agent
- Agent accepts inputs
- Produces a valid output
- Runs in simulator
Driving agent produces a valid action when an action is required. Rewards and penalties are received in the simulation by the driving agent in accordance with the action taken.
in `choose_action`
```
# Choose a random action
action = random.choice(self.valid_actions)
```

#### Basic Agent Simulation Analysis
Summarize observations about the basic driving agent and its behavior. Optionally, if a visualization is included, analysis is conducted on the results provided.
### Inform the Driving Agent
#### Identify States
Justify a set of features that best model each state of the driving agent in the environment. Unnecessary features not included in the state (if applicable) are similarly justified. 

Inspecting the 'build_state()' agent function shows that the driving agent is given the following data from the environment:
- 'waypoint', which is the direction the Smartcab should drive leading to the destination, relative to the Smartcab's heading.
- 'inputs', which is the sensor data from the Smartcab. It includes
    - 'light', the color of the light.
    - 'left', the intended direction of travel for a vehicle to the Smartcab's left. Returns None if no vehicle is present.
    - 'right', the intended direction of travel for a vehicle to the Smartcab's right. Returns None if no vehicle is present.
    - 'oncoming', the intended direction of travel for a vehicle across the intersection from the Smartcab. Returns None if no vehicle is present.
- 'deadline', which is the number of actions remaining for the Smartcab to reach the destination before running out of time.
Questions:
- Which features available to the agent are most relevant for learning both safety and efficiency? 
- Why are these features appropriate for modeling the Smartcab in the environment? 
- If you did not choose some features, why are those features not appropriate?
```
waypoint:
    relevant for efficiency not safety
inputs:
    relevant for safety not efficiency
deadline
    relevant for efficiency not safety
If I have three opitions, I choose waypoint and inputs, for both safety and efficiency. If I have two options, I will not choose deadline, because I only need to know the right direction.
    state = (inputs['oncoming'], inputs['light'], waypoint)
    I choose these 3 features and omit inputs['right'],inputs['left'] because 3 features have less states than 5 features which needs less training trials and these 3 features are the most relevant to both safety and efficiency
```
#### State Space
The total number of possible states is correctly reported. 

Discuss whether the driving agent could learn a feasible policy within a reasonable number of trials for the given state space.

If a state is defined using the features you've selected from Question 4, what would be the size of the state space? Given what you know about the evironment and how it is simulated, do you think the driving agent could learn a policy for each possible state within a reasonable number of training trials?

- I choose waypoint, inputs[light] and inputs[oncoming].
- waypoint is [forward,left,right], inputs is [[red, green],[left,right,none,forward]], 24 = 3*2*4.
- No.Number of training trials is 20, while the number of state is 24.

#### Update Driving Agent State
The driving agent successfully updates its state based on the state definition and input provided.
in `build_state`
```
# Set 'state' as a tuple of relevant data for the agent        
state = (inputs['oncoming'], inputs['light'], waypoint)
```

### Implement a Q-Learning Driving Agent
#### Q-Learning Agent
- Agent updates Q-values
- Picks the best action
- Agent correctly implements a "tie- breaker" between best actions
- Correct implement eplison decay and exploration (agent takes a random action with epsilon probability)
- Implement required 'learning' flags

The driving agent: 
(1) Chooses best available action from the set of Q-values for a given state. (2) Implements a 'tie-breaker' between best actions correctly 
(3)Updates a mapping of Q-values for a given state correctly while considering the learning rate and the reward or penalty received. 
(4) Implements exploration with epsilon probability 
(5) implements are required 'learning' flags correctly

For this project, you will be implementing a decaying, $\epsilon$-greedy Q-learning algorithm with no discount factor. 

```
    def reset(self, destination=None, testing=False):
        """ The reset function is called at the beginning of each trial.
            'testing' is set to True if testing trials are being used
            once training trials have completed. """

        # Select the destination as the new location to route to
        self.planner.route_to(destination)
        
        ########### 
        ## TO DO ##
        ###########
        # Update epsilon using a decay function of your choice
        # Update additional class parameters as needed
        # If 'testing' is True, set epsilon and alpha to 0
        self.epsilon = self.epsilon -0.05
        if testing:
            self.epsilon = 0
            self.alpha = 0
        return None
```

```
def get_maxQ(self, state):
        """ The get_max_Q function is called when the agent is asked to find the
            maximum Q-value of all actions based on the 'state' the smartcab is in. """

        ########### 
        ## TO DO ##
        ###########
        # Calculate the maximum Q-value of all actions for a given state
        maxQ = 0

        for action in self.valid_actions:
            if self.Q[state][action] > maxQ:
                maxQ = self.Q[state][action]
               
        return maxQ
```

```
    def createQ(self, state):
        """ The createQ function is called when a state is generated by the agent. """

        ########### 
        ## TO DO ##
        ###########
        # When learning, check if the 'state' is not in the Q-table
        # If it is not, create a new dictionary for that state
        #   Then, for each action available, set the initial Q-value to 0.0
        if self.learning:
            if state not in self.Q:
                #print "state: ",state
                self.Q[state] = {}
                for actions in self.valid_actions:
                    #print "actions: ",actions
                    self.Q[state][actions] = 0.0
                    #print "self.Q: ",self.Q

        return
```

```
 def choose_action(self, state):
        """ The choose_action function is called when the agent is asked to choose
            which action to take, based on the 'state' the smartcab is in. """

        # Set the agent state and default action
        self.state = state
        self.next_waypoint = self.planner.next_waypoint()
        
        action = None
        total_action = []
        ########### 
        ## TO DO ##
        ###########
        # When not learning, choose a random action
        if self.learning == False:
            action = random.choice(self.valid_actions)
        # When learning, choose a random action with 'epsilon' probability
        elif self.epsilon > random.random():
            action = random.choice(self.valid_actions)
        #   Otherwise, choose an action with the highest Q-value for the current state
        else:
            for final_action in self.valid_actions:
                if self.Q[state][final_action] >= self.get_maxQ(state):
                    print "final_action: ",final_action
                    total_action.append(final_action)
                    print "total_action: ",total_action
                    action = random.choice(total_action)
        print "action: ",action
        return action
```

```
 def learn(self, state, action, reward):
        """ The learn function is called after the agent completes an action and
            receives an award. This function does not consider future rewards 
            when conducting learning. """

        ########### 
        ## TO DO ##
        ###########
        # When learning, implement the value iteration update rule
        #   Use only the learning rate 'alpha' (do not use the discount factor 'gamma')
        if self.learning == True:
            self.Q[state][action] = ((1.0 - self.alpha) * self.Q[state][action]) + ((reward)* self.alpha)#  + self.get_maxQ(state)
        
        return

```
#### Q-Learning Agent Simulation Analysis
Summarize observations about the initial/default Q-Learning driving agent and its behavior, and compares them to the observations made about the basic agent. If a visualization is included, analysis is conducted on the results provided.

- Are there any observations that are similar between the basic driving agent and the default Q-Learning agent?
- Approximately how many training trials did the driving agent require before testing? Does that number make sense given the epsilon-tolerance?
- Is the decaying function you implemented for $\epsilon$ (the exploration factor) accurately represented in the parameters panel?
- As the number of training trials increased, did the number of bad actions decrease? Did the average reward increase?
- How does the safety and reliability rating compare to the initial driving agent?

```
1. Their total bad actions are decreasing. Both reward per action increase
1. 20 training trials. It make sense,we keep training until epsilon less than tolerance.
1. Yes,from 0.95 to 0
    - yes, from the image
    - yes, from the image
1. It does better.
```
### Improve the Q-Learning Driving Agent
#### Improved Q-Learning Agent
Improvements reported

The driving agent performs Q-Learning with alternative parameters or schemes beyond the initial/default implementation.

in `reset`
```
self.epsilon = self.epsilon *0.99
```
#### Improved Q-Learning Agent Simulation Analysis
Summarize observations about the optimized Q-Learning driving agent and its behavior, and further compares them to the observations made about the initial/default Q-Learning driving agent. If a visualization is included, analysis is conducted on the results provided.

- What decaying function was used for epsilon (the exploration factor)?
- Approximately how many training trials were needed for your agent before begining testing?
- What epsilon-tolerance and alpha (learning rate) did you use? Why did you use them?
- How much improvement was made with this Q-Learner when compared to the default Q-Learner from the previous section?
- Would you say that the Q-Learner results show that your driving agent successfully learned an appropriate policy?
- Are you satisfied with the safety and reliability ratings of the Smartcab?

```
1. $$\epsilon = a^t, \textrm{for } a = 0.99$$
1. approximately 300
1. tolerance is the default 0.05, alpha is the default 0.5. With this epsilon-tolerance, I have 25 testing trials.
1. From A+/B to A+/A+. Frequency of all bad actions decrease in a very low rate,reward per action get nearly 2,rate of reliability approaches to nearly 1 with the decreasing of training trials.
1. Yes, but I might need to use inputs[left] and inputs[right] to do something more
1. Yes, but I might try more features and more testing trials
```
#### Safety and Reliability
The driving agent is able to safely and reliably guide the Smartcab to the destination before the deadline.
#### Define an Optimal Policy
Describe what an optimal policy for the driving agent would be for the given environment. 
The policy of the improved Q- Learning driving agent is compared to the stated optimal policy. 
Present entries from the learned Q-table that demonstrate the optimal policy and sub- optimal policies. If either are missing, discussion is made as to why.

```
inputs['oncoming'], inputs['light'], waypoint
('left', 'green', 'forward')
-- forward : 1.79
-- right : 1.18
-- None : 0.47
-- left : 0.72
this is correct, oncoming is left,light is green, we need to forward, so the reward of forward is the highest.
```
#### Future Rewards
Correctly identify the two characteristics about the project that invalidate the use of future rewards in the Q-Learning implementation.

1. the agent cannot plan ahead to find a route to the destination
2. Since both the smartcab's initial position and the destination throughout the trials runs


