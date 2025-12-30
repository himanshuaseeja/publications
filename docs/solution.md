<link rel="stylesheet" href="assets/custom.css">

# Proposed Solution

## Edge-Based Inference
I propose integrating an Inference Engine directly into the telemetry collection tier (e.g., an OpenTelemetry Collector or a dedicated edge gateway). This transforms the collector from a passive router into an intelligent filter.

### Key Components:
* The Collector (Host): The point of ingestion (sidecar or daemon-set).  
* The Model: A lightweight algorithm (e.g., Random Cut Forest, Isolation Forest, or Autoencoder) optimized for the edge. 
* The Policy Engine: Executes actions based on inference (e.g., "If anomaly score $> 0.8$, bypass sampling and retain 100% of related spans").

## Implementation Strategies: Hosting Patterns
Architects must choose between two primary hosting patterns based on their resource constraints and complexity requirements:

| Feature  | Local(In-Process) | Remote (Sidecar/Gateway) |
| ------------- | ------------- | ------------------------ |
| Hosting | Model runs inside the collector (e.g., WASM in OTel) | Model runs in a dedicated container on the same node | 
| Performance | Near-zero network overhead; minimal latency | Higher resource ceiling; protects collector stability |
| Complexity | High (WASM/Go integration) | Low (Standard REST/gRPC interfaces) |
| Best Use Case | Simple metric thresholding and trend shifts, span analysis | Complex trace patterns and multi-metric correlation |

## Technical Deep-Dive: Metrics vs. Traces

### Anomaly Detection for Metrics
The edge model monitors "Golden Signals" (Latency, Errors, Traffic, Saturation). Rather than streaming every data point, the edge node follows a "Report by Exception" model:
* Steady State: Sends standard heartbeats and summaries (histograms).
* Anomaly State: Streams high-fidelity raw data only during detected deviations.

### The Hybrid Approach to Trace Sampling
While edge processing offers speed, it is limited to span-level analysis. To solve this, I propose a Hybrid Framework that bridges the gap between local speed and global context.

1. Structural Anomaly Sampling (Intra-Span)
Focuses on deviations within the metadata of individual spans, identifying localized resource exhaustion.
* Adaptive Z-Score: Computes a running mean and standard deviation. A trace is sampled if any span duration $D$ exceeds $D > \mu + k\sigma$.
* Attribute "Newness": Uses Bloom filters to identify unseen error codes or unique exception tags, forcing a 100% sampling rate for novel events.

2. Topological Anomaly Sampling (Inter-Span)
Detects changes in the "call graph"—how services interact.
* Causal Graph Sketching: Uses lightweight hashes of parent-child relationships (e.g., $A \rightarrow B \rightarrow C$). If a request bypasses a service ($A \rightarrow C$), it is flagged as a mutation.
* GNN Latent Representation: Encodes the trace tree into a vector. A trace is sampled if the distance between its vector and the "Normal Cluster" centroid exceeds a predefined $\epsilon$.

## Challenges and Mitigation

### The Model Drift Challenge: Edge models can become "stale" as application code changes.
- Solution: Implement a Federated Learning loop. The central backend retrains models using the "Anomaly Store" and pushes updated weights/filters back to the edge collectors.

### The Resource Contention Challenge: Observability agents must not starve the primary application.
- Solution: Leverage WebAssembly (WASM) for sandboxed, resource-constrained execution of models with strict CPU/Memory limits.


