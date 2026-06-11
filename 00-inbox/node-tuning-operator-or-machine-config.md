# NTO or MachineConfig

**Node Tuning Operator (NOT)** can be used to modify kernel parameters but the standard way is to use**MachineConfig** mechanism, which one to choose?

## The General Recommendation

Use Node Tuning Operator (NTO) whenever possible. It is the recommended approach for 90% of performance tuning,`sysctl` settings, and runtime kernel parameters.

Use **MachineConfig** only when strictly necessary. Specifically, for kernel boot arguments or deep operating system (CoreOS) configurations that absolutely require a system reboot to take effect.

### 1. Node Tuning Operator (NTO) — _The Dynamic & Safe Approach_

The NTO utilizes the native Red Hat Enterprise Linux `tuned` daemon. It is the ideal choice for optimizing the OS based on the specific type of workload running on your nodes.

- **What to use it for:** `sysctl` tweaks (`net.ipv4.*`, `vm.max_map_count`), power/CPU profiles, network latency optimizations, and hugepages allocation.
    
- **Advantages:**
    
    - **No reboots:** Most changes are applied "in hot" (at runtime) without disrupting your running pods.
        
    - **Flexibility:** You can assign `Tuned` profiles dynamically using node labels or inherit from predefined Red Hat profiles (like `openshift-node`, `throughput-performance`, etc.).
        
    - **Reversibility:** If you delete the `Tuned` custom resource, the node automatically reverts to its previous state.
        

### 2. MachineConfig — _The "Immutable" Low-Level Approach_

MachineConfig defines the desired state of Red Hat Enterprise Linux CoreOS (RHCOS). Instead of modifying the system at runtime, it reconfigures the OS from the root layer.

- **What to use it for:** Kernel boot arguments (`kernelArguments`), enabling FIPS mode, configuring encrypted root storage, injecting SSH keys, or OS-level certificates.
    
- **Advantages:** It guarantees total cluster immutability. If a node fails and is replaced, the new node spins up with the exact same configuration from second zero.
    
- **Disadvantage:** It **always** triggers a rolling update, meaning the Machine Config Operator will drain and reboot your nodes one by one to apply the changes.