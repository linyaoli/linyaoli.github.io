### Steps to solve a DP
#### Identify if it is a DP problem
* Typically DP problems that require to maximize/minimize certain quantity or counting problems under certain conditions.
* DP problems satisfy the overlapping subproblem and optimal substructure property.

#### Decide a state expression with least parameters
* Usually starts with the first few cases. For example, when i = 0, what is state(0); then look into state(1), and go on. You should be able to find connection between the states.

#### Formulate state relationship
* Use the connection you find during proposing examples, the find out the transfer formula between states.

#### Do tabulation (or add memoization)
* Because you will use an array/hash or other structures to save the states, therefore memoization is needed.
