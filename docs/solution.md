# Proposed Solution

## Edge-Based Inference
I propose integrating an Inference Engine directly into the telemetry collection tier (e.g., an OpenTelemetry Collector or a dedicated edge gateway). This transforms the collector from a passive router into an intelligent filter.

### Key Components:
* The Collector (Host): The point of ingestion (sidecar or daemon-set).  
* The Model: A lightweight algorithm (e.g., Random Cut Forest, Isolation Forest, or Autoencoder) optimized for the edge. 
* The Policy Engine: Executes actions based on inference (e.g., "If anomaly score $> 0.8$, bypass sampling and retain 100% of related spans").

