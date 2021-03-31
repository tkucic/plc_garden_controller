# PLC Garden Controller | [MAIN] | [NAMESPACES] | [METRICS] | [BACK]  

## Documentation for Function checkOverlapRectangle (BOOL)  

```pascal
INTERFACE
    VAR_INPUT
        R1_X : INT; (*Rectangle 1 x position*)
        R1_Y : INT; (*Rectangle 1 y position*)
        R1_W : INT; (*Rectangle 1 width*)
        R1_H : INT; (*Rectangle 1 height*)
        R2_X : INT; (*Rectangle 2 x position*)
        R2_Y : INT; (*Rectangle 2 y position*)
        R2_W : INT; (*Rectangle 2 width*)
        R2_H : INT; (*Rectangle 2 height*)
    END_VAR
END_INTERFACE
FUNCTION checkOverlapRectangle :
    (*Calculate if these two rectangles are overlapping*)
IF R1_X < (R2_X + R2_W) AND
   (R1_X + R1_W) > R2_X AND
   R1_Y < (R2_Y + R2_H) AND
  (R1_H + R1_Y) > R2_Y THEN
   checkOverlapRectangle := TRUE;
ELSE
	checkOverlapRectangle := FALSE;
END_IF
END_FUNCTION
```

## Metrics  

| VAR_IN | VAR_OUT | VAR_IN_OUT | VAR_TEMP | VAR_LOCAL |
| ------ | ------- | ---------- | --------- | -------- |
| 8 | 0 | 0 | 0 | 0 |  

| Lines of code | Maintainable size |
| ------------- | ----------------- |
| 9 | 25 |

---
Autogenerated with [ia_tools](https://github.com/tkucic/ia_tools)  

[MAIN]: ../../../../index_st.md
[NAMESPACES]: ../../nsList_st.md
[METRICS]: ../../../metrics_st.md
[BACK]: ../nsMain_st.md