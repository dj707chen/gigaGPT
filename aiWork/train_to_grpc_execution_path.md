# train.py → PyTorch Simfab (gRPC Appliance Server) Execution Path

## Question

> Find the code execution path that train.py connects to PyTorch Simfab (gRPC Appliance Server)

---

## Answer

### Layer 1 — `gigaGPT/train.py` (User entry point)

```
main()
  ├── cstorch.backend("CSX", cluster_config=cs_config)   [line 36]
  ├── cstorch.compile(model, backend)                    [line 46]
  ├── cstorch.utils.data.DataExecutor(dataloader, ...)   [line 162]
  └── for batch in executor:
        training_step(batch)    ← @cstorch.trace decorated [line 171]
```

---

### Layer 2 — `cerebras/pytorch/backend/__init__.py` (cerebras-pytorch-src)

`cstorch.backend("CSX")` calls `Backend(BackendType.CSX, cluster_config=...)`:

```python
# backend/__init__.py:89-93
if self.backend_type == BackendType.CSX:
    from .ltc_backend import PyTorchLtcBackendImpl
    self._impl = PyTorchLtcBackendImpl(self.backend_type, *args, **kwargs)
```

---

### Layer 3 — `cerebras/pytorch/backend/ltc_backend.py` (cerebras-pytorch-src)

`PyTorchLtcBackendImpl` (LTC = Lazy Tensor Core):
- Sets up a `LazyDevice` — a fake PyTorch device that records ops without executing them
- Creates an `ApplianceMode` instance (`core/appliance.py`) for CSX interaction
- On the first call to the `@cstorch.trace`-decorated `training_step()`, it traces the entire forward+backward graph and triggers compilation

---

### Layer 4 — `cerebras/pytorch/core/appliance.py` — `ApplianceMode(ApplianceManager)` (cerebras-pytorch-src)

This is the PyTorch-specific appliance orchestration layer. The training loop drives two phases:

**Compile phase** (`ApplianceMode.compile()`):
```python
# appliance.py:185-210
compile_request = CompileRequest(
    cirh_content=cirh_str,   # the traced graph (CIRH format)
    compile_dir=...,
    batch_size=...,
    num_csx=...,
)
super().compile(compile_request)   # → ApplianceManager.compile()
```

**Execute phase** (`ApplianceMode.execute()`):
```python
# appliance.py:344-380
self.execute_prep(cleanup_stack)          # allocate cluster resources + connect to CRD
self.initialize_session(run_request, ...) # load RTIR + send RunRequest to CRD
self.execute_session(initial_state_dict)  # send weights + start streaming
```

---

### Layer 5 — `cerebras/appliance/appliance_manager.py` — `ApplianceManager` (cerebras-appliance-src)

Manages the full resource lifecycle via two sub-clients:

#### 5a. Cluster Management (`ClusterManagementClient`) — resource allocation
```python
# appliance_manager.py:836, 908
with ClusterManagementClient(**self._mgmt_client_args) as mgmt_client:
    mgmt_client.init_compile_job(...)   # schedule compile pod on k8s cluster
    # → returns coordinator gRPC address + TLS certificate

mgmt_client.init_execute_job(...)       # schedule execute pods
```

#### 5b. gRPC Client creation (`ApplianceClient`) — connects to CRD/Simfab
```python
# appliance_manager.py:372-383
self._grpc_client = ApplianceClient(
    self._coord_address,         # coordinator IP:port from k8s
    credentials=self._credentials,   # TLS cert from cluster mgmt
    execution_strategy=ExecutionStrategy.ES_WEIGHT_STREAMING,
)
```

#### 5c. Compile → gRPC call
```python
# appliance_manager.py:973
compile_resp = self.grpc_client.compile(compile_request)
# → stub.UnaryCompile(CompileRequest)
```

