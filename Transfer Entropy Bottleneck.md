### $sin$ Wave Data Generation:
#### `get_A` Functions:
```python
get_A(k, fs=20.)
```
The function `get_A(k, fs=20.)` returns a 2x2 matrix $A$ which is defined as:

$$
A = \begin{bmatrix}
0 & -\frac{2\pi k}{f_s} \\
\frac{2\pi k}{f_s} & 0
\end{bmatrix}
$$
where:

- $k$ is a variable that could represent a frequency-related parameter.
- $f_s$ is the sampling frequency.

This matrix is a skew-symmetric matrix because $A^T = -A$, where  $A^T$ is the transpose of  $A$ . The off-diagonal elements $\frac{2\pi k}{f_s}$ and $-\frac{2\pi k}{f_s}$ are the negatives of each other, while the diagonal elements are zero. This matrix structure is commonly associated with rotational dynamics in the phase space of a system, particularly in the context of linear time-invariant systems.
___
#### Data set Generation
Variables notations:
- `num_dataset['train']` = $N$ (number of samples to generate for the training set)
- `precompute_len_y` = $T_y$ (length of the initial time sequence for which the system is observed without switching)
- `precompute_len_yp` = $T_{yp}$ (length of the subsequent time sequence where a switch in angular velocity may occur)
- `angularVelo_mul` = $\vec{v} = [v_1, v_2, v_3, v_4, v_5]$ (the set of angular velocity multipliers)
- `switch_probability` = $P_{swt}$ (probability of a switch in angular velocity)

The training dataset $D_{train}$ is constructed as follows:
1. **Initialization**: A dataset $D_{train}$ is initialized as a zero tensor with dimensions $N \times (T_y + T_{yp}) \times 5$, where the last dimension corresponds to the number of different angular velocities.
2. **Simulation Loop**: For each sample $i$ from 1 to $N$:
   a. A permutation $\pi$ of the indices of $\vec{v}$ is generated without replacement.
   b. A potential switch index $idx_{swt}$ is determined based on a random draw from a uniform distribution; if the drawn value is less than $P_{swt}$, then a switch is scheduled.
   c. The system's initial state $\vec{x}_0$ is determined by a random angle $w$ uniformly drawn from [0, 2π].
   d. For each angular velocity $v_m$ defined by $\pi$:
      i. A matrix $A_m$ is generated using $v_m$: 
      $$
      A_m = 
      \begin{bmatrix}
      0 & -2 \cdot \pi \cdot v_m / f_s \\
      2 \cdot \pi \cdot v_m / f_s & 0
      \end{bmatrix}
      $$
      where $f_s$  is the sampling frequency.
      ii. The Ordinary Differential Equation (ODE) is solved using $A_m$ and $\vec{x}_0$ to obtain the system state $\vec{x}(t)$ over $t \in [0, T_y]$.
      iii. If a switch is scheduled at $idx_{swt}$, the ODE is solved again using a new angular velocity multiplier for $t \in [T_y, T_y + T_{yp}]$.
3. **Storage**: The resulting system state time series and the corresponding labels (angular velocity multipliers) for each time point are stored in $D_{train}$.
The mathematical representation for the entire training dataset can be expressed as a set of ordered pairs $(S, L)$ where $S$ is the simulated state and $L$ is the label:
$$
D_{train} = \{(\vec{x}_i(t), l_i(t)) \mid t \in [1, T_y + T_{yp}], i \in [1, N]\}
$$
Here, $\vec{x}_i(t)$ represents the state of the $i$-th sample at time $t$, and $l_i(t)$ represents the corresponding label which is the angular velocity multiplier in effect at time $t$. For $t > T_y$ and if $idx_{swt} = i$, $l_i(t)$ may switch to a different value from $\vec{v}$.