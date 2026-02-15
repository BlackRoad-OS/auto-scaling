# BlackRoad Intelligent Auto-Scaling

**Smart scaling for AI workloads using KEDA, HPA, and custom metrics**

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    BlackRoad Scaling Controller              │
├─────────────┬─────────────┬─────────────┬──────────────────┤
│     HPA     │     VPA     │    KEDA     │  Custom Metrics   │
│   (Pods)    │  (Resources)│  (Events)   │   (AI-Aware)      │
└─────────────┴─────────────┴─────────────┴──────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│              AI Workload Metrics (Prometheus)                │
│  • GPU Utilization  • Inference Latency  • Queue Depth      │
│  • Memory Pressure  • Token Throughput   • Cost Efficiency   │
└─────────────────────────────────────────────────────────────┘
```

## Quick Start

```bash
# Install with Helm
helm repo add blackroad https://charts.blackroad.io
helm install auto-scaling blackroad/auto-scaling

# Or apply manifests directly
kubectl apply -f manifests/
```

## Scaling Strategies

### 1. Inference Load Scaling
```yaml
# Scale based on inference queue depth
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: blackroad-inference-scaler
spec:
  scaleTargetRef:
    name: blackroad-inference
  minReplicaCount: 1
  maxReplicaCount: 100
  triggers:
  - type: prometheus
    metadata:
      serverAddress: http://prometheus:9090
      metricName: inference_queue_depth
      threshold: "50"
      query: sum(blackroad_inference_queue_pending)
```

### 2. GPU-Aware Scaling
```yaml
# Scale based on GPU memory utilization
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: blackroad-gpu-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: blackroad-gpu-workload
  minReplicas: 1
  maxReplicas: 50
  metrics:
  - type: External
    external:
      metric:
        name: nvidia_gpu_memory_used_bytes
      target:
        type: AverageValue
        averageValue: "8Gi"
```

### 3. Cost-Optimized Scaling
```yaml
# Scale down during off-peak, scale up during demand
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: blackroad-cost-scaler
spec:
  scaleTargetRef:
    name: blackroad-agents
  minReplicaCount: 0  # Scale to zero when idle
  maxReplicaCount: 30000
  cooldownPeriod: 300
  triggers:
  - type: cron
    metadata:
      timezone: America/Chicago
      start: 0 8 * * 1-5   # 8 AM weekdays
      end: 0 18 * * 1-5    # 6 PM weekdays
      desiredReplicas: "100"
```

## Metrics Exposed

| Metric | Description | Labels |
|--------|-------------|--------|
| `blackroad_scaling_replicas` | Current replica count | deployment, namespace |
| `blackroad_scaling_target` | Target replica count | deployment, namespace |
| `blackroad_scaling_decisions` | Scaling decisions made | direction, reason |
| `blackroad_cost_per_inference` | Cost per inference | model, tier |

## Installation

```bash
# Prerequisites
kubectl create namespace blackroad-system

# Install KEDA
helm repo add kedacore https://kedacore.github.io/charts
helm install keda kedacore/keda --namespace blackroad-system

# Install BlackRoad Auto-Scaling
kubectl apply -f manifests/
```

## Configuration

See `config/` for customization options.

---

*BlackRoad OS - Scaling AI to 30,000 Agents*