#### 5d. Execute sequence
```python
# appliance_manager.py:634-698
self.grpc_client.init_servers(init_request)   # stub.UnaryInit
self.grpc_client.load_rtir(load_request)      # stub.UnaryLoad
self.grpc_client.run_deferred(run_request)    # stub.UnaryRun
# (ApplianceMode.send_weights)
self.grpc_client.send_check(...)              # stub.UnarySendCheck
self.grpc_client.send_weight(...)             # stub.SendInputBidirStream (streaming)
self.grpc_client.sync()                       # stub.UnarySync
self.grpc_client.start_streaming()            # stub.UnaryStartStreaming
# Per training step:
self.grpc_client.recv_output(iteration, name) # stub.GetOutputStream (streaming)
```

---

### Layer 6 — `cerebras/appliance/appliance_client.py` — `ApplianceClient` (gRPC leaf)

Opens the actual gRPC channel to the **Coordinator (CRD)** — Simfab's gRPC Appliance Server:

```python
# appliance_client.py:251-268
self._channel = grpc.secure_channel(crd_address, credentials, options=channel_options)
self._channel = grpc.intercept_channel(self._channel, ExceptionFormattingInterceptor())
self.stub = ApplianceStub(self._channel)
```

The `ApplianceStub` (generated from `appliance_service_pb2_grpc.py`) exposes all gRPC methods to the Simfab Coordinator:

| gRPC Call | Method | Purpose |
|---|---|---|
| `stub.UnaryStart` | `StartRequest` | Initialize service + version check |
| `stub.UnaryClockSync` | `ClockSyncRequest` | Verify server READY state |
| `stub.UnaryCompile` | `CompileRequest` | Compile model graph (CIRH → RTIR) |
| `stub.UnaryInit` | `InitRequest` | Initialize CMD/WGT servers |
| `stub.UnaryLoad` | `LoadRequest` | Load compiled RTIR onto hardware |
| `stub.UnaryRun` | `RunRequest` | Start training iteration |
| `stub.UnarySendCheck` | `SendCheckRequest` | Query expected weight tensor IDs |
| `stub.SendInputBidirStream` | `SendInputRequest` | Stream weight tensors (chunked) |
| `stub.UnarySendDeferredInput` | `SendDeferredInputRequest` | Send deferred/graph weights |
| `stub.UnarySync` | `SyncRequest` | Sync all servers before streaming |
| `stub.UnaryStartStreaming` | `StartStreamingRequest` | Put runtime in streaming mode |
| `stub.GetOutputStream` | `GetOutputRequest` | Receive loss/activation tensors |
| `stub.UnaryMonitorError` | `MonitorErrorRequest` | Async error monitoring thread |
| `stub.UnaryHeartBeat` | `HeartBeatRequest` | Keep-alive to CRD (every 30s) |
| `stub.UnaryFinalize` | `FinalizeRequest` | Clean shutdown |

---

### Summary diagram

```
train.py
  └─ cstorch.backend("CSX")
       └─ Backend → PyTorchLtcBackendImpl (ltc_backend.py)
            └─ LazyDevice (traces ops, builds CIRH graph)
                 └─ ApplianceMode (core/appliance.py)
                      └─ ApplianceManager (appliance_manager.py)
                           ├─ ClusterManagementClient ──► k8s cluster mgmt API
                           │    (allocates CRD/WGT/WRK/ACT pods, returns gRPC addr)
                           └─ ApplianceClient (appliance_client.py)
                                └─ grpc.Channel → ApplianceStub
                                     └─► Simfab Coordinator gRPC Server (CRD)
                                              ├─ Weight Servers (WGT)
                                              ├─ Worker Servers (WRK)
                                              ├─ Activation Servers (ACT)
                                              ├─ Command Servers (CMD)
                                              ├─ Chief (CHF)
                                              └─ Wafer Scale Engine (WSE / CS-X chip)
```

The key connection point: `ApplianceClient.__init__` in `appliance_client.py:247-268` opens the gRPC channel using the coordinator address obtained from `ClusterManagementClient` (via `stage_execute_coordinator()` in `appliance_manager.py:613-628`). From that point all interaction with Simfab is through `ApplianceStub` method calls on that channel.
