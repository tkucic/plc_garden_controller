# PLC Garden Controller | [MAIN] | [NAMESPACES] | [METRICS] | [BACK]  

## Documentation for struct HMI_Cmds_typ  

```pascal
STRUCT HMI_Cmds_typ:
    CtrlOn : BOOL := TRUE; (*Control is activated*)
    UpCmd : BOOL; (*Command for platform UP*)
    DownCmd : BOOL; (*Command for platform DOWN*)
    LeftCmd : BOOL; (*Command for platform LEFT*)
    RightCmd : BOOL; (*Command for platform RIGHT*)
    LeftUpCmd : BOOL; (*Command for platform LEFT AND UP*)
    RightUpCmd : BOOL; (*Command for platform RIGHT AND UP*)
    LeftDownCmd : BOOL; (*Command for platform LEFT AND DOWN*)
    RightDownCmd : BOOL; (*Command for platform RIGHT AND DOWN*)
    OpenWaterValve : BOOL; (*Command to open the water valve*)
  
END_STRUCT
```

## Metrics  

Number of components: 10  

---
Autogenerated with [ia_tools](https://github.com/tkucic/ia_tools)  

[MAIN]: ../../../../index_st.md
[NAMESPACES]: ../../nsList_st.md
[METRICS]: ../../../metrics_st.md
[BACK]: ../nsMain_st.md