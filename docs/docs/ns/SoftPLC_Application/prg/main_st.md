# PLC Garden Controller | [MAIN] | [NAMESPACES] | [METRICS] | [BACK]  

## Documentation for Program main  

```pascal
INTERFACE
    VAR
        vG1Platform_Instructions : ARRAY[0..24, 0..1] OF INT := ['365', '320', '285', '320', '205', '320', '125', '320', '45', '320', '45', '240', '125', '240', '205', '240', '285', '240', '365', '240', '365', '160', '285', '160', '205', '160', '125', '160', '45', '160', '45', '80', '125', '80', '205', '80', '285', '80', '365', '80', '365', '0', '285', '0', '205', '0', '125', '0', '45', '0']; (*Instructions for platform servos in garden 1*)
        vG2Platform_Instructions : ARRAY[0..24, 0..1] OF INT := ['50', '320', '50', '240', '50', '160', '50', '80', '50', '0', '130', '0', '130', '80', '130', '160', '130', '240', '130', '320', '210', '320', '210', '240', '210', '160', '210', '80', '210', '0', '290', '0', '290', '80', '290', '160', '290', '240', '290', '320', '370', '320', '370', '240', '370', '160', '370', '80', '370', '0']; (*Instructions for platform servos in garden 2*)
        vG3Platform_Instructions : ARRAY[0..24, 0..1] OF INT := ['358', '0', '278', '0', '358', '80', '358', '160', '278', '80', '198', '0', '118', '0', '198', '80', '278', '160', '358', '240', '358', '320', '278', '240', '198', '160', '118', '80', '38', '0', '38', '80', '118', '160', '198', '240', '278', '320', '198', '320', '118', '240', '38', '160', '38', '240', '118', '320', '38', '320']; (*Instructions for platform servos in garden 3*)
        vG4Platform_Instructions : ARRAY[0..24, 0..1] OF INT := ['200', '160', '120', '160', '120', '240', '200', '240', '280', '240', '280', '160', '280', '80', '200', '80', '120', '80', '40', '80', '40', '160', '40', '240', '40', '320', '120', '320', '200', '320', '280', '320', '360', '320', '360', '240', '360', '160', '360', '80', '360', '0', '280', '0', '200', '0', '120', '0', '40', '0']; (*Instructions for platform servos in garden 4*)
        garden1AutoWater : AutomaticFlowerWatering := Struct initialization unsupported; (*Automatic watering logic: Left to right in a smart way*)
        garden2AutoWater : AutomaticFlowerWatering := Struct initialization unsupported; (*Automatic watering logic: Going top to bottom in a smart way*)
        garden3AutoWater : AutomaticFlowerWatering := Struct initialization unsupported; (*Automatic watering logic: Going side ways from the top to bottom*)
        garden4AutoWater : AutomaticFlowerWatering := Struct initialization unsupported; (*Automatic watering logic: Goign from the middle out*)
    END_VAR
END_INTERFACE
PROGRAM main:
    (*FUNCTIONAL DESCRIPTION:
- there are four gardens holding 25 flowers each
- the flowers need to be watered on a regular basis in order to keep their humidity in correct levels
- correct humidity is 0 > x < 200 %, if the humidity is not in those limits, the flower starts dying
- the watering is done by means of a watering platform, moved by x a y servos
- the job of this program is to move the platform of each garden and water the plants in an optimized way
- program should strive to keep all plants alive (easier said than done)
- the platforms move quite slowly as the water tank might spill so it is a given that 25 flower can be 
watered in 25 seconds
- on top of each platform there is a water tank with a valve that can be open remotely by the main program
- the program also needs to implement manual control from the HMI with the prepared variables in gIO structure
*)
CONTROL_GARDEN1();
CONTROL_GARDEN2();
CONTROL_GARDEN3();
CONTROL_GARDEN4();

END_PROGRAM
ACTION CONTROL_GARDEN1:
    IF gIO.Garden1_HMI.CtrlOn THEN
	(*HMI Control*)
	gIO.Garden1_ServoX_CW := gIO.Garden1_HMI.LeftCmd OR gIO.Garden1_HMI.LeftDownCmd OR gIO.Garden1_HMI.LeftUpCmd;
	gIO.Garden1_ServoX_CCW := gIO.Garden1_HMI.RightCmd OR gIO.Garden1_HMI.RightDownCmd OR gIO.Garden1_HMI.RightUpCmd;
	
	gIO.Garden1_ServoY_CW := gIO.Garden1_HMI.UpCmd OR gIO.Garden1_HMI.LeftUpCmd OR gIO.Garden1_HMI.RightUpCmd;
	gIO.Garden1_ServoY_CCW := gIO.Garden1_HMI.DownCmd OR gIO.Garden1_HMI.LeftDownCmd OR gIO.Garden1_HMI.RightDownCmd;
	
	gIO.Garden1_WaterTankOpenCmd := gIO.Garden1_HMI.OpenWaterValve;
ELSE
	(*Automatic control*)
	garden1AutoWater(instructions:=vG1Platform_Instructions,
		xAxis := gIO.Garden1_ServoX_Offset,
		yAxis := gIO.Garden1_ServoY_Offset,
		servoX_CW => gIO.Garden1_ServoX_CW,
		servoX_CCW => gIO.Garden1_ServoX_CCW,
		servoY_CW => gIO.Garden1_ServoY_CW,
		servoY_CCW => gIO.Garden1_ServoY_CCW,
		waterTankOpenCmd => gIO.Garden1_WaterTankOpenCmd);
END_IF
END_ACTION
ACTION CONTROL_GARDEN2:
    IF gIO.Garden2_HMI.CtrlOn THEN
	(*HMI Control*)
	(*X servos are flipped on this garden*)
	gIO.Garden2_ServoX_CCW := gIO.Garden2_HMI.LeftCmd OR gIO.Garden2_HMI.LeftDownCmd OR gIO.Garden2_HMI.LeftUpCmd;
	gIO.Garden2_ServoX_CW := gIO.Garden2_HMI.RightCmd OR gIO.Garden2_HMI.RightDownCmd OR gIO.Garden2_HMI.RightUpCmd;
	
	gIO.Garden2_ServoY_CW := gIO.Garden2_HMI.UpCmd OR gIO.Garden2_HMI.LeftUpCmd OR gIO.Garden2_HMI.RightUpCmd;
	gIO.Garden2_ServoY_CCW := gIO.Garden2_HMI.DownCmd OR gIO.Garden2_HMI.LeftDownCmd OR gIO.Garden2_HMI.RightDownCmd;
	
	gIO.Garden2_WaterTankOpenCmd := gIO.Garden2_HMI.OpenWaterValve;
ELSE
	(*Automatic control*)
	garden2AutoWater(instructions:=vG2Platform_Instructions,
		xAxis := gIO.Garden2_ServoX_Offset,
		yAxis := gIO.Garden2_ServoY_Offset,
		servoX_CW => gIO.Garden2_ServoX_CW,
		servoX_CCW => gIO.Garden2_ServoX_CCW,
		servoY_CW => gIO.Garden2_ServoY_CW,
		servoY_CCW => gIO.Garden2_ServoY_CCW,
		waterTankOpenCmd => gIO.Garden2_WaterTankOpenCmd);
END_IF
END_ACTION
ACTION CONTROL_GARDEN3:
    IF gIO.Garden3_HMI.CtrlOn THEN
	(*HMI Control*)
	gIO.Garden3_ServoX_CW := gIO.Garden3_HMI.LeftCmd OR gIO.Garden3_HMI.LeftDownCmd OR gIO.Garden3_HMI.LeftUpCmd;
	gIO.Garden3_ServoX_CCW := gIO.Garden3_HMI.RightCmd OR gIO.Garden3_HMI.RightDownCmd OR gIO.Garden3_HMI.RightUpCmd;
	
	gIO.Garden3_ServoY_CCW := gIO.Garden3_HMI.UpCmd OR gIO.Garden3_HMI.LeftUpCmd OR gIO.Garden3_HMI.RightUpCmd;
	gIO.Garden3_ServoY_CW := gIO.Garden3_HMI.DownCmd OR gIO.Garden3_HMI.LeftDownCmd OR gIO.Garden3_HMI.RightDownCmd;
	
	gIO.Garden3_WaterTankOpenCmd := gIO.Garden3_HMI.OpenWaterValve;
ELSE
	(*Automatic control*)
	garden3AutoWater(instructions:=vG3Platform_Instructions,
		xAxis := gIO.Garden3_ServoX_Offset,
		yAxis := gIO.Garden3_ServoY_Offset,
		servoX_CW => gIO.Garden3_ServoX_CW,
		servoX_CCW => gIO.Garden3_ServoX_CCW,
		servoY_CW => gIO.Garden3_ServoY_CW,
		servoY_CCW => gIO.Garden3_ServoY_CCW,
		waterTankOpenCmd => gIO.Garden3_WaterTankOpenCmd);
END_IF
END_ACTION
ACTION CONTROL_GARDEN4:
    IF gIO.Garden4_HMI.CtrlOn THEN
	(*HMI Control*)
	gIO.Garden4_ServoX_CCW := gIO.Garden4_HMI.LeftCmd OR gIO.Garden4_HMI.LeftDownCmd OR gIO.Garden4_HMI.LeftUpCmd;
	gIO.Garden4_ServoX_CW := gIO.Garden4_HMI.RightCmd OR gIO.Garden4_HMI.RightDownCmd OR gIO.Garden4_HMI.RightUpCmd;
	
	gIO.Garden4_ServoY_CCW := gIO.Garden4_HMI.UpCmd OR gIO.Garden4_HMI.LeftUpCmd OR gIO.Garden4_HMI.RightUpCmd;
	gIO.Garden4_ServoY_CW := gIO.Garden4_HMI.DownCmd OR gIO.Garden4_HMI.LeftDownCmd OR gIO.Garden4_HMI.RightDownCmd;
	
	gIO.Garden4_WaterTankOpenCmd := gIO.Garden4_HMI.OpenWaterValve;
ELSE
	(*Automatic control*)
	garden4AutoWater(instructions:=vG4Platform_Instructions,
		xAxis := gIO.Garden4_ServoX_Offset,
		yAxis := gIO.Garden4_ServoY_Offset,
		servoX_CW => gIO.Garden4_ServoX_CW,
		servoX_CCW => gIO.Garden4_ServoX_CCW,
		servoY_CW => gIO.Garden4_ServoY_CW,
		servoY_CCW => gIO.Garden4_ServoY_CCW,
		waterTankOpenCmd => gIO.Garden4_WaterTankOpenCmd);
END_IF
END_ACTION
```

## Metrics  

| VAR_IN | VAR_OUT | VAR_IN_OUT | VAR_LOCAL | VAR_EXTERNAL | VAR_GLOBAL | VAR_ACCESS | VAR_TEMP | Actions | Lines of code | Maintainable size |
| ------ | ------- | ---------- | --------- | ------------ | ---------- | ---------- | -------- | ------- | ------------- | ----------------- |
| 0 | 0 | 0 | 8 | 0 | 0 | 0 | 0 | 4 | 97 | 105 |  

---
Autogenerated with [ia_tools](https://github.com/tkucic/ia_tools)  

[MAIN]: ../../../../index_st.md
[NAMESPACES]: ../../nsList_st.md
[METRICS]: ../../../metrics_st.md
[BACK]: ../nsMain_st.md
