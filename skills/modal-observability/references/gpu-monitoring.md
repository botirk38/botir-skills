# GPU Monitoring — Reference

## Dashboard Metrics

### GPU Utilization %

Percentage of time the GPU was executing at least one CUDA kernel. Same metric as `nvidia-smi`.

**Limitations**: Only indicates kernel occupancy, not compute efficiency. A GPU can show 100% utilization while using a fraction of its CUDA cores or memory bandwidth.

### GPU Power Utilization %

Percentage of max power draw. Best proxy for actual work being done — a fully-saturated GPU draws near its entire power budget.

**Why it matters**: GPUs are fundamentally power-limited. Power draw directly correlates with compute throughput and memory bandwidth usage.

### GPU Temperature

Die temperature in Celsius. At high temperatures (mid-70s°C for H100), increased error correction from thermal noise can reduce performance. Generally, power utilization is a better performance proxy.

### GPU Memory Used

Allocated GPU memory in bytes. Useful for detecting memory leaks or sizing models.

## GPU Health

Modal monitors GPU hardware automatically:

- ECC (Error Correcting Code) memory error detection
- Hardware fault detection
- Automatic retry on healthy GPUs when faults detected
- No user configuration needed

## Profiling

For deep performance analysis:

1. Use PyTorch Profiler with `torch.profiler.profile()`
2. Export traces for Chrome `chrome://tracing` or TensorBoard
3. Profile on Modal with `modal run` — see Modal examples for guide

## Recommendations

- Use **power draw** as primary performance indicator, not GPU utilization %
- Monitor **GPU memory** to detect OOM risks before they crash containers
- Use profiling tools for compute efficiency optimization
- GPU metrics are aggregate signals — they cannot directly debug performance issues
