# must-gather

## Volume usage makes must-gather to kill itself
```bash title="Image used"
; from a fresh crc 4.21.14 cluster
%> oc adm must-gather
...
[must-gather-fn8t6] POD 2026-05-26T14:02:00.367761778Z [disk usage checker] Started
[must-gather-fn8t6] POD 2026-05-26T14:02:00.404640920Z [disk usage checker] Volume usage percentage: current = 80 ; allowed = 70
[must-gather-fn8t6] POD 2026-05-26T14:02:00.404640920Z [disk usage checker] Disk usage exceeds the volume percentage of 70 for mounted directory, terminating...
[must-gather-fn8t6] POD 2026-05-26T14:02:00.434989085Z /bin/bash: line 33:     3 Killed                  setsid -w bash <<-MUSTGATHER_EOF
...

; try raising the allowed upper limit
%> oc adm must-gather --volume-percentage=90

```