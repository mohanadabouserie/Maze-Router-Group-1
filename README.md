# Maze-Router-Group-1

This project implements a maze router designed for a two-layer grid-based routing environment. The router establishes optimized paths between multiple points (nets) while navigating obstacles, adhering to physical constraints, and minimizing routing costs. It also implements a net ordering heuristic as a bonus feature.

## Implementation

### Overview

The maze router operates on a NxM grid with two layers:

- Layer M0: Optimized for horizontal routing.
- Layer M1: Optimized for vertical routing.

### Key features include:

- **Input Parsing**: Reads grid dimensions, penalties, obstructions, and nets from a text file.

- **3D Grid Initialization**: Creates a grid representation with attributes such as cost, obstructions, and pin locations.

- **Pathfinding**: Implements A* algorithm to find optimized routes for each net, putting in consideration:
    - Bend penalties for deviating from preferred directions.
    - Via penalties for transitioning between layers.
    - Existing routes for other already implemented nets

- **Algorithm Behavior**:
    - Takes multiple sources in the case that we already found an optimal path from one pin to the other (all cells on the path are treated as sources)
    - takes multiple targets and tries to find a path to any of them (the target that is reached is returned to inform the routing algorithm which target of them that we reached so not to treat as target anymore)
    - Initialize two 2D dictionaries (dist and paths). These dictionaries keep track of the minimum cost to reach each node and the different paths taken to get there.
    - Note that for each cell we need to store multiple paths (the parent is used as a key for the dictionary of each cell). The same goes for the “dist” dictionary
    - We begin by pushing all sources in a priority queue to always explore the least-cost path first:
    - While the priority queue is not empty:
        - you pop the node with the least cost 
        - mark it as visited
        - check if we reached any of the targets
        - if we reach the target we traceback the path and store it along with the cost in an array of potential routes
        - we explore its neighbours from all directions
        - for each valid neighbour (not an obstacle or a via or a cell already used in another net or out of the grid) we calculate the cost by taking into consideration the normal move     
    - We finish when the priority queue is empty
    - If the potential routes array is empty then we return an error that we cannot find a route
    - We check all the possible routes that we were able to construct and choose the path with the lowest cost (the normal cost, not the heuristic cost)
    - Return the path, the cost, and the target we reached
    - Routing of the net ends when all targets are reached 

- **Cost Calculation**: The total cost for each net is a combination of:
    - **Base Cost:** The inherent cost of traversing a cell.
    - **Bend Penalty:** Additional cost for deviating from the layer's preferred routing direction.
    - **Via Penalty:** Additional cost for transitioning between layers (e.g., from M0 to M1).
    - Note that we keep track of two costs (the normal cost and the heuristic cost that we use to guide the algorithm to prioritize nodes closer to targets)

- **Output**: 
    - The algorithm returns the computed path for a net, marking the routed cells as ```used_in_path``` and updating their metadata with path numbers, net numbers, and via information. Thus, this path won't be used by any other nets on the same grid.

- **Net Routing**: Sequentially routes nets in order of complexity using a net ordering heuristic.

- **Visualization**: The visualization component provides a detailed graphical representation of the routed paths and grid states. It is implemented using ```matplotlib``` and includes the following features:

    - **Layer Representation**:
        - Separate visualizations for Layer M0 (horizontal) and Layer M1 (vertical).
        - Each layer's grid is displayed as a 2D matrix.
        - A combined representation of both layers in one 2D matrix where the wires are color-coded based on each layer.

    - **Color Coding**:
        - **White**: Empty cells.
        - **Light Green**: Cells used in a path / Color of the wire in layer 1 in the combined version.
        - **Red**: Obstructions.
        - **Dark Green**: Via points, with annotated via numbers.
        - **BLUE**: Color of the wire in Layer 0 in the combined graph.

    - **Annotations**:
        - **Pins**: Marked as ```P```, with identifiers indicating their net and pin numbers (e.g., ```P01``` for Net 0, Pin 1).
        - **Vias**: Marked as ```V```, with unique via numbers (e.g., ```V1``` for the first via).
    
    - **Enhanced Features**:
        - **Dynamic Titles**: Clearly indicate the layer being visualized (e.g., "Layer M0 (Horizontal)" or "Layer M1 (Vertical)").
        - **Interactive Gridlines**: Minor gridlines separate cells for clarity.
        - **Color Bars**: Legends explaining the color codes for easy interpretation.

    - **Via and Pin Overlap Handling**:
        - Cells containing both pins and vias display both annotations with clear separation for readability.

    - **Customizable Display**:
        - The visualization scales dynamically with the grid size and displays in high resolution for large grids.

### Assumptions

There are some assumptions that were made to smooth the process of implementing and testing the code such as:
- Input is within boundries of the grid size.

### Validations

We are validating the following:
- When there is no possible route to a pin.
- When nets share a pin.
- When pins are placed on top of obstacles.

### Bonus Implementation

We implemented a net ordering heuristic that changes the order of the nets implemented based on three different factors:
- **Manhattan distance:** Larger nets (in terms of spread) are prioritized secondarily.   
- **Estimated Costs:** Based on the total number of bends and vias needed. Lower costs are prioritized
- **Number of Pins:** Nets with more pins are deprioritized (as a tie-breaker, indicated by the negative sign).

## Compilation and Execution Instructions

### Prerequisites

To run the maze router, ensure the following are installed on your system:

- **Python:** Version 3.6 or later.
- **Required Libraries:** Install using ```pip```.
```
pip install matplotlib numpy
```

### Running the Project

- **Prepare the Input File:**

Create a text file (e.g., input.txt) with the following format:

- **First line**: ```<grid_rows>,<grid_cols>,<bend_penalty>,<via_penalty>```
- **Obstructions**: ```OBS (<layer>,<x>,<y>)```
- **Nets**: ```<net_name> (<layer>,<x>,<y>) (<layer>,<x>,<y>) ...```

Example:
```
10,10,5,10
OBS (0,2,3)
OBS (1,5,7)
net1 (0,1,1) (0,8,8)
net2 (1,0,0) (1,9,9)
```

## Challenges

+ **Choosing the Right Algorithm:**
    - Deciding on the most suitable algorithm for optimal pathfinding was a key challenge. We selected dijkstra search for its ability to efficiently explore the grid while minimizing the cost. However, fine-tuning the heuristic to balance between computational efficiency and accuracy required careful consideration.

+ **Accounting for Vias and bend penalties:**
    - Incorporating the penalties for bends and vias into the pathfinding logic added complexity. Each movement had to be evaluated not just for its distance but also for its associated penalties, particularly for transitions between layers (vias) and deviations from preferred directions.

+ **Flexibility in Via Placement:**
    - Ensuring that vias could be placed at any location in the grid without overlapping or causing routing conflicts was challenging. We had to dynamically manage via numbers, update the grid's metadata, and ensure the visualization clearly highlighted vias even when combined with other elements like pins.


## Contributions

- **Algorithm:** Mohanad and Mahmoud Nour
- **Visualization:** Adham and Mohanad
- **Testing and debugging**: Adham and Noureldeen Allama
- **Documentation:** Adham and Noureldeen Allama