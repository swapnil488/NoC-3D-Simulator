# NoC 3D Simulator

An interactive 3D model of a Network on Chip (NoC). Watch packets travel from core to core as trains of flits, hopping router to router, and see how the topology and routing algorithm change latency and congestion.

It covers both studies of the project:

- **2D chip:** Mesh, Diagonal Mesh and Inverted-Z Mesh.
- **3D chip:** the six topologies of the 3D scope (T1 to T6), on any grid size up to 8x8x8.

**Live demo:** https://swapnil488.github.io/noc-3d-simulator/

## What you can change

| Control | Options |
| --- | --- |
| Chip | 2D chip or 3D chip |
| Grid size | Columns, rows and layers, each set separately. 2D: 2 to 12 columns and rows. 3D: 2 to 8 columns, rows and layers. Buttons for the scope's sizes: 4x4 and 8x8, 4x4x4 and 8x8x8 |
| Topology (2D) | Mesh, Diagonal Mesh (both diagonals), Inverted-Z Mesh (one diagonal per square, top-left to bottom-right) |
| Topology (3D) | T1 to T6, see below |
| Routing algorithm | Fixed path, DyAD, Q routing |
| Traffic | Uniform Random, Transpose, Butterfly, four single flows (the scope's pair table), QoS mix of GT and BE tiles |
| Source | CBR (steady interval) or Bursty (3 packets back to back, then a pause) |
| QoS | On or off, for the QoS mix |
| Load | 5 % to 100 % (100 % = one flit every 2 cycles per tile) |
| Packet size | Payload 10 to 100 bytes (5-byte flits plus a head flit: 20 B = 5 flits) |
| Speed | 1x to 10x, with Pause and Restart |
| Gap between layers | Spreads the layers of a 3D chip apart so the links are easier to see |

Live results show average latency (cycles), average hops, flits delivered per cycle and packets held at the source. With the QoS mix they also show GT and BE latency separately.

## The six 3D topologies

| # | Topology | Links of an inner router | Fixed-path routing |
| --- | --- | --- | --- |
| T1 | 3D Mesh | 6 | XYZ |
| T2 | 3D Diagonal Mesh: T1 + both diagonals inside every layer | 10 | Diagonal-first in the layer, then Z |
| T3 | T2 + face diagonals (change layer + one step N, S, E or W) | 18 | DIAGONAL-3D |
| T4 | T2 + body diagonals (change layer + one diagonal step) | 18 | DIAGONAL-3D |
| T5 | All diagonals | 26 | DIAGONAL-3D |
| T6 | Partially vertical 3D Mesh: Up/Down links only where column + row is even ("elevators") | 6 or 4 | Elevator-first |

For the chosen topology and grid, the page shows the number of links, the links between layers, the ports per router, the longest shortest route and the average shortest route. At 4x4x4 and 8x8x8 these match the tables in the 3D scope (for example T1 at 4x4x4: 144 links, 48 between layers, 9 hops, 3.81 hops).

Tile number = `layer x rows x columns + row x columns + column`.

## What you see

- **Tiles:** each tile has a core (small block) and a router (large block), joined by a line. In a 3D chip the layers are stacked, each on a faint plate.
- **Links:** grey = North-South and East-West; blue = diagonal inside a layer; green = vertical link between layers (TSV); purple = face diagonal; pink = body diagonal.
- **Flits:** each packet is one colour. The larger sphere is the head flit; the others follow it on the same route (wormhole switching). In the QoS mix, GT packets are amber and BE packets are blue.
- **Congestion:** a router turns orange as its input buffers fill.
- **Elevators (T6):** routers with Up/Down links are tinted green.

## How the model works

- Each tile sends packets at an interval set by the load, to a destination chosen by the traffic pattern.
  - **Transpose:** 2D (row, column) to (column, row); 3D (column, row, layer) to (layer, row, column).
  - **Butterfly:** swap the first and last bits of the tile number.
  - **Single flows:** tile 0 to the last tile and back, and the other two corners (for 4x4x4: 0-63, 63-0, 3-60, 60-3).
  - **QoS mix:** tiles whose number leaves remainder 3 when divided by 8 are GT and send bursts at 30 % load; the others are BE and send CBR at the chosen load. With QoS on, GT packets are served first at every router.
- Every router input from a neighbour holds up to 4 flits. A head flit reserves an output and the tail flit releases it, so packets never interleave on a link.
- **Fixed path** follows the project's rules:
  - XYZ (T1): finish the columns, then the rows, then change layer.
  - Diagonal-first (2D and T2): take a diagonal while both the row and the column differ, then X, then Y, then change layer.
  - DIAGONAL-3D (T3 to T5): while the layer differs, take the link that changes layer and moves closest to the destination (body diagonal, then face diagonal, then straight Up/Down); then diagonal-first in the layer.
  - Elevator-first (T6): step to the router's own elevator (its West neighbour; North in column 0), ride Up or Down, then XY.
- **DyAD:** keep the usual path while the next router is quiet; when it is filling up, pick the least busy of the allowed directions.
- **Q routing:** each router keeps an estimate of the delivery time to every destination through every neighbour, picks the lowest among the allowed directions, and updates it from the next router's estimate (learning rate 0.5).
- The directions DyAD and Q routing may choose from are the same as in the project's `routing_rules3d.h`, so they cannot deadlock: first every link that changes layer toward the destination, then Odd-Even (Mesh, T1), the dominant-axis rule (Diagonal Mesh, T2 to T5) or West-first (T6) inside the layer. Inverted-Z makes all west/north-ward moves before any east/south-ward move.

This is a simplified teaching model. Its numbers show the trends but do not match a cycle-accurate simulator such as NIRGAM, and its DyAD, Q routing and QoS are simpler than the project's.

## Run it locally

It is a single HTML file. Open `index.html` in a browser, or serve the folder:

```bash
python3 -m http.server 8000
# then open http://localhost:8000
```

three.js r128 and its OrbitControls load from public CDNs, so an internet connection is needed.

## Controls

Drag to rotate, scroll or pinch to zoom.
