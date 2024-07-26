# Cura 

Download latest cura: https://ultimaker.com/software/ultimaker-cura/


1. Add x1 sidewinder (there is no x2...)
2. Change nozle to 0.6 - default profiles are great no need to tweak here anything
3. **Retraction distance** to 1mm instead of 2mm
4. Temp: 195/50 for PLA
5. In start gcode add after G28: 
```
G28 ; home all axes
M420 S1 Z10; load bed Mesh
```
