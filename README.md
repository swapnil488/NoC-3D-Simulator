# NoC 3D Simulator

An interactive 3D model of an 8x8 Network on Chip (NoC). Watch packets travel from core to core as trains of flits, hopping router to router, and see how the topology and routing algorithm change latency and congestion.

**Live demo:** https://swapnil488.github.io/noc-3d-simulator/

## What you can change

| Control | Options |
| --- | --- |
| Topology | Mesh, Diagonal Mesh (both diagonals), Inverted-Z Mesh (one diagonal per square, top-left to bottom-right) |
| Routing algorithm | Fixed path (XY on Mesh, diagonal-first on the diagonal topologies), DyAD, Q routing |
| Traffic | Uniform Random, Transpose, Butterfly, one flow from tile 0 to 63, one flow from tile 7 to 56 |
| Load | 5 % to 100 % (100 % = one flit every 2 cycles per tile) |
| Speed | 1x to 10x, with Pause and Restart |

Live results show average latency (cycles), average hops, flits delivered per cycle and packets held at the source.

## What you see

- **Tiles:** each of the 64 tiles has a core (top block) and a router (bottom block), joined by a vertical line.
- **Links:** grey lines are North-South and East-West links; blue lines are diagonal links.
- **Flits:** each packet is 5 flits in one colour. The larger sphere is the head flit; the others follow it on the same route (wormhole switching).
- **Congestion:** a router turns orange as its input buffers fill.

## How the model works

- Each cycle, a tile starts a new packet with a probability set by the load, to a destination chosen by the traffic pattern.
- Every router input from a neighbour holds up to 4 flits. A head flit reserves an output and the tail flit releases it, so packets never interleave on a link.
- **Fixed path:** take a diagonal while both the row and the column differ and that diagonal exists, then finish with XY.
- **DyAD:** keep the fixed path while the next router is quiet; when it is filling up, pick the least busy of the minimal directions.
- **Q routing:** each router keeps an estimate of the delivery time to every destination through every neighbour, picks the lowest, and updates it from the next router's estimate (learning rate 0.5).
- Adaptive routing only uses minimal directions and makes all west/north-ward moves before any east/south-ward move, so it cannot deadlock.

This is a simplified teaching model. Its numbers show the trends but do not match a cycle-accurate simulator such as NIRGAM.

## Run it locally

It is a single HTML file. Open `index.html` in a browser, or serve the folder:

```bash
python3 -m http.server 8000
# then open http://localhost:8000
```

three.js r128 and its OrbitControls load from public CDNs, so an internet connection is needed.

## Controls

Drag to rotate, scroll or pinch to zoom.
