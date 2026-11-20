# CSCI218 Group Project: Optimizing Last-Mile Delivery Routes Using Genetic Algorithms

**Course:** CSCI218 – Foundations of Artificial Intelligence (Autumn 2026)  
**Institution:** University of Wollongong in Dubai (UOWD)  
**Project Topic:** Topic 9 – Travelling Salesman Problem (TSP) Using Genetic Algorithms  

---

## 1. Project Overview
E-commerce and delivery platforms such as **Talabat** and **noon** regularly assign multiple customer drop-offs to a single driver in one trip. Determining the shortest closed-loop delivery sequence across $N$ stops (leave the start point, visit every stop once, return) is an instance of the **Travelling Salesman Problem (TSP)**, an NP-hard combinatorial optimization problem where evaluating every permutation becomes computationally intractable as $N$ grows.

This repository contains a Python implementation of a **Genetic Algorithm (GA)** applied to delivery locations taken from a synthetic, Talabat-inspired order dataset (`talabat_enhanced_orders.csv`, see Section 3). We evaluate the GA's route quality, convergence behavior, and execution speed against two classical baselines:
1. **Brute-Force Search:** Exact solver used on small subsets ($N \le 9$) to establish the true global minimum distance.
2. **Greedy Nearest-Neighbor Search:** Fast $O(N^2)$ heuristic baseline used to benchmark route distance savings.

---

## 2. Repository Structure
```text
├── 218_Project.ipynb              # Main Jupyter Notebook (data processing, algorithms, experiments, plots)
├── talabat_enhanced_orders.csv    # Synthetic delivery-order dataset (source in Section 3)
├── requirements.txt               # Python packages needed to run the notebook
├── results/                       # CSV tables and figures written by Steps 6 and 8 of the notebook
└── README.md                      # Project documentation and setup instructions
```

---

## 3. Dataset Description
* **File:** `talabat_enhanced_orders.csv` (100,000 orders, 23 columns, 7 Egyptian cities)
* **Source:** Waddah Ali, *Order Delivery Dataset*, Kaggle, licensed **CC BY-SA 4.0**: <https://www.kaggle.com/datasets/waddahali/order-delivery-dataset>
* **Synthetic data:** Kaggle labels this dataset as synthetically generated. It is a simulation inspired by Talabat, **not** real Talabat operational data. Within each city, the coordinates are spread evenly over a box of roughly 4.4 km × 3.9 km around the city centre. We use it as a reproducible source of delivery-stop locations.
* **Features Used:** `City`, `Order_ID`, `Item_Name`, `Driver_Lat`, `Driver_Lon`, `Customer_Lat`, `Customer_Lon`.
* **Experimental Setup:** 
  * Orders are filtered by city (default: **Cairo**).
  * **Node 0** is the start and return point of the route, taken from the driver location (`Driver_Lat`, `Driver_Lon`) of the first sampled order.
  * **Nodes 1–14** represent 14 customer drop-off locations (`Customer_Lat`, `Customer_Lon`).
  * Straight-line (great-circle) distances in kilometers between all node pairs are computed with the **Haversine formula** to build a symmetric $15 \times 15$ distance matrix. These are not road distances.

---

## 4. Algorithms & Parameters
* **Genetic Algorithm (GA):**
  * **Representation:** Permutation list of node indices starting and ending at Node `0`.
  * **Fitness Function:** $\text{Fitness} = \frac{1}{\text{Total Route Distance (km)}}$
  * **Selection:** Tournament Selection ($k = 5$)
  * **Crossover:** Ordered crossover with rate `0.85`. It copies a random slice from parent 1, then fills the remaining positions left to right with parent 2's stops in their original order. This keeps every child a valid route with no duplicate stops.
  * **Mutation:** Inversion Mutation with rate `0.15` (reverses sub-route segments to untangle crossing edges)
  * **Elitism:** Top `4` individuals carried over unchanged per generation
  * **Default Hyperparameters:** `pop_size = 120`, `generations = 400`
* **Baselines:**
  * **Brute-Force:** Evaluates all $(N-1)!$ permutations via `itertools.permutations`.
  * **Greedy Search:** Starts at Node 0 and repeatedly moves to the closest unvisited stop.

---

## 5. Setup & Installation

### Prerequisites
* **Python 3** (tested with Python 3.12)
* The packages in `requirements.txt`: `numpy`, `pandas`, `matplotlib`, `jupyter`

### Local Installation
1. Clone the repository:
   ```bash
   git clone https://github.com/RamiKH1/218_Group4_TSP_Genetic_Algorithm.git
   cd 218_Group4_TSP_Genetic_Algorithm
   ```
2. Install the required Python packages:
   ```bash
   pip install -r requirements.txt
   ```
3. Launch Jupyter Notebook from the repository folder (the notebook loads the CSV by a relative path):
   ```bash
   jupyter notebook 218_Project.ipynb
   ```
4. Select **Kernel > Restart & Run All**.

