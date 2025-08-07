```mermaid
graph TB
    subgraph "Kubernetes v1.34.0 Architecture Overview"
        subgraph "Control Plane"
            API[kube-apiserver<br/>✨ APIServerTracing GA<br/>🔧 Async API calls<br/>📊 New metrics]
            ETCD[etcd v3.6.4<br/>🔄 Compaction improvements]
            SCHED[kube-scheduler<br/>🚀 Async API calls<br/>🎯 DRA GA<br/>📈 Performance metrics]
            CM[kube-controller-manager<br/>🔄 ResourceClaim controller<br/>📊 Enhanced metrics]
        end
        
        subgraph "Node Components"
            KUBELET[kubelet<br/>🏷️ Pod-level resources (Beta)<br/>🔧 DRA device health<br/>📝 Observed generation tracking]
            PROXY[kube-proxy<br/>🪟 WinDSR & WinOverlay GA<br/>🌐 EndpointSlice based]
            CRI[Container Runtime<br/>🔄 Service account credentials<br/>📦 Protocol buffer migration]
        end
        
        subgraph "Storage & Resources"
            CSI[CSI Drivers<br/>🚨 Attachment limit detection<br/>🔄 Volume expansion recovery GA]
            DRA_DRIVER[DRA Drivers<br/>✅ v1 API (GA)<br/>🏥 Health monitoring<br/>📊 Resource tracking]
            PV[Persistent Volumes<br/>🏷️ VolumeAttributesClass GA<br/>🔧 Dynamic attribute updates]
        end
        
        subgraph "New Features & Enhancements"
            POD_LEVEL[Pod-Level Resources<br/>📋 pod.spec.resources<br/>🎯 HPA support<br/>🔧 Topology manager integration]
            SECURITY[Enhanced Security<br/>🔐 Service account validation<br/>🛡️ AppArmor SecurityContext<br/>📝 Certificate request validation]
            OBSERVABILITY[Enhanced Observability<br/>📊 New metrics collection<br/>🔍 Tracing improvements<br/>📈 Memory tracking]
        end
    end
    
    API --> SCHED
    API --> CM
    API --> ETCD
    KUBELET --> API
    KUBELET --> CRI
    KUBELET --> DRA_DRIVER
    PROXY --> API
    CSI --> KUBELET
    DRA_DRIVER --> SCHED
    
    style API fill:#e1f5fe
    style SCHED fill:#f3e5f5
    style DRA_DRIVER fill:#e8f5e8
    style POD_LEVEL fill:#fff3e0
    style SECURITY fill:#ffebee
    style OBSERVABILITY fill:#f1f8e9
```

```mermaid
graph LR
    subgraph "DRA (Dynamic Resource Allocation) Flow"
        RC[ResourceClaim<br/>📋 User request] --> DC[DeviceClass<br/>🏷️ Device selection]
        DC --> RS[ResourceSlice<br/>📊 Available devices]
        RS --> SCHED[Scheduler<br/>🎯 Device allocation]
        SCHED --> POD[Pod<br/>🚀 Running workload]
        POD --> HEALTH[Device Health<br/>🏥 Monitoring]
    end
    
    subgraph "Pod-Level Resources Architecture"
        POD_SPEC[pod.spec.resources<br/>📋 Pod-level limits] --> CONTAINER[Container Resources<br/>➕ Additive to pod-level]
        POD_SPEC --> HPA_NEW[HPA v2<br/>📈 Pod-level aware]
        POD_SPEC --> TOPOLOGY[Topology Manager<br/>🚫 Disabled for pod-level]
    end
    
    style RC fill:#e3f2fd
    style DC fill:#e8f5e8
    style RS fill:#fff3e0
    style SCHED fill:#f3e5f5
    style POD_SPEC fill:#fff3e0
    style HPA_NEW fill:#e8f5e8
```

```mermaid
sequenceDiagram
    participant User as User/Controller
    participant API as kube-apiserver
    participant Sched as kube-scheduler
    participant Kubelet as kubelet
    participant DRA as DRA Driver
    
    Note over User,DRA: Dynamic Resource Allocation Flow (v1.34.0 GA)
    
    User->>API: Create ResourceClaim
    API->>API: Validate with v1 schema
    API-->>User: ResourceClaim created
    
    User->>API: Create Pod with ResourceClaim
    API->>Sched: Pod scheduling request
    
    Sched->>DRA: Query available devices
    DRA-->>Sched: Device capabilities
    Sched->>Sched: Allocate devices (async)
    Sched->>API: Update Pod with allocation
    
    API->>Kubelet: Pod creation
    Kubelet->>DRA: Configure device (v1 API)
    DRA-->>Kubelet: Device ready
    Kubelet->>Kubelet: Start containers
    
    loop Health Monitoring
        DRA->>Kubelet: Device health status
        Kubelet->>API: Update pod.status.containerStatuses.allocatedResourcesStatus
    end
```