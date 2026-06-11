# NTO or MachineConfig

**Node Tuning Operator (NOT)** can be used to modify kernel parameters but the standard way is to use**MachineConfig** mechanism, which one to choose?

## The General Recommendation

Use Node Tuning Operator (NTO) whenever possible. It is the recommended approach for 90% of performance tuning,`sysctl` settings, and runtime kernel parameters.

Use **MachineConfig** only when strictly necessary. Specifically, for kernel boot arguments or deep operating system (CoreOS) configurations that absolutely require a system reboot to take effect.