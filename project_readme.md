# CSCI218 Group Project: Optimizing Last-Mile Delivery Routes Using Genetic Algorithms

**Course:** CSCI218 – Foundations of Artificial Intelligence (Autumn 2026)  
**Institution:** University of Wollongong in Dubai (UOWD)  
**Project Topic:** Topic 9 – Travelling Salesman Problem (TSP) Using Genetic Algorithms  

---

## 1. Project Overview
E-commerce and quick-delivery platforms such as **Talabat** routinely assign multiple customer drop-offs to a single driver in one trip. Determining the shortest closed-loop delivery sequence across $N$ stops is an instance of the **Travelling Salesman Problem (TSP)**—an NP-hard combinatorial optimization problem where evaluating every permutation becomes computationally intractable as $N$ grows.

This repository contains a Python implementation of a **Genetic Algorithm (GA)** applied to real-world delivery geolocation data (`talabat_enhanced_orders.csv`). We evaluate the GA's route quality, convergence behavior, and execution speed against two classical baselines:
1. **Brute-Force Search:** Exact solver used on small subsets ($N \le 9$) to establish the true global minimum distance.
2. **Greedy Nearest-Neighbor Search:** Fast $O(N^2)$ heuristic baseline used to benchmark route distance savings across full multi-stop delivery shifts.

---

## 2. Repository Structure
```text
├── 218_Project.ipynb              # Main Jupyter Notebook (data processing, algorithms, experiments, plots)
├── talabat_enhanced_orders.csv    # Real-world Talabat delivery order dataset (GPS coordinates)
└── README.md                      # Project documentation and setup instructions
```

---

## 3. Dataset Description
* **File:** `talabat_enhanced_orders.csv`
* **Features Used:** `City`, `Order_ID`, `Item_Name`, `Driver_Lat`, `Driver_Lon`, `Customer_Lat`, `Customer_Lon`.
* **Experimental Setup:** 
  * Orders are filtered by city (default: **Cairo**).
  * **Node 0** represents the driver's initial dispatch location (`Driver_Lat`, `Driver_Lon`).
  * **Nodes 1–14** represent 14 customer drop-off locations (`Customer_Lat`, `Customer_Lon`).
  * Real-world spherical distances (in kilometers) between all node pairs are computed using the **Haversine formula** to build a symmetric $15 \times 15$ distance matrix.

---

## 4. Algorithms & Parameters
* **Genetic Algorithm (GA):**
  * **Representation:** Permutation list of node indices starting and ending at Node `0`.
  * **Fitness Function:** $\text{Fitness} = \frac{1}{\text{Total Route Distance (km)}}$
  * **Selection:** Tournament Selection ($k = 5$)
  * **Crossover:** Ordered Crossover (OX) with rate `0.85` (preserves permutation validity without duplicate stops)
  * **Mutation:** Inversion Mutation with rate `0.15` (reverses sub-route segments to untangle crossing edges)
  * **Elitism:** Top `4` individuals carried over unchanged per generation
  * **Default Hyperparameters:** `pop_size = 120`, `generations = 400`
* **Baselines:**
  * **Brute-Force:** Evaluates all $(N-1)!$ permutations via `itertools.permutations`.
  * **Greedy Search:** Iteratively selects the closest unvisited stop from the current node.

---

## 5. Setup & Installation

### Prerequisites
Ensure you have **Python 3.8+** installed along with the following dependencies:
* `numpy`
* `pandas`
* `matplotlib`
* `jupyter` (or Google Colab)

### Local Installation
1. Clone the repository:
   ```bash
   git clone <your-github-repository-url>
   cd <repository-folder-name>
   ```
2. Install the required Python packages:
   ```bash
   pip install numpy pandas matplotlib jupyter
   ```
3. Launch Jupyter Notebook:
   ```bash
   jupyter notebook 218_Project.ipynb
   ```

### Running on Google Colab
1. Open `218_Project.ipynb` in Google Colab.
2. Upload `talabat_enhanced_orders.csv` into the Colab working directory (`/content/`).
3. Select **Runtime > Run all** to execute all cells sequentially.

---

## 6. Reproducibility & Key Results
To guarantee deterministic and reproducible experimental results, fixed random seeds (`SEED = 42`, incrementing sequentially across repeated runs `SEED + run`) are applied to both Python's `random` module and `numpy.random`.

### Benchmark Summary (15-Stop Cairo Delivery Route, 10 Runs)
| Algorithm | Mean Distance (km) | Std Dev (km) | Best Distance (km) | Mean Runtime (s) |
| :--- | :---: | :---: | :---: | :---: |
| **Greedy (Nearest Neighbor)** | 18.22 | 0.00 | 18.22 | 0.0001 |
| **Genetic Algorithm (10 Runs)** | 15.84 | 0.60 | 15.34 | 1.2887 |
| **Brute-Force (15 Stops)** | Infeasible ($14!$) | N/A | N/A | > 24 hrs |

* **Optimality Verification ($N = 9$ subset):** The GA matched the exact Brute-Force global minimum of **12.60 km** (outperforming Greedy's 14.04 km).
* **Full Route Efficiency ($N = 15$):** The GA reduced total travel distance by **2.88 km (15.8%)** per delivery trip compared to the Greedy baseline.

---

## 7. Team Members
* **Member 1:** *[Name & Student ID]* – Dataset preprocessing, Haversine distance matrix, README documentation
* **Member 2:** *[Name & Student ID]* – Brute-Force and Greedy baseline implementations
* **Member 3:** *[Name & Student ID]* – Genetic Algorithm core operators (Selection, Ordered Crossover, Inversion Mutation)
* **Member 4:** *[Name & Student ID]* – Statistical benchmarking (10-run evaluation) and Matplotlib visualizations
* **Member 5:** *[Name & Student ID]* – Hyperparameter experimentation and report/presentation integration