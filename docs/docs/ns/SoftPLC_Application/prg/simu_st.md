# PLC Garden Controller | [MAIN] | [NAMESPACES] | [METRICS] | [BACK]  

## Documentation for Program simu  

```pascal
INTERFACE
    VAR 
        clock : ClockPulse; (*Clock pulse for the program*)
        i : UINT; (*Iterator variable*)
        vG1PlantsAlive : UINT; (*Number of plants alive in garden 1*)
        vG2PlantsAlive : UINT; (*Number of plants alive in garden 2*)
        vG3PlantsAlive : UINT; (*Number of plants alive in garden 3*)
        vG4PlantsAlive : UINT; (*Number of plants alive in garden 4*)
        vGarden1 : ARRAY[0..24] OF Flower_typ; (*Locations of plant cells in the HMI for the garden 1. Not to be mistaken with servo offset*)
        vGarden2 : ARRAY[0..24] OF Flower_typ; (*Locations of plant cells in the HMI for the garden 2. Not to be mistaken with servo offset*)
        vGarden3 : ARRAY[0..24] OF Flower_typ; (*Locations of plant cells in the HMI for the garden 3. Not to be mistaken with servo offset*)
        vGarden4 : ARRAY[0..24] OF Flower_typ; (*Locations of plant cells in the HMI for the garden 4. Not to be mistaken with servo offset*)
        vGarden1Platform : Platform_typ; (*Platform 1 properties like x,y, water valve open etc*)
        vGarden2Platform : Platform_typ; (*Platform 2 properties like x,y, water valve open etc*)
        vGarden3Platform : Platform_typ; (*Platform 3 properties like x,y, water valve open etc*)
        vGarden4Platform : Platform_typ; (*Platform 4 properties like x,y, water valve open etc*)
    END_VAR
END_INTERFACE
PROGRAM simu:
    (* SIMULATOR FUNCTIONAL DESCRIPTION
    FLOWERS SIMULATION:
    - there are 25 flowers in each garden
    - each flower consumes humidity for life, this is represented as a number, this draw is constant
    - each flower, if it has no humidity, it start decaying, decaying is represented with color
    if color is red, the flower dies, if its green or yellow it is still alive
    - if the flower gets too much water (>200%) this has same effect as it being too dry so the 
    platform controller must maintain a proper level of humidity.
    - there are no humidity sensors deployed to the gIO structure but if the user wants to, it is very easy

    PLATFORM SIMULATION:
    - platforms consist out of one Y Axis servo and two X axis servos
    - X axis servos are electrically connected so they move as one for simplicity
    - each servo motor is reporting its offset from the beggining of the servo
    - for simplicity, each servo starts at 0 offset. If necessary, these starting points can be initialized
    - commands CW increase the axis while CCW decrease the axis
    - servo motors are identical but they are  flipped horizontally or verticaly, depending on the garden
    but they always follow CW and CCW commands respectively
    - upon command the open water valve releases the onboard stored water. For simplicity, the platform has 
    unlimited supply of water

    PLATFORM/FLOWER INTERACTION SIMULATION:
    - if the nozzle (not the whole platform) of the platform is above a flower cell (80x80 pixels)
    and the water valve is open, the flowers humidity increases
    - the flower cells are divided by 1 px width and the nozzle is 10px wide so it is possible to
    water multiple cells at once

    COUNTING OF ALIVE FLOWERS:
    - for each garden a counting of alive plants is done and it is displayed in the HMI*)

    clock(f:=T#20MS);

    (*Run the garden simulators*)
    SIMULATE_GARDEN1();
    SIMULATE_GARDEN2();
    SIMULATE_GARDEN3();
    SIMULATE_GARDEN4();
END_PROGRAM
ACTION SIMULATE_GARDEN1:
    (*COLOR : 16# XX XX XX XX*)
    (*            A  R  G  B *)
    vG1PlantsAlive := 0;
    FOR i:=0 TO 24 DO
    	(*Simulate the humidity of one flower cell*)
    	(*If not being watered, it slowly looses humidity*)
    	vGarden1[i].hum := MAX(0, vGarden1[i].hum - 0.07 * BOOL_TO_REAL(clock.pulse));
    	
    	(*If the watering platform is above the cell, water valve is open and tank has water, increase the humidity*)
    	IF checkOverlapRectangle(R1_X:=vGarden1[i].x, R1_Y:=vGarden1[i].y, R1_W:=80, R1_H:=80,
    							R2_X:=406 + vGarden1Platform.x + 45, R2_Y:=361 + vGarden1Platform.y + 45, R2_W:=10, R2_H:=10) AND vGarden1Platform.OpenValveCmd THEN
    		vGarden1[i].hum := vGarden1[i].hum + 4 * BOOL_TO_REAL(clock.pulse);
    	END_IF
    	
    	(*Simulate flowers response to the humidity of the cell*)
    	(*Flag if the plant is alive*)
    	vGarden1[i].alive := NOT (vGarden1[i].red >= 254 AND vGarden1[i].green < 100);
    	IF vGarden1[i].hum <= 0 OR vGarden1[i].hum >= 200 THEN
    		(*Flower starts decaying if not enough or too much humiditiy*)
    		(*Change color towards red*)
    		vGarden1[i].red := vGarden1[i].red + 1 * BOOL_TO_BYTE(clock.pulse);
    		vGarden1[i].green := vGarden1[i].green - 1 * BOOL_TO_BYTE(clock.pulse AND vGarden1[i].red > 253);
    	ELSE
    		IF vGarden1[i].alive THEN
    			(*Flower gets better much faster than it dies*)
    			(*Change color towards green*)
    			vGarden1[i].green := vGarden1[i].green + 1 * BOOL_TO_BYTE(clock.pulse);
    			vGarden1[i].red := vGarden1[i].red - 1 * BOOL_TO_BYTE(clock.pulse AND vGarden1[i].green > 253);
    		(*ELSE It is dead for good*)
    			
    		END_IF
    	END_IF
    	vGarden1[i].red := LIMIT(1, vGarden1[i].red, 254);
    	vGarden1[i].green := LIMIT(1, vGarden1[i].green, 254);
    	
    	(*Combine the channel bytes into dword color*)
    	vGarden1[i].color := BYTES_TO_DWORD(B0 := vGarden1[i].blue,B1 := vGarden1[i].green,
    										B2 := vGarden1[i].red, B3 := vGarden1[i].alpha);
    										
    	(*Count how many plants are alive*)
    	vG1PlantsAlive := vG1PlantsAlive + BOOL_TO_UINT(vGarden1[i].alive);
    END_FOR

    (*Servo motors*)
    (*Move on X*)
    IF gIO.Garden1_ServoX_CCW THEN
    	vGarden1Platform.x := vGarden1Platform.x + 1;
    ELSIF gIO.Garden1_ServoX_CW THEN
    	vGarden1Platform.x := vGarden1Platform.x - 1;
    END_IF
    (*Limit the platforms movement on x axis*)
    vGarden1Platform.x := LIMIT(-404, vGarden1Platform.x, 0);
    (*Move on Y*)
    IF gIO.Garden1_ServoY_CCW THEN
    	vGarden1Platform.y := vGarden1Platform.y + 1;
    ELSIF gIO.Garden1_ServoY_CW THEN
    	vGarden1Platform.y := vGarden1Platform.y - 1;
    END_IF
    (*Limit the platforms movement on y axis*)
    vGarden1Platform.y := LIMIT(-320, vGarden1Platform.y, 0);

    (*Report the position of the platform*)
    gIO.Garden1_ServoX_Offset := ABS(vGarden1Platform.x);
    gIO.Garden1_ServoY_Offset := ABS(vGarden1Platform.y);

    (*Control the water valve*)
    vGarden1Platform.OpenValveCmd := gIO.Garden1_WaterTankOpenCmd;
END_ACTION
ACTION SIMULATE_GARDEN2:
    (*COLOR : 16# XX XX XX XX*)
    (*            A  R  G  B *)
    vG2PlantsAlive := 0;
    FOR i:=0 TO 24 DO
    	(*Simulate the humidity of one flower cell*)
    	(*If not being watered, it slowly looses humidity*)
    	vGarden2[i].hum := MAX(0, vGarden2[i].hum - 0.07 * BOOL_TO_REAL(clock.pulse));
    	
    	(*If the watering platform is above the cell, water valve is open and tank has water, increase the humidity*)
    	IF checkOverlapRectangle(R1_X:=vGarden2[i].x, R1_Y:=vGarden2[i].y, R1_W:=80, R1_H:=80,
    							R2_X:=594 + vGarden2Platform.x + 45, R2_Y:=361 + vGarden2Platform.y + 45, R2_W:=10, R2_H:=10) AND vGarden2Platform.OpenValveCmd THEN
    		vGarden2[i].hum := vGarden2[i].hum + 4 * BOOL_TO_REAL(clock.pulse);
    	END_IF
    	
    	(*Simulate flowers response to the humidity of the cell*)
    	(*Flag if the plant is alive*)
    	vGarden2[i].alive := NOT (vGarden2[i].red >= 254 AND vGarden2[i].green < 100);
    	IF vGarden2[i].hum <= 0 OR vGarden2[i].hum >= 200 THEN
    		(*Flower starts decaying if not enough or too much humiditiy*)
    		(*Change color towards red*)
    		vGarden2[i].red := vGarden2[i].red + 1 * BOOL_TO_BYTE(clock.pulse);
    		vGarden2[i].green := vGarden2[i].green - 1 * BOOL_TO_BYTE(clock.pulse AND vGarden2[i].red > 253);
    	ELSE
    		IF vGarden2[i].alive THEN
    			(*Flower gets better much faster than it dies*)
    			(*Change color towards green*)
    			vGarden2[i].green := vGarden2[i].green + 1 * BOOL_TO_BYTE(clock.pulse);
    			vGarden2[i].red := vGarden2[i].red - 1 * BOOL_TO_BYTE(clock.pulse AND vGarden2[i].green > 253);
    		(*ELSE It is dead for good*)
    			
    		END_IF
    	END_IF
    	vGarden2[i].red := LIMIT(1, vGarden2[i].red, 254);
    	vGarden2[i].green := LIMIT(1, vGarden2[i].green, 254);
    	
    	(*Combine the channel bytes into dword color*)
    	vGarden2[i].color := BYTES_TO_DWORD(B0 := vGarden2[i].blue,B1 := vGarden2[i].green,
    										B2 := vGarden2[i].red, B3 := vGarden2[i].alpha);
    										
    	(*Count how many plants are alive*)
    	vG2PlantsAlive := vG2PlantsAlive + BOOL_TO_UINT(vGarden2[i].alive);
    END_FOR

    (*Servo motors*)
    (*Move on X*)
    IF gIO.Garden2_ServoX_CCW THEN
    	vGarden2Platform.x := vGarden2Platform.x - 1;
    ELSIF gIO.Garden2_ServoX_CW THEN
    	vGarden2Platform.x := vGarden2Platform.x + 1;
    END_IF
    (*Limit the platforms movement on x axis*)
    vGarden2Platform.x := LIMIT(0, vGarden2Platform.x, 404);
    (*Move on Y*)
    IF gIO.Garden2_ServoY_CCW THEN
    	vGarden2Platform.y := vGarden2Platform.y + 1;
    ELSIF gIO.Garden2_ServoY_CW THEN
    	vGarden2Platform.y := vGarden2Platform.y - 1;
    END_IF
    (*Limit the platforms movement on y axis*)
    vGarden2Platform.y := LIMIT(-320, vGarden2Platform.y, 0);

    (*Report the position of the platform*)
    gIO.Garden2_ServoX_Offset := ABS(vGarden2Platform.x);
    gIO.Garden2_ServoY_Offset := ABS(vGarden2Platform.y);

    (*Control the water valve*)
    vGarden2Platform.OpenValveCmd := gIO.Garden2_WaterTankOpenCmd;
END_ACTION
ACTION SIMULATE_GARDEN3:
    (*COLOR : 16# XX XX XX XX*)
    (*            A  R  G  B *)
    vG3PlantsAlive := 0;
    FOR i:=0 TO 24 DO
    	(*Simulate the humidity of one flower cell*)
    	(*If not being watered, it slowly looses humidity*)
    	vGarden3[i].hum := MAX(0, vGarden3[i].hum - 0.07 * BOOL_TO_REAL(clock.pulse));
    	
    	(*If the watering platform is above the cell, water valve is open and tank has water, increase the humidity*)
    	IF checkOverlapRectangle(R1_X:=vGarden3[i].x, R1_Y:=vGarden3[i].y, R1_W:=80, R1_H:=80,
    							R2_X:=401 + vGarden3Platform.x + 45, R2_Y:=640 + vGarden3Platform.y + 45, R2_W:=10, R2_H:=10) AND vGarden3Platform.OpenValveCmd THEN
    		vGarden3[i].hum := vGarden3[i].hum + 4 * BOOL_TO_REAL(clock.pulse);
    	END_IF
    	
    	(*Simulate flowers response to the humidity of the cell*)
    	(*Flag if the plant is alive*)
    	vGarden3[i].alive := NOT (vGarden3[i].red >= 254 AND vGarden3[i].green < 100);
    	IF vGarden3[i].hum <= 0 OR vGarden3[i].hum >= 200 THEN
    		(*Flower starts decaying if not enough or too much humiditiy*)
    		(*Change color towards red*)
    		vGarden3[i].red := vGarden3[i].red + 1 * BOOL_TO_BYTE(clock.pulse);
    		vGarden3[i].green := vGarden3[i].green - 1 * BOOL_TO_BYTE(clock.pulse AND vGarden3[i].red > 253);
    	ELSE
    		IF vGarden3[i].alive THEN
    			(*Flower gets better much faster than it dies*)
    			(*Change color towards green*)
    			vGarden3[i].green := vGarden3[i].green + 1 * BOOL_TO_BYTE(clock.pulse);
    			vGarden3[i].red := vGarden3[i].red - 1 * BOOL_TO_BYTE(clock.pulse AND vGarden3[i].green > 253);
    		(*ELSE It is dead for good*)
    			
    		END_IF
    	END_IF
    	vGarden3[i].red := LIMIT(1, vGarden3[i].red, 254);
    	vGarden3[i].green := LIMIT(1, vGarden3[i].green, 254);
    	
    	(*Combine the channel bytes into dword color*)
    	vGarden3[i].color := BYTES_TO_DWORD(B0 := vGarden3[i].blue,B1 := vGarden3[i].green,
    										B2 := vGarden3[i].red, B3 := vGarden3[i].alpha);
    										
    	(*Count how many plants are alive*)
    	vG3PlantsAlive := vG3PlantsAlive + BOOL_TO_UINT(vGarden3[i].alive);
    END_FOR

    (*Servo motors*)
    (*Move on X*)
    IF gIO.Garden3_ServoX_CCW THEN
    	vGarden3Platform.x := vGarden3Platform.x + 1;
    ELSIF gIO.Garden3_ServoX_CW THEN
    	vGarden3Platform.x := vGarden3Platform.x - 1;
    END_IF
    (*Limit the platforms movement on x axis*)
    vGarden3Platform.x := LIMIT(-404, vGarden3Platform.x, 0);
    (*Move on Y*)
    IF gIO.Garden3_ServoY_CCW THEN
    	vGarden3Platform.y := vGarden3Platform.y - 1;
    ELSIF gIO.Garden3_ServoY_CW THEN
    	vGarden3Platform.y := vGarden3Platform.y + 1;
    END_IF
    (*Limit the platforms movement on y axis*)
    vGarden3Platform.y := LIMIT(0, vGarden3Platform.y, 320);

    (*Report the position of the platform*)
    gIO.Garden3_ServoX_Offset := ABS(vGarden3Platform.x);
    gIO.Garden3_ServoY_Offset := ABS(vGarden3Platform.y);

    (*Control the water valve*)
    vGarden3Platform.OpenValveCmd := gIO.Garden3_WaterTankOpenCmd;
END_ACTION
ACTION SIMULATE_GARDEN4:
    (*COLOR : 16# XX XX XX XX*)
    (*            A  R  G  B *)
    vG4PlantsAlive := 0;
    FOR i:=0 TO 24 DO
    	(*Simulate the humidity of one flower cell*)
    	(*If not being watered, it slowly looses humidity*)
    	vGarden4[i].hum := MAX(0, vGarden4[i].hum - 0.07 * BOOL_TO_REAL(clock.pulse));
    	
    	(*If the watering platform is above the cell, water valve is open and tank has water, increase the humidity*)
    	IF checkOverlapRectangle(R1_X:=vGarden4[i].x, R1_Y:=vGarden4[i].y, R1_W:=80, R1_H:=80,
    							R2_X:=600 + vGarden4Platform.x + 45, R2_Y:=642 + vGarden4Platform.y + 45, R2_W:=10, R2_H:=10) AND vGarden4Platform.OpenValveCmd THEN
    		vGarden4[i].hum := vGarden4[i].hum + 4 * BOOL_TO_REAL(clock.pulse);
    	END_IF
    	
    	(*Simulate flowers response to the humidity of the cell*)
    	(*Flag if the plant is alive*)
    	vGarden4[i].alive := NOT (vGarden4[i].red >= 254 AND vGarden4[i].green < 100);
    	IF vGarden4[i].hum <= 0 OR vGarden4[i].hum >= 200 THEN
    		(*Flower starts decaying if not enough or too much humiditiy*)
    		(*Change color towards red*)
    		vGarden4[i].red := vGarden4[i].red + 1 * BOOL_TO_BYTE(clock.pulse);
    		vGarden4[i].green := vGarden4[i].green - 1 * BOOL_TO_BYTE(clock.pulse AND vGarden4[i].red > 253);
    	ELSE
    		IF vGarden4[i].alive THEN
    			(*Flower gets better much faster than it dies*)
    			(*Change color towards green*)
    			vGarden4[i].green := vGarden4[i].green + 1 * BOOL_TO_BYTE(clock.pulse);
    			vGarden4[i].red := vGarden4[i].red - 1 * BOOL_TO_BYTE(clock.pulse AND vGarden4[i].green > 253);
    		(*ELSE It is dead for good*)
    			
    		END_IF
    	END_IF
    	vGarden4[i].red := LIMIT(1, vGarden4[i].red, 254);
    	vGarden4[i].green := LIMIT(1, vGarden4[i].green, 254);
    	
    	(*Combine the channel bytes into dword color*)
    	vGarden4[i].color := BYTES_TO_DWORD(B0 := vGarden4[i].blue,B1 := vGarden4[i].green,
    										B2 := vGarden4[i].red, B3 := vGarden4[i].alpha);
    										
    	(*Count how many plants are alive*)
    	vG4PlantsAlive := vG4PlantsAlive + BOOL_TO_UINT(vGarden4[i].alive);
    END_FOR

    (*Servo motors*)
    (*Move on X*)
    IF gIO.Garden4_ServoX_CCW THEN
    	vGarden4Platform.x := vGarden4Platform.x - 1;
    ELSIF gIO.Garden4_ServoX_CW THEN
    	vGarden4Platform.x := vGarden4Platform.x + 1;
    END_IF
    (*Limit the platforms movement on x axis*)
    vGarden4Platform.x := LIMIT(0, vGarden4Platform.x, 404);
    (*Move on Y*)
    IF gIO.Garden4_ServoY_CCW THEN
    	vGarden4Platform.y := vGarden4Platform.y - 1;
    ELSIF gIO.Garden4_ServoY_CW THEN
    	vGarden4Platform.y := vGarden4Platform.y + 1;
    END_IF
    (*Limit the platforms movement on y axis*)
    vGarden4Platform.y := LIMIT(0, vGarden4Platform.y, 320);

    (*Report the position of the platform*)
    gIO.Garden4_ServoX_Offset := ABS(vGarden4Platform.x);
    gIO.Garden4_ServoY_Offset := ABS(vGarden4Platform.y);

    (*Control the water valve*)
    vGarden4Platform.OpenValveCmd := gIO.Garden4_WaterTankOpenCmd;
END_ACTION
```

## Metrics  

- VAR : 14

| Actions | Lines of code | Maintainable size |
| ------- | ------------- | ----------------- |
| 4 | 305 | 319 |

---
Autogenerated with [ia_tools](https://github.com/tkucic/ia_tools)  

[MAIN]: ../../../../index_st.md
[NAMESPACES]: ../../nsList_st.md
[METRICS]: ../../../metrics_st.md
[BACK]: ../nsMain_st.md
