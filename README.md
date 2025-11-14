# VLSI-Placement
# Placement in VLSI Design

Placement in VLSI design is a crucial step in the physical design flow, following partitioning and chip planning, and it involves positioning blocks or components on a chip, PCB, or system to optimize area, routing, and performance. Placement occurs at three levels: system-level, where multiple PCBs are arranged to minimize area while ensuring proper heat dissipation; board-level, where chips and devices are positioned on a PCB to minimize routing layers, optimize signal routing, and maintain thermal balance; and chip-level, where the placement of logic gates or standard cells is constrained by the number of routing layers and aims to achieve the smallest possible area while ensuring routability. 

The placement problem is formulated as arranging a set of blocks $B_1, B_2, \dots, B_n$ with known height, width, aspect ratio, and pin locations into iso-oriented rectangles $R_1, R_2, \dots, R_n$ such that no two rectangles overlap ($R_i \cap R_j = \emptyset$), all nets are routable, the total area is minimized, and the total wirelength $L_i$ for nets $N_1, N_2, \dots, N_m$ is minimized. Performance-driven placement also aims to reduce the length of critical paths. Placement strategies depend on design style: in full-custom design, placement resembles a packing problem with irregular blocks, while in semi-custom standard-cell design, cells have uniform height and are arranged row-wise, making channel height and row width minimization equivalent to minimizing area. Gate-array designs assign subcircuits to uniform slots while avoiding overlap and ensuring routability. Placement algorithms can be categorized based on input (constructive or iterative), output (deterministic or probabilistic), and process (simulation-based, e.g., simulated annealing or force-directed placement; partitioning-based, e.g., min-cut; or analytical, e.g., quadratic placement).

Wirelength estimation is a key step in placement, as it directly affects routing and placement quality. For two-pin nets, Manhattan distance is preferred:

$$
d_M(P_1, P_2) = |x_2 - x_1| + |y_2 - y_1|
$$

while Euclidean distance is

$$
E(P_1, P_2) = \sqrt{(x_2 - x_1)^2 + (y_2 - y_1)^2}
$$

For multi-pin nets, several estimation models exist, including Half-Perimeter Wirelength (HPWL), Complete Graph or Clique model:

$$
L_\text{clique} = \frac{2}{p} \sum_e d_M(e)
$$

Monotone Chain, Star, Rectilinear Minimum Spanning Tree (RMST), Rectilinear Steiner Minimum Tree (RSMT), Rectilinear Steiner Arborescence (RSA), and Single Trunk Steiner Tree (STST). Weighted wirelength can be expressed as:

$$
L(P) = \sum_{\text{nets}} W_\text{net} \cdot L_\text{net}
$$

Routing demand is analyzed using maximum cut size, where a net is considered cut if a vertical or horizontal cut line intersects at least one pin on each side, and routing demand is:

$$
L(P) = \sum_{v \in V} \psi(v) + \sum_{h \in H} \psi(h)
$$

Optimized placement reduces cut size, $X(P)$, $Y(P)$, and routing congestion.

Min-cut placement algorithms, such as Kernighan-Lin (KL) and Fiduccia-Mattheyses (FM), partition the layout recursively using alternating horizontal and vertical cuts, ensuring that each subregion eventually contains a unique cell. KL computes a cut difference for node $i$ as:

$$
D_i = E_C(B) - E_{NC}(B)
$$

and swap gain for nodes $A$ and $B$ as:

$$
\Delta G_{A,B} = D_A + D_B - 2 C_{A,B}
$$

FM enhances KL by enforcing partition balance using gain:

$$
\Delta G_C = FS(C) - TE(C)
$$

while satisfying the balance criterion:

$$
R \cdot A_T - \max(A_G) \le \text{Area of partition} \le R \cdot A_T + \max(A_G)
$$

Terminal propagation improves min-cut placement by considering external pins, reducing interconnect length between partitions.

Analytical placement methods, particularly quadratic placement, formulate placement as an optimization problem minimizing total wirelength. The cost function is:

$$
L(P) = \sum_{i=1}^{n} \sum_{j=1}^{n} c_{ij} \big[ (x_i - x_j)^2 + (y_i - y_j)^2 \big]
$$

where $c_{ij} = 1$ if cells $i$ and $j$ are connected, otherwise $0$, and $(x_i, y_i)$ are cell coordinates. X and Y coordinates are optimized independently:

$$
L_x(P) = \sum_{i=1}^{n} \sum_{j=1}^{n} c_{ij} (x_i - x_j)^2, \quad 
L_y(P) = \sum_{i=1}^{n} \sum_{j=1}^{n} c_{ij} (y_i - y_j)^2
$$

The resulting quadratic system:

$$
A_x x = b_x, \quad A_y y = b_y
$$

is solved for global placement, followed by detailed placement to remove overlaps and achieve legal positions.

Force-directed placement models cells and connections as a mass-spring system, with attractive forces between connected cells:

$$
F(A, B) = C_{AB} \cdot (B - A)
$$

and total force on cell $i$ as:

$$
F_i = \sum_{j \ne i, C_{ij} \ne 0} F(i, j)
$$

Cells are iteratively moved to the Zero Force Target (ZFT) position by solving:

$$
\sum_{j \ne i, C_{ij} \ne 0} C_{ij} (x_i - x_j) = 0, \quad
\sum_{j \ne i, C_{ij} \ne 0} C_{ij} (y_i - y_j) = 0
$$

Conflicts are resolved using relocation, chain moves, or ripple moves. While simple and effective for global placement, this method does not scale well for large designs.

Simulated annealing is an iterative optimization technique inspired by thermal annealing, probabilistically accepting worse solutions to escape local minima. Acceptance probability is:

$$
P = e^{-\Delta \text{cost} / T}
$$

where $\Delta \text{cost}$ is the change in objective function and $T$ is the current temperature, which gradually decreases using a cooling factor $\alpha$ $(0 < \alpha < 1)$. Perturb functions generate new placements via moves, swaps, or mirror operations. Placement is optimized using a weighted cost function:

$$
\Gamma = \Gamma_1 + \Gamma_2 + \Gamma_3
$$

where

$$
\Gamma_1 = \sum_\text{net} w_\text{net} H x_\text{net} + w_\text{net} V y_\text{net} \quad (\text{wirelength})
$$

$$
\Gamma_2 = \sum_{i \ne j} O_{i,j}^2 \quad (\text{cell overlap})
$$

$$
\Gamma_3 = \sum_\text{rows} |L_\text{row} - L_\text{opt}| \quad (\text{row length deviation})
$$

The TimberWolf algorithm applies simulated annealing in three steps: initial placement for minimal wirelength, global routing with channel insertion, and local optimization to minimize channel height.

Finally, legalization and detailed placement align cells to power rails, assign discrete legal positions, and ensure no overlaps while providing proper VDD/GND connections. Techniques include neighbor swapping to reduce wirelength and sliding cells within rows to utilize available space. Legalization ensures design-rule-compliant, DRC-safe layouts while minimizing adverse impacts on wirelength, timing, and routing efficiency.
