# Cura 

Download latest cura: https://ultimaker.com/software/ultimaker-cura/


1. Add artillery>sidewinder x1 (there is no x2...) 
2. Change nozle to 0.6 - default profiles are great no need to tweak here anything
3. **Retraction distance** to 1mm instead of 2mm
4. Temp: 195/50 for PLA
5. In start gcode add after G28 (settings>printers>manage printers and machine settings)
6. Dragonhotend update + all metal extruder BMG:

Printer:\
X: 314 (+14)\
Y: 314 (+14)

X Min: -10\
Y Min: -10\
X Max: 20\
Y Max: 0


Extruder 1:\
Nozzle offset Y = 14
```
G28 ; home all axes
M420 S1 Z10; load bed Mesh
```
