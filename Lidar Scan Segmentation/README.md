# Lidar Scan Segmentation with Sequential RANSAC

## 🎯 Overview

This project implements a Sequential RANSAC (Random Sample Consensus) algorithm for detecting and fitting geometric primitives—**line segments** (representing walls/boundaries) and **circular arcs** (representing poles/cylindrical objects)—to 2D Lidar point cloud data.

The methodology focuses on robust model estimation, including optimal line fitting using SVD and nonlinear least-squares refinement for circles. Results are visualized by generating color-coded PLY files compatible with tools like MeshLab.

## 🚀 Key Features

* **Data Preprocessing:** Load polar coordinates (degrees, mm) and convert them to Cartesian coordinates (meters).
* **Sequential RANSAC:** Iterative extraction and removal of detected structures. **The process is repeated until no new line/circle segments are found.**
* **Optimal Line Fitting:** Uses Total Least Squares (SVD) for highly accurate, geometrically consistent line models, fulfilling the requirement for the **optimal method** for line fitting.
* **Robust Circle Fitting:** Employs Nonlinear Least Squares (`scipy.optimize.least_squares`) to accurately estimate circle center and radius.
* **Visualization:** Outputting the segmented point cloud in **PLY format** with specific color-coding (Blue for lines, Red for circles).

## 🛠️ Implementation Details

### 1. Data Loading and Conversion
Points are read from the text files (angle in degrees, radius in mm) and converted to meters for use in RANSAC:
$$
x = \frac{r}{1000} \cos(\theta), 

y = \frac{r}{1000} \sin(\theta)
$$


### 2. Model Fitting Parameters and Constraints

The algorithm uses specific, assignment-driven criteria to validate detected models.

| Model | Minimal Sample Set (MSS) | Refitting Method | Minimum Inliers | Constraints |
| :--- | :--- | :--- | :--- | :--- |
| **Line Segment** | 2 points | Total Least Squares (SVD) | 10 | Length $\ge 0.4 \text{ m}$ |
| **Circular Arc**| 3 non-collinear points | Nonlinear Least Squares | 8 | Radius $R \in [0.2, 0.5] \text{ m}$. **Note:** Due to self-occlusion, only an arc is typically visible. |

### 3. Sequential RANSAC Strategy

The core strategy is to iterate, attempting to find the largest, most valid line *or* circle model in the remaining data before removing its inliers.

1.  **Candidate Search:** In each major iteration, RANSAC is run separately to find the best line model and the best circle model.
2.  **Refinement:** The inliers found by RANSAC are used to **re-estimate** the model parameters using the optimal fitting methods (SVD/Least Squares).
3.  **Validation:** The refined models are checked against the constraints (inlier count, length/radius).
4.  **Extraction:** The inliers belonging to the validated line and circle models are removed from the remaining point cloud.
5.  **Termination:** The overall sequential process stops when neither a line nor a circle can satisfy its validation criteria.



## 📝 Results Summary

The algorithm was executed on the three required files: **$x+1$**, **$x+11$**, and **$x+21$**.

### Total Run Summary

| Metric | Value |
| :--- | :--- |
| Files Processed | 3 |
| Total Points | 2408 |
| Total Lines Detected | 19 |
| Total Circles Detected | 2 |

### Detailed Results per File

| File | Total Points | Detected Models | Longest Line (m) | Detected Circle Center/Radius |
| :--- | :--- | :--- | :--- | :--- |
| **1** | 764 | 8 (7 Lines, 1 Circle) | $2.84 \text{ m}$ | $(0.096, -0.061)$, $R=0.465 \text{ m}$ |
| **11**| 788 | 5 (5 Lines, 0 Circles) | $19.75 \text{ m}$ | None |
| **21**| 856 | 8 (7 Lines, 1 Circle) | $27.72 \text{ m}$ | $(0.777, -0.015)$, $R=0.299 \text{ m}$ |

### Visualization

The final PLY output uses the following color scheme:

* **Line Inliers:** **Blue**
* **Circle Inliers:** **Red**
* **Other Points:** **White**

---
#### Result for File **1.ply**
A scene with multiple smaller line segments and one distinct circular object.
[Placeholder for the image of the result visualization for File 1, showing red circle inliers and blue line inliers]

---
#### Result for File **11.ply**
A scene dominated by very long line segments (up to $19.75 \text{ m}$), likely representing an open area.
[Placeholder for the image of the result visualization for File 11, showing long blue line segments]

---
#### Result for File **21.ply**
A scene combining long line segments with one clear circular object detected at $R \approx 0.299 \text{ m}$.
[Placeholder for the image of the result visualization for File 21, showing lines and a circle]

## 🧑‍💻 Contribution Statement (AI Use Disclosure)

| Component | My Contribution (Kipchirchir Raphael) | AI (ChatGPT/Grok AI) Contribution |
| :--- | :--- | :--- |
| **Initial Implementation** | Wrote the initial code for file loading, **polar-to-Cartesian conversion**, and the first draft of the RANSAC algorithms. | Guided the **Code Refactoring** to simplify and optimize function structure and improve variable naming and modular design. |
| **Core Logic** | Designed the overall Sequential RANSAC pipeline that iteratively extracts shapes and implements **re-estimation of models**. | Provided **Debugging Assistance**, identifying numerical edge cases (like division-by-zero) and suggesting corrections for circle parameter estimation. |
| **Validation & Output** | Set the specific **selection of thresholds**, implemented the line length/radius bounds, and wrote the initial **PLY export function**. | Assisted in fixing validation bounds and helped correct the **PLY file structure** so that Meshlab loads it correctly. |
| **Documentation** | Performed all testing, verification, and wrote the initial project text. | Generated clear **comments and docstrings**, and helped ensure the implementation meets all assignment requirements. |

**Prompts used with AI included:** 
1. "Refactor my RANSAC code to improve structure and readability," 
2. "Help me debug errors in circle fitting and line fitting," 
3. "Suggest improvements for re-estimating circle and line models using inliers," 
4. "Explain how to correctly write PLY files with RGB colors," 
5. "Check if my implementation meets the assignment requirements," 
6. "Add clear comments and docstrings to my code."

**AI did not write the entire project. All algorithmic decisions, logic, testing, selection of thresholds, and implementation-specific details were performed and verified by me.**

---
## 📦 Running the Code

### Prerequisites

* Python >= 3.11
* Required libraries: `numpy`, `scipy`, `tqdm`, `matplotlib`.

```bash
pip install numpy scipy tqdm matplotlib