### Running on Google Colab
1. Open `218_Project.ipynb` in Google Colab.
2. Upload `talabat_enhanced_orders.csv` into the Colab working directory (`/content/`).
3. Select **Runtime > Run all** to execute all cells sequentially.

---

## 6. Reproducibility & Key Results
To make results reproducible, fixed random seeds are applied to both Python's `random` module and `numpy.random`: `SEED = 42` at the start, and `SEED + run` for each repeated GA run.

Run times depend on the computer. All times below come from one run of the notebook on our test PC.

### Benchmark Summary (15-Stop Cairo Delivery Route, 10 Runs)
| Algorithm | Mean Distance (km) | Std Dev (km) | Best Distance (km) | Mean Runtime (s) |
| :--- | :---: | :---: | :---: | :---: |
| **Greedy (Nearest Neighbor)** | 18.22 | 0.00 | 18.22 | 0.00003 |
| **Genetic Algorithm (10 Runs)** | 15.84 | 0.60 | 15.34 | 0.53 |
| **Brute-Force (15 Stops)** | Infeasible ($14!$) | N/A | N/A | at least 3 days (estimated in Step 6) |

* **Optimality Verification ($N = 9$ subset):** The GA matched the exact Brute-Force global minimum of **12.60 km** (outperforming Greedy's 14.04 km).
* **Full Route Efficiency ($N = 15$):** The best of the 10 GA runs reduced total travel distance by **2.88 km (15.8%)** compared to the Greedy baseline. The average GA run reduced it by **2.39 km (13.1%)**.

### Step 6: 10 Delivery Batches (Cairo, 2 June 2025, 08:00–17:00)
Each batch takes the non-cancelled Cairo orders from one hour. Every route starts and ends at a hub in the centre of the Cairo delivery area. The GA keeps its default settings and runs 10 times per batch (seeds 42–51). Full tables are in `results/`.

| Experiment | Result |
| :--- | :--- |
| **A. GA vs exact optimum** (hub + 9 drop-offs) | The GA found the exact shortest route in **86 of 100 runs**. Mean gap to the optimum: **0.26%** (worst run 8.68%), against **7.20%** for greedy. The GA's route was longer than greedy's in 4 of 100 runs. |
| **B. Brute-force time** | 11 stops (3.6 million routes) took 10.9 s. Estimated at **at least 3 days for 15 stops** and over 200,000 years for 21 stops. |
| **C. GA vs greedy** (hub + 20 drop-offs) | The GA saved **9.4%** distance on average (range −6.0% to 22.7%): **86 wins, 3 ties, 11 losses** in 100 runs. Its average route was shorter in 9 of 10 batches. One GA run takes about 0.9 s. |

Figures: `results/route_over_generations.png` (best route so far at generations 1, 10, 25, 50, 100 and 400 of one run) and `results/experiment_summary.png` (brute-force time vs stops, and greedy vs GA per batch).

### Step 7: Tests
18 checks of the building blocks all pass: haversine (latitude and longitude), route length, brute force, greedy, population, crossover, mutation, tournament selection, and the full GA on 6 stops (starting from 10 random routes that miss the optimum, it evolves to the brute-force optimum). The GA needs at least 3 stops (hub + 2 drop-offs). With only 2 stops, crossover and mutation cannot pick two different positions, so it stops with an error.

### Step 8: Mutation Rate (Experiment D)
The same 10 batches, 5 seeds each (42–46). Only the mutation rate changes; population 120 and 400 generations stay the same.

| Mutation rate | Exact optimum found (10 stops) | Mean gap to optimum (10 stops) | Mean saving over greedy (21 stops) |
| :---: | :---: | :---: | :---: |
| 0.0 | 28% | 3.32% | −13.6% |
| **0.15 (default)** | 84% | 0.47% | 9.5% |
| 0.5 | 100% | 0.00% | 10.8% |
| 0.8 | 100% | 0.00% | 11.5% |

Without mutation the GA stops improving early (median generation 16 on 21 stops), and its 21-stop routes are on average 13.6% longer than greedy's (worse in 38 of 50 runs). On 10 stops it finds the exact optimum in only 28% of runs. Our default rate of 0.15 is not the best rate we tried: 0.5 and 0.8 found the exact optimum in every 10-stop run, and 0.8 gave the largest saving on 21 stops. Step 6 keeps 0.15 so the main results are not tuned on the batches they are reported on, which makes them slightly pessimistic for the GA. The rates were compared on the same batches, with the other settings fixed, so the gain may be a little smaller on new batches. In a quick exploratory run, crossover rate and elitism changed the results much less, so only the mutation rate is shown. Figure: `results/mutation_rate_study.png`.

In a separate full run, the whole notebook took about 9 minutes on our test PC, about 6 of them for Step 8.

---

## 7. Team Members
* **Member 1:** *[Rami / 9176391]* 
* **Member 2:** *[Jordan / 8328997]* 
* **Member 3:** *[Taha / 8328997]* 
* **Member 4:** *[Mohammad / 7449148]* 
* **Member 5:** *[Tasnia / 9095822]* 
