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

### Benchmark Summary (15-Stop Cairo Delivery Route, 10 Runs)
| Algorithm | Mean Distance (km) | Std Dev (km) | Best Distance (km) | Mean Runtime (s) |
| :--- | :---: | :---: | :---: | :---: |
| **Greedy (Nearest Neighbor)** | 18.22 | 0.00 | 18.22 | 0.0001 |
| **Genetic Algorithm (10 Runs)** | 15.84 | 0.60 | 15.34 | 1.2887 |
| **Brute-Force (15 Stops)** | Infeasible ($14!$) | N/A | N/A | > 24 hrs (estimate, not measured) |

* **Optimality Verification ($N = 9$ subset):** The GA matched the exact Brute-Force global minimum of **12.60 km** (outperforming Greedy's 14.04 km).
* **Full Route Efficiency ($N = 15$):** The best of the 10 GA runs reduced total travel distance by **2.88 km (15.8%)** compared to the Greedy baseline. The average GA run reduced it by **2.38 km (13.1%)**.

---

## 7. Team Members
* **Member 1:** *[Rami / 9176391]* 
* **Member 2:** *[Jordan / 7712674]* 
* **Member 3:** *[Taha / 8328997]* 
* **Member 4:** *[Mohammad / 7449148]* 
* **Member 5:** *[Tasnia / 9095822]* 
