# Example from the Modelica Conference 2021
Tutorial by Tobias Thummerer, Johannes Stoljar

This example was updated over time to keep track with developments and changes in *FMI.jl*.

🚧 This tutorial is under revision and will be replaced by an up-to-date version soon 🚧

## License


```julia
# Copyright (c) 2021 Tobias Thummerer, Lars Mikelsons, Josef Kircher, Johannes Stoljar
# Licensed under the MIT license. 
# See LICENSE (https://github.com/thummeto/FMI.jl/blob/main/LICENSE) file in the project root for details.
```

## Introduction to the example
FMUs can be simulated in multiple ways using *FMI.jl*. You can use a very simple interface, that offers possibilities that satisfy almost any user requirement. However, if you need to build a custom simulation loop for your use case using the core FMI functions, we show that too.

![svg](https://github.com/thummeto/FMI.jl/blob/main/docs/src/examples/pics/SpringFrictionPendulum1D.svg?raw=true)  

## Other formats
Besides, this [Jupyter Notebook](https://github.com/thummeto/FMI.jl/blob/examples/examples/src/modelica_conference_2021.ipynb) there is also a [Julia file](https://github.com/thummeto/FMI.jl/blob/examples/examples/src/modelica_conference_2021.jl) with the same name, which contains only the code cells and for the documentation there is a [Markdown file](https://github.com/thummeto/FMI.jl/blob/examples/examples/src/modelica_conference_2021.md) corresponding to the notebook.  

## Code section

To run the example, the previously installed packages must be included. 


```julia
# imports
using FMI
using FMIZoo
using Plots
```

### Simulation setup

Next, the start time and end time of the simulation are set. Finally, a step size is specified to store the results of the simulation at these time steps.


```julia
tStart = 0.0
tStep = 0.1
tStop = 8.0
tSave = tStart:tStep:tStop
```




    0.0:0.1:8.0



### Simple FMU Simulation
Next, the FMU model from *FMIZoo.jl* is loaded and the information about the FMU is shown.


```julia
# we use an FMU from the FMIZoo.jl
fmu = loadFMU("SpringFrictionPendulum1D", "Dymola", "2022x")
info(fmu)
```

    #################### Begin information for FMU ####################
    	Model name:			SpringFrictionPendulum1D
    	FMI-Version:			2.0
    	GUID:				{2e178ad3-5e9b-48ec-a7b2-baa5669efc0c}
    	Generation tool:		Dymola Version 2022x (64-bit), 2021-10-08
    	Generation time:		2022-05-19T06:54:12Z
    	Var. naming conv.:		structured
    	Event indicators:		24
    	Inputs:				0
    	Outputs:			0
    	States:				2
    

    		33554432 ["mass.s"]
    		33554433 ["mass.v", "mass.v_relfric"]
    	Parameters:			12
    		16777216 ["fricScale"]
    		16777217 ["s0"]
    		16777218 ["v0"]
    		16777219 ["fixed.s0"]
    		...
    		16777223 ["mass.smin"]
    		16777224 ["mass.v_small"]
    		16777225 ["mass.L"]
    		16777226 ["mass.m"]
    		16777227 ["mass.fexp"]
    	Supports Co-Simulation:		true
    		Model identifier:	SpringFrictionPendulum1D
    		Get/Set State:		true
    		Serialize State:	true
    		Dir. Derivatives:	true
    		Var. com. steps:	true
    		Input interpol.:	true
    		Max order out. der.:	1
    	Supports Model-Exchange:	true
    		Model identifier:	SpringFrictionPendulum1D
    		Get/Set State:		true
    		Serialize State:	true
    		Dir. Derivatives:	true
    ##################### End information for FMU #####################
    

### Easy Simulation
In the next commands the FMU is simulated, for which the start and end time and recorded variables are declared. Afterwards the simulation result is plotted. In the plot for the FMU, it can be seen that the oscillation keeps decreasing due to the effect of friction. If one simulates long enough, the oscillation comes to a standstill after a certain time.


```julia
simData = simulate(fmu, (tStart, tStop); recordValues=["mass.s"], saveat=tSave)
plot(simData)
```




<?xml version="1.0" encoding="utf-8"?>
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="600" height="400" viewBox="0 0 2400 1600">
<defs>
  <clipPath id="clip890">
    <rect x="0" y="0" width="2400" height="1600"/>
  </clipPath>
</defs>
<path clip-path="url(#clip890)" d="M0 1600 L2400 1600 L2400 0 L0 0  Z" fill="#ffffff" fill-rule="evenodd" fill-opacity="1"/>
<defs>
  <clipPath id="clip891">
    <rect x="480" y="0" width="1681" height="1600"/>
  </clipPath>
</defs>
<path clip-path="url(#clip890)" d="M156.274 1423.18 L2352.76 1423.18 L2352.76 47.2441 L156.274 47.2441  Z" fill="#ffffff" fill-rule="evenodd" fill-opacity="1"/>
<defs>
  <clipPath id="clip892">
    <rect x="156" y="47" width="2197" height="1377"/>
  </clipPath>
</defs>
<polyline clip-path="url(#clip892)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:2; stroke-opacity:0.1; fill:none" points="218.439,1423.18 218.439,47.2441 "/>
<polyline clip-path="url(#clip892)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:2; stroke-opacity:0.1; fill:none" points="736.477,1423.18 736.477,47.2441 "/>
<polyline clip-path="url(#clip892)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:2; stroke-opacity:0.1; fill:none" points="1254.52,1423.18 1254.52,47.2441 "/>
<polyline clip-path="url(#clip892)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:2; stroke-opacity:0.1; fill:none" points="1772.55,1423.18 1772.55,47.2441 "/>
<polyline clip-path="url(#clip892)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:2; stroke-opacity:0.1; fill:none" points="2290.59,1423.18 2290.59,47.2441 "/>
<polyline clip-path="url(#clip892)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:2; stroke-opacity:0.1; fill:none" points="156.274,1244.06 2352.76,1244.06 "/>
<polyline clip-path="url(#clip892)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:2; stroke-opacity:0.1; fill:none" points="156.274,963.692 2352.76,963.692 "/>
<polyline clip-path="url(#clip892)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:2; stroke-opacity:0.1; fill:none" points="156.274,683.328 2352.76,683.328 "/>
<polyline clip-path="url(#clip892)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:2; stroke-opacity:0.1; fill:none" points="156.274,402.964 2352.76,402.964 "/>
<polyline clip-path="url(#clip892)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:2; stroke-opacity:0.1; fill:none" points="156.274,122.6 2352.76,122.6 "/>
<polyline clip-path="url(#clip890)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="156.274,1423.18 2352.76,1423.18 "/>
<polyline clip-path="url(#clip890)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="218.439,1423.18 218.439,1404.28 "/>
<polyline clip-path="url(#clip890)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="736.477,1423.18 736.477,1404.28 "/>
<polyline clip-path="url(#clip890)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="1254.52,1423.18 1254.52,1404.28 "/>
<polyline clip-path="url(#clip890)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="1772.55,1423.18 1772.55,1404.28 "/>
<polyline clip-path="url(#clip890)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="2290.59,1423.18 2290.59,1404.28 "/>
<path clip-path="url(#clip890)" d="M218.439 1454.1 Q214.828 1454.1 212.999 1457.66 Q211.193 1461.2 211.193 1468.33 Q211.193 1475.44 212.999 1479.01 Q214.828 1482.55 218.439 1482.55 Q222.073 1482.55 223.879 1479.01 Q225.707 1475.44 225.707 1468.33 Q225.707 1461.2 223.879 1457.66 Q222.073 1454.1 218.439 1454.1 M218.439 1450.39 Q224.249 1450.39 227.304 1455 Q230.383 1459.58 230.383 1468.33 Q230.383 1477.06 227.304 1481.67 Q224.249 1486.25 218.439 1486.25 Q212.629 1486.25 209.55 1481.67 Q206.494 1477.06 206.494 1468.33 Q206.494 1459.58 209.55 1455 Q212.629 1450.39 218.439 1450.39 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip890)" d="M731.13 1481.64 L747.449 1481.64 L747.449 1485.58 L725.505 1485.58 L725.505 1481.64 Q728.167 1478.89 732.75 1474.26 Q737.357 1469.61 738.537 1468.27 Q740.782 1465.74 741.662 1464.01 Q742.565 1462.25 742.565 1460.56 Q742.565 1457.8 740.62 1456.07 Q738.699 1454.33 735.597 1454.33 Q733.398 1454.33 730.945 1455.09 Q728.514 1455.86 725.736 1457.41 L725.736 1452.69 Q728.56 1451.55 731.014 1450.97 Q733.468 1450.39 735.505 1450.39 Q740.875 1450.39 744.069 1453.08 Q747.264 1455.77 747.264 1460.26 Q747.264 1462.39 746.454 1464.31 Q745.667 1466.2 743.56 1468.8 Q742.981 1469.47 739.88 1472.69 Q736.778 1475.88 731.13 1481.64 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip890)" d="M1257.52 1455.09 L1245.72 1473.54 L1257.52 1473.54 L1257.52 1455.09 M1256.3 1451.02 L1262.18 1451.02 L1262.18 1473.54 L1267.11 1473.54 L1267.11 1477.43 L1262.18 1477.43 L1262.18 1485.58 L1257.52 1485.58 L1257.52 1477.43 L1241.92 1477.43 L1241.92 1472.92 L1256.3 1451.02 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip890)" d="M1772.96 1466.44 Q1769.81 1466.44 1767.96 1468.59 Q1766.13 1470.74 1766.13 1474.49 Q1766.13 1478.22 1767.96 1480.39 Q1769.81 1482.55 1772.96 1482.55 Q1776.11 1482.55 1777.94 1480.39 Q1779.79 1478.22 1779.79 1474.49 Q1779.79 1470.74 1777.94 1468.59 Q1776.11 1466.44 1772.96 1466.44 M1782.24 1451.78 L1782.24 1456.04 Q1780.48 1455.21 1778.68 1454.77 Q1776.89 1454.33 1775.13 1454.33 Q1770.5 1454.33 1768.05 1457.45 Q1765.62 1460.58 1765.27 1466.9 Q1766.64 1464.89 1768.7 1463.82 Q1770.76 1462.73 1773.24 1462.73 Q1778.44 1462.73 1781.45 1465.9 Q1784.49 1469.05 1784.49 1474.49 Q1784.49 1479.82 1781.34 1483.03 Q1778.19 1486.25 1772.96 1486.25 Q1766.96 1486.25 1763.79 1481.67 Q1760.62 1477.06 1760.62 1468.33 Q1760.62 1460.14 1764.51 1455.28 Q1768.4 1450.39 1774.95 1450.39 Q1776.71 1450.39 1778.49 1450.74 Q1780.3 1451.09 1782.24 1451.78 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip890)" d="M2290.59 1469.17 Q2287.26 1469.17 2285.34 1470.95 Q2283.44 1472.73 2283.44 1475.86 Q2283.44 1478.98 2285.34 1480.77 Q2287.26 1482.55 2290.59 1482.55 Q2293.92 1482.55 2295.85 1480.77 Q2297.77 1478.96 2297.77 1475.86 Q2297.77 1472.73 2295.85 1470.95 Q2293.95 1469.17 2290.59 1469.17 M2285.92 1467.18 Q2282.91 1466.44 2281.22 1464.38 Q2279.55 1462.32 2279.55 1459.35 Q2279.55 1455.21 2282.49 1452.8 Q2285.45 1450.39 2290.59 1450.39 Q2295.75 1450.39 2298.69 1452.8 Q2301.63 1455.21 2301.63 1459.35 Q2301.63 1462.32 2299.94 1464.38 Q2298.28 1466.44 2295.29 1467.18 Q2298.67 1467.96 2300.54 1470.26 Q2302.44 1472.55 2302.44 1475.86 Q2302.44 1480.88 2299.36 1483.57 Q2296.31 1486.25 2290.59 1486.25 Q2284.87 1486.25 2281.8 1483.57 Q2278.74 1480.88 2278.74 1475.86 Q2278.74 1472.55 2280.64 1470.26 Q2282.54 1467.96 2285.92 1467.18 M2284.2 1459.79 Q2284.2 1462.48 2285.87 1463.98 Q2287.56 1465.49 2290.59 1465.49 Q2293.6 1465.49 2295.29 1463.98 Q2297 1462.48 2297 1459.79 Q2297 1457.11 2295.29 1455.6 Q2293.6 1454.1 2290.59 1454.1 Q2287.56 1454.1 2285.87 1455.6 Q2284.2 1457.11 2284.2 1459.79 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip890)" d="M1202.83 1522.27 L1202.83 1532.4 L1214.89 1532.4 L1214.89 1536.95 L1202.83 1536.95 L1202.83 1556.3 Q1202.83 1560.66 1204 1561.9 Q1205.21 1563.14 1208.87 1563.14 L1214.89 1563.14 L1214.89 1568.04 L1208.87 1568.04 Q1202.09 1568.04 1199.52 1565.53 Q1196.94 1562.98 1196.94 1556.3 L1196.94 1536.95 L1192.64 1536.95 L1192.64 1532.4 L1196.94 1532.4 L1196.94 1522.27 L1202.83 1522.27 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip890)" d="M1242.77 1518.52 L1256.27 1518.52 L1256.27 1523.07 L1248.63 1523.07 L1248.63 1572.09 L1256.27 1572.09 L1256.27 1576.64 L1242.77 1576.64 L1242.77 1518.52 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip890)" d="M1291.47 1533.45 L1291.47 1538.98 Q1288.99 1537.71 1286.31 1537.07 Q1283.64 1536.44 1280.77 1536.44 Q1276.41 1536.44 1274.22 1537.77 Q1272.05 1539.11 1272.05 1541.79 Q1272.05 1543.82 1273.61 1545 Q1275.17 1546.15 1279.88 1547.2 L1281.89 1547.64 Q1288.13 1548.98 1290.74 1551.43 Q1293.38 1553.85 1293.38 1558.21 Q1293.38 1563.17 1289.43 1566.07 Q1285.52 1568.97 1278.64 1568.97 Q1275.78 1568.97 1272.66 1568.39 Q1269.57 1567.85 1266.13 1566.74 L1266.13 1560.69 Q1269.38 1562.38 1272.53 1563.24 Q1275.68 1564.07 1278.77 1564.07 Q1282.91 1564.07 1285.13 1562.66 Q1287.36 1561.23 1287.36 1558.65 Q1287.36 1556.27 1285.74 1554.99 Q1284.15 1553.72 1278.7 1552.54 L1276.67 1552.07 Q1271.23 1550.92 1268.81 1548.56 Q1266.39 1546.18 1266.39 1542.04 Q1266.39 1537.01 1269.95 1534.27 Q1273.52 1531.54 1280.07 1531.54 Q1283.32 1531.54 1286.18 1532.01 Q1289.05 1532.49 1291.47 1533.45 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip890)" d="M1316.39 1518.52 L1316.39 1576.64 L1302.89 1576.64 L1302.89 1572.09 L1310.5 1572.09 L1310.5 1523.07 L1302.89 1523.07 L1302.89 1518.52 L1316.39 1518.52 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><polyline clip-path="url(#clip890)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="156.274,1423.18 156.274,47.2441 "/>
<polyline clip-path="url(#clip890)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="156.274,1244.06 175.172,1244.06 "/>
<polyline clip-path="url(#clip890)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="156.274,963.692 175.172,963.692 "/>
<polyline clip-path="url(#clip890)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="156.274,683.328 175.172,683.328 "/>
<polyline clip-path="url(#clip890)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="156.274,402.964 175.172,402.964 "/>
<polyline clip-path="url(#clip890)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="156.274,122.6 175.172,122.6 "/>
<path clip-path="url(#clip890)" d="M62.9365 1229.85 Q59.3254 1229.85 57.4967 1233.42 Q55.6912 1236.96 55.6912 1244.09 Q55.6912 1251.2 57.4967 1254.76 Q59.3254 1258.3 62.9365 1258.3 Q66.5707 1258.3 68.3763 1254.76 Q70.205 1251.2 70.205 1244.09 Q70.205 1236.96 68.3763 1233.42 Q66.5707 1229.85 62.9365 1229.85 M62.9365 1226.15 Q68.7467 1226.15 71.8022 1230.76 Q74.8809 1235.34 74.8809 1244.09 Q74.8809 1252.82 71.8022 1257.42 Q68.7467 1262.01 62.9365 1262.01 Q57.1264 1262.01 54.0477 1257.42 Q50.9921 1252.82 50.9921 1244.09 Q50.9921 1235.34 54.0477 1230.76 Q57.1264 1226.15 62.9365 1226.15 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip890)" d="M83.0984 1255.46 L87.9827 1255.46 L87.9827 1261.34 L83.0984 1261.34 L83.0984 1255.46 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip890)" d="M108.746 1242.19 Q105.598 1242.19 103.746 1244.35 Q101.918 1246.5 101.918 1250.25 Q101.918 1253.98 103.746 1256.15 Q105.598 1258.3 108.746 1258.3 Q111.895 1258.3 113.723 1256.15 Q115.575 1253.98 115.575 1250.25 Q115.575 1246.5 113.723 1244.35 Q111.895 1242.19 108.746 1242.19 M118.029 1227.54 L118.029 1231.8 Q116.27 1230.97 114.464 1230.53 Q112.682 1230.09 110.922 1230.09 Q106.293 1230.09 103.839 1233.21 Q101.409 1236.34 101.061 1242.66 Q102.427 1240.64 104.487 1239.58 Q106.547 1238.49 109.024 1238.49 Q114.233 1238.49 117.242 1241.66 Q120.274 1244.81 120.274 1250.25 Q120.274 1255.57 117.126 1258.79 Q113.978 1262.01 108.746 1262.01 Q102.751 1262.01 99.5798 1257.42 Q96.4085 1252.82 96.4085 1244.09 Q96.4085 1235.9 100.297 1231.04 Q104.186 1226.15 110.737 1226.15 Q112.496 1226.15 114.279 1226.5 Q116.084 1226.85 118.029 1227.54 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip890)" d="M63.1911 949.491 Q59.58 949.491 57.7513 953.056 Q55.9458 956.597 55.9458 963.727 Q55.9458 970.833 57.7513 974.398 Q59.58 977.94 63.1911 977.94 Q66.8254 977.94 68.6309 974.398 Q70.4596 970.833 70.4596 963.727 Q70.4596 956.597 68.6309 953.056 Q66.8254 949.491 63.1911 949.491 M63.1911 945.787 Q69.0013 945.787 72.0568 950.394 Q75.1355 954.977 75.1355 963.727 Q75.1355 972.454 72.0568 977.06 Q69.0013 981.644 63.1911 981.644 Q57.381 981.644 54.3023 977.06 Q51.2468 972.454 51.2468 963.727 Q51.2468 954.977 54.3023 950.394 Q57.381 945.787 63.1911 945.787 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip890)" d="M83.3531 975.093 L88.2373 975.093 L88.2373 980.972 L83.3531 980.972 L83.3531 975.093 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip890)" d="M108.422 964.56 Q105.089 964.56 103.168 966.343 Q101.27 968.125 101.27 971.25 Q101.27 974.375 103.168 976.158 Q105.089 977.94 108.422 977.94 Q111.756 977.94 113.677 976.158 Q115.598 974.352 115.598 971.25 Q115.598 968.125 113.677 966.343 Q111.779 964.56 108.422 964.56 M103.746 962.57 Q100.737 961.829 99.0474 959.769 Q97.3808 957.709 97.3808 954.746 Q97.3808 950.602 100.321 948.195 Q103.284 945.787 108.422 945.787 Q113.584 945.787 116.524 948.195 Q119.464 950.602 119.464 954.746 Q119.464 957.709 117.774 959.769 Q116.108 961.829 113.121 962.57 Q116.501 963.357 118.376 965.648 Q120.274 967.94 120.274 971.25 Q120.274 976.273 117.195 978.958 Q114.14 981.644 108.422 981.644 Q102.705 981.644 99.6261 978.958 Q96.5706 976.273 96.5706 971.25 Q96.5706 967.94 98.4687 965.648 Q100.367 963.357 103.746 962.57 M102.034 955.185 Q102.034 957.871 103.7 959.375 Q105.39 960.88 108.422 960.88 Q111.432 960.88 113.121 959.375 Q114.834 957.871 114.834 955.185 Q114.834 952.5 113.121 950.996 Q111.432 949.491 108.422 949.491 Q105.39 949.491 103.7 950.996 Q102.034 952.5 102.034 955.185 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip890)" d="M53.9088 696.673 L61.5476 696.673 L61.5476 670.308 L53.2375 671.974 L53.2375 667.715 L61.5013 666.048 L66.1772 666.048 L66.1772 696.673 L73.8161 696.673 L73.8161 700.608 L53.9088 700.608 L53.9088 696.673 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip890)" d="M83.2605 694.729 L88.1447 694.729 L88.1447 700.608 L83.2605 700.608 L83.2605 694.729 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip890)" d="M108.33 669.127 Q104.719 669.127 102.89 672.692 Q101.084 676.234 101.084 683.363 Q101.084 690.47 102.89 694.034 Q104.719 697.576 108.33 697.576 Q111.964 697.576 113.77 694.034 Q115.598 690.47 115.598 683.363 Q115.598 676.234 113.77 672.692 Q111.964 669.127 108.33 669.127 M108.33 665.423 Q114.14 665.423 117.195 670.03 Q120.274 674.613 120.274 683.363 Q120.274 692.09 117.195 696.696 Q114.14 701.28 108.33 701.28 Q102.52 701.28 99.4409 696.696 Q96.3854 692.09 96.3854 683.363 Q96.3854 674.613 99.4409 670.03 Q102.52 665.423 108.33 665.423 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip890)" d="M55.506 416.309 L63.1448 416.309 L63.1448 389.944 L54.8347 391.61 L54.8347 387.351 L63.0985 385.684 L67.7744 385.684 L67.7744 416.309 L75.4133 416.309 L75.4133 420.244 L55.506 420.244 L55.506 416.309 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip890)" d="M84.8577 414.365 L89.7419 414.365 L89.7419 420.244 L84.8577 420.244 L84.8577 414.365 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip890)" d="M103.955 416.309 L120.274 416.309 L120.274 420.244 L98.3298 420.244 L98.3298 416.309 Q100.992 413.555 105.575 408.925 Q110.182 404.272 111.362 402.93 Q113.608 400.407 114.487 398.67 Q115.39 396.911 115.39 395.221 Q115.39 392.467 113.445 390.731 Q111.524 388.995 108.422 388.995 Q106.223 388.995 103.77 389.758 Q101.339 390.522 98.5613 392.073 L98.5613 387.351 Q101.385 386.217 103.839 385.638 Q106.293 385.059 108.33 385.059 Q113.7 385.059 116.895 387.745 Q120.089 390.43 120.089 394.92 Q120.089 397.05 119.279 398.971 Q118.492 400.87 116.385 403.462 Q115.807 404.133 112.705 407.351 Q109.603 410.545 103.955 416.309 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip890)" d="M53.4227 135.945 L61.0615 135.945 L61.0615 109.58 L52.7514 111.246 L52.7514 106.987 L61.0152 105.32 L65.6911 105.32 L65.6911 135.945 L73.33 135.945 L73.33 139.88 L53.4227 139.88 L53.4227 135.945 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip890)" d="M82.7744 134.001 L87.6586 134.001 L87.6586 139.88 L82.7744 139.88 L82.7744 134.001 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip890)" d="M110.691 109.395 L98.8854 127.843 L110.691 127.843 L110.691 109.395 M109.464 105.32 L115.344 105.32 L115.344 127.843 L120.274 127.843 L120.274 131.732 L115.344 131.732 L115.344 139.88 L110.691 139.88 L110.691 131.732 L95.0891 131.732 L95.0891 127.218 L109.464 105.32 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><polyline clip-path="url(#clip892)" style="stroke:#009af9; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="218.439,1384.24 244.341,1353.91 270.243,1264.1 296.144,1122.2 322.046,941.759 347.948,740.285 373.85,537.787 399.752,354.597 425.654,209.03 451.556,115.987 477.458,86.1857 503.36,114.41 529.262,191.764 555.164,312.146 581.065,464.311 606.967,633.356 632.869,802.554 658.771,955.08 684.673,1075.61 710.575,1151.57 736.477,1173.79 762.379,1149.44 788.281,1086.49 814.183,989.727 840.085,868.02 865.986,733.157 891.888,598.346 917.79,477.078 943.692,381.776 969.594,322.668 995.496,307.014 1021.4,325.617 1047.3,371.505 1073.2,441.424 1099.1,529.246 1125.01,626.715 1150.91,724.248 1176.81,812.074 1202.71,881.092 1228.61,923.665 1254.52,934.28 1280.42,923.674 1306.32,898.284 1332.22,859.662 1358.12,810.994 1384.02,756.625 1409.93,701.81 1435.83,652.076 1461.73,612.721 1487.63,588.339 1513.53,582.117 1539.44,582.117 1565.34,582.117 1591.24,582.117 1617.14,582.117 1643.04,582.117 1668.95,582.117 1694.85,582.117 1720.75,582.117 1746.65,582.117 1772.55,582.117 1798.46,582.117 1824.36,582.117 1850.26,582.117 1876.16,582.117 1902.06,582.117 1927.96,582.117 1953.87,582.117 1979.77,582.117 2005.67,582.117 2031.57,582.117 2057.47,582.117 2083.38,582.117 2109.28,582.117 2135.18,582.117 2161.08,582.117 2186.98,582.117 2212.89,582.117 2238.79,582.117 2264.69,582.117 2290.59,582.117 "/>
<path clip-path="url(#clip890)" d="M1610.52 196.789 L2279.54 196.789 L2279.54 93.1086 L1610.52 93.1086  Z" fill="#ffffff" fill-rule="evenodd" fill-opacity="1"/>
<polyline clip-path="url(#clip890)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="1610.52,196.789 2279.54,196.789 2279.54,93.1086 1610.52,93.1086 1610.52,196.789 "/>
<polyline clip-path="url(#clip890)" style="stroke:#009af9; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="1634.92,144.949 1781.36,144.949 "/>
<path clip-path="url(#clip890)" d="M1826.11 141.28 Q1827.71 138.409 1829.93 137.044 Q1832.15 135.678 1835.16 135.678 Q1839.21 135.678 1841.41 138.525 Q1843.61 141.349 1843.61 146.581 L1843.61 162.229 L1839.33 162.229 L1839.33 146.719 Q1839.33 142.993 1838.01 141.187 Q1836.69 139.382 1833.98 139.382 Q1830.67 139.382 1828.75 141.581 Q1826.83 143.78 1826.83 147.576 L1826.83 162.229 L1822.54 162.229 L1822.54 146.719 Q1822.54 142.969 1821.22 141.187 Q1819.91 139.382 1817.15 139.382 Q1813.89 139.382 1811.97 141.604 Q1810.04 143.803 1810.04 147.576 L1810.04 162.229 L1805.76 162.229 L1805.76 136.303 L1810.04 136.303 L1810.04 140.331 Q1811.5 137.946 1813.54 136.812 Q1815.58 135.678 1818.38 135.678 Q1821.2 135.678 1823.17 137.113 Q1825.16 138.548 1826.11 141.28 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip890)" d="M1863.89 149.196 Q1858.72 149.196 1856.73 150.377 Q1854.74 151.557 1854.74 154.405 Q1854.74 156.673 1856.22 158.016 Q1857.73 159.335 1860.3 159.335 Q1863.84 159.335 1865.97 156.835 Q1868.12 154.312 1868.12 150.145 L1868.12 149.196 L1863.89 149.196 M1872.38 147.437 L1872.38 162.229 L1868.12 162.229 L1868.12 158.293 Q1866.66 160.655 1864.49 161.789 Q1862.31 162.9 1859.16 162.9 Q1855.18 162.9 1852.82 160.678 Q1850.48 158.432 1850.48 154.682 Q1850.48 150.307 1853.4 148.085 Q1856.34 145.863 1862.15 145.863 L1868.12 145.863 L1868.12 145.446 Q1868.12 142.507 1866.18 140.909 Q1864.26 139.289 1860.76 139.289 Q1858.54 139.289 1856.43 139.821 Q1854.33 140.354 1852.38 141.419 L1852.38 137.483 Q1854.72 136.581 1856.92 136.141 Q1859.12 135.678 1861.2 135.678 Q1866.83 135.678 1869.6 138.594 Q1872.38 141.511 1872.38 147.437 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip890)" d="M1897.68 137.067 L1897.68 141.094 Q1895.88 140.169 1893.93 139.706 Q1891.99 139.243 1889.9 139.243 Q1886.73 139.243 1885.14 140.215 Q1883.56 141.187 1883.56 143.131 Q1883.56 144.613 1884.7 145.469 Q1885.83 146.303 1889.26 147.067 L1890.72 147.391 Q1895.25 148.363 1897.15 150.145 Q1899.07 151.905 1899.07 155.076 Q1899.07 158.687 1896.2 160.793 Q1893.35 162.9 1888.35 162.9 Q1886.27 162.9 1884 162.483 Q1881.76 162.09 1879.26 161.28 L1879.26 156.881 Q1881.62 158.108 1883.91 158.733 Q1886.2 159.335 1888.45 159.335 Q1891.46 159.335 1893.08 158.317 Q1894.7 157.275 1894.7 155.4 Q1894.7 153.664 1893.52 152.738 Q1892.36 151.812 1888.4 150.956 L1886.92 150.608 Q1882.96 149.775 1881.2 148.062 Q1879.44 146.326 1879.44 143.317 Q1879.44 139.659 1882.03 137.669 Q1884.63 135.678 1889.4 135.678 Q1891.76 135.678 1893.84 136.025 Q1895.92 136.372 1897.68 137.067 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip890)" d="M1922.38 137.067 L1922.38 141.094 Q1920.58 140.169 1918.63 139.706 Q1916.69 139.243 1914.6 139.243 Q1911.43 139.243 1909.84 140.215 Q1908.26 141.187 1908.26 143.131 Q1908.26 144.613 1909.4 145.469 Q1910.53 146.303 1913.96 147.067 L1915.41 147.391 Q1919.95 148.363 1921.85 150.145 Q1923.77 151.905 1923.77 155.076 Q1923.77 158.687 1920.9 160.793 Q1918.05 162.9 1913.05 162.9 Q1910.97 162.9 1908.7 162.483 Q1906.46 162.09 1903.96 161.28 L1903.96 156.881 Q1906.32 158.108 1908.61 158.733 Q1910.9 159.335 1913.15 159.335 Q1916.15 159.335 1917.78 158.317 Q1919.4 157.275 1919.4 155.4 Q1919.4 153.664 1918.22 152.738 Q1917.06 151.812 1913.1 150.956 L1911.62 150.608 Q1907.66 149.775 1905.9 148.062 Q1904.14 146.326 1904.14 143.317 Q1904.14 139.659 1906.73 137.669 Q1909.33 135.678 1914.09 135.678 Q1916.46 135.678 1918.54 136.025 Q1920.62 136.372 1922.38 137.067 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip890)" d="M1931.15 156.349 L1936.04 156.349 L1936.04 162.229 L1931.15 162.229 L1931.15 156.349 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip890)" d="M1962.15 137.067 L1962.15 141.094 Q1960.34 140.169 1958.4 139.706 Q1956.46 139.243 1954.37 139.243 Q1951.2 139.243 1949.6 140.215 Q1948.03 141.187 1948.03 143.131 Q1948.03 144.613 1949.16 145.469 Q1950.3 146.303 1953.72 147.067 L1955.18 147.391 Q1959.72 148.363 1961.62 150.145 Q1963.54 151.905 1963.54 155.076 Q1963.54 158.687 1960.67 160.793 Q1957.82 162.9 1952.82 162.9 Q1950.74 162.9 1948.47 162.483 Q1946.22 162.09 1943.72 161.28 L1943.72 156.881 Q1946.09 158.108 1948.38 158.733 Q1950.67 159.335 1952.91 159.335 Q1955.92 159.335 1957.54 158.317 Q1959.16 157.275 1959.16 155.4 Q1959.16 153.664 1957.98 152.738 Q1956.83 151.812 1952.87 150.956 L1951.39 150.608 Q1947.43 149.775 1945.67 148.062 Q1943.91 146.326 1943.91 143.317 Q1943.91 139.659 1946.5 137.669 Q1949.09 135.678 1953.86 135.678 Q1956.22 135.678 1958.31 136.025 Q1960.39 136.372 1962.15 137.067 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip890)" d="M1995.62 126.257 Q1992.52 131.581 1991.02 136.789 Q1989.51 141.997 1989.51 147.344 Q1989.51 152.692 1991.02 157.946 Q1992.54 163.178 1995.62 168.479 L1991.92 168.479 Q1988.45 163.039 1986.71 157.784 Q1985 152.53 1985 147.344 Q1985 142.182 1986.71 136.951 Q1988.42 131.72 1991.92 126.257 L1995.62 126.257 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip890)" d="M2018.65 143.594 Q2022.01 144.312 2023.89 146.581 Q2025.78 148.849 2025.78 152.182 Q2025.78 157.298 2022.27 160.099 Q2018.75 162.9 2012.27 162.9 Q2010.09 162.9 2007.77 162.46 Q2005.48 162.043 2003.03 161.187 L2003.03 156.673 Q2004.97 157.807 2007.29 158.386 Q2009.6 158.965 2012.13 158.965 Q2016.52 158.965 2018.82 157.229 Q2021.13 155.493 2021.13 152.182 Q2021.13 149.127 2018.98 147.414 Q2016.85 145.678 2013.03 145.678 L2009 145.678 L2009 141.835 L2013.21 141.835 Q2016.66 141.835 2018.49 140.469 Q2020.32 139.081 2020.32 136.488 Q2020.32 133.826 2018.42 132.414 Q2016.55 130.979 2013.03 130.979 Q2011.11 130.979 2008.91 131.395 Q2006.71 131.812 2004.07 132.692 L2004.07 128.525 Q2006.73 127.784 2009.05 127.414 Q2011.39 127.044 2013.45 127.044 Q2018.77 127.044 2021.87 129.474 Q2024.97 131.882 2024.97 136.002 Q2024.97 138.872 2023.33 140.863 Q2021.69 142.831 2018.65 143.594 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip890)" d="M2048.82 143.594 Q2052.17 144.312 2054.05 146.581 Q2055.95 148.849 2055.95 152.182 Q2055.95 157.298 2052.43 160.099 Q2048.91 162.9 2042.43 162.9 Q2040.25 162.9 2037.94 162.46 Q2035.64 162.043 2033.19 161.187 L2033.19 156.673 Q2035.14 157.807 2037.45 158.386 Q2039.77 158.965 2042.29 158.965 Q2046.69 158.965 2048.98 157.229 Q2051.29 155.493 2051.29 152.182 Q2051.29 149.127 2049.14 147.414 Q2047.01 145.678 2043.19 145.678 L2039.16 145.678 L2039.16 141.835 L2043.38 141.835 Q2046.83 141.835 2048.65 140.469 Q2050.48 139.081 2050.48 136.488 Q2050.48 133.826 2048.58 132.414 Q2046.71 130.979 2043.19 130.979 Q2041.27 130.979 2039.07 131.395 Q2036.87 131.812 2034.23 132.692 L2034.23 128.525 Q2036.89 127.784 2039.21 127.414 Q2041.55 127.044 2043.61 127.044 Q2048.93 127.044 2052.03 129.474 Q2055.14 131.882 2055.14 136.002 Q2055.14 138.872 2053.49 140.863 Q2051.85 142.831 2048.82 143.594 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip890)" d="M2064.86 127.669 L2083.21 127.669 L2083.21 131.604 L2069.14 131.604 L2069.14 140.076 Q2070.16 139.729 2071.18 139.567 Q2072.2 139.382 2073.21 139.382 Q2079 139.382 2082.38 142.553 Q2085.76 145.724 2085.76 151.141 Q2085.76 156.719 2082.29 159.821 Q2078.82 162.9 2072.5 162.9 Q2070.32 162.9 2068.05 162.53 Q2065.81 162.159 2063.4 161.418 L2063.4 156.719 Q2065.48 157.854 2067.7 158.409 Q2069.93 158.965 2072.4 158.965 Q2076.41 158.965 2078.75 156.858 Q2081.08 154.752 2081.08 151.141 Q2081.08 147.53 2078.75 145.423 Q2076.41 143.317 2072.4 143.317 Q2070.53 143.317 2068.65 143.733 Q2066.8 144.15 2064.86 145.03 L2064.86 127.669 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip890)" d="M2095.02 127.669 L2113.38 127.669 L2113.38 131.604 L2099.3 131.604 L2099.3 140.076 Q2100.32 139.729 2101.34 139.567 Q2102.36 139.382 2103.38 139.382 Q2109.16 139.382 2112.54 142.553 Q2115.92 145.724 2115.92 151.141 Q2115.92 156.719 2112.45 159.821 Q2108.98 162.9 2102.66 162.9 Q2100.48 162.9 2098.21 162.53 Q2095.97 162.159 2093.56 161.418 L2093.56 156.719 Q2095.64 157.854 2097.87 158.409 Q2100.09 158.965 2102.57 158.965 Q2106.57 158.965 2108.91 156.858 Q2111.25 154.752 2111.25 151.141 Q2111.25 147.53 2108.91 145.423 Q2106.57 143.317 2102.57 143.317 Q2100.69 143.317 2098.82 143.733 Q2096.96 144.15 2095.02 145.03 L2095.02 127.669 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip890)" d="M2137.98 131.743 L2126.18 150.192 L2137.98 150.192 L2137.98 131.743 M2136.76 127.669 L2142.64 127.669 L2142.64 150.192 L2147.57 150.192 L2147.57 154.081 L2142.64 154.081 L2142.64 162.229 L2137.98 162.229 L2137.98 154.081 L2122.38 154.081 L2122.38 149.567 L2136.76 127.669 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip890)" d="M2168.14 131.743 L2156.34 150.192 L2168.14 150.192 L2168.14 131.743 M2166.92 127.669 L2172.8 127.669 L2172.8 150.192 L2177.73 150.192 L2177.73 154.081 L2172.8 154.081 L2172.8 162.229 L2168.14 162.229 L2168.14 154.081 L2152.54 154.081 L2152.54 149.567 L2166.92 127.669 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip890)" d="M2199.63 143.594 Q2202.98 144.312 2204.86 146.581 Q2206.76 148.849 2206.76 152.182 Q2206.76 157.298 2203.24 160.099 Q2199.72 162.9 2193.24 162.9 Q2191.06 162.9 2188.75 162.46 Q2186.45 162.043 2184 161.187 L2184 156.673 Q2185.95 157.807 2188.26 158.386 Q2190.57 158.965 2193.1 158.965 Q2197.5 158.965 2199.79 157.229 Q2202.1 155.493 2202.1 152.182 Q2202.1 149.127 2199.95 147.414 Q2197.82 145.678 2194 145.678 L2189.97 145.678 L2189.97 141.835 L2194.19 141.835 Q2197.63 141.835 2199.46 140.469 Q2201.29 139.081 2201.29 136.488 Q2201.29 133.826 2199.39 132.414 Q2197.52 130.979 2194 130.979 Q2192.08 130.979 2189.88 131.395 Q2187.68 131.812 2185.04 132.692 L2185.04 128.525 Q2187.7 127.784 2190.02 127.414 Q2192.36 127.044 2194.42 127.044 Q2199.74 127.044 2202.84 129.474 Q2205.94 131.882 2205.94 136.002 Q2205.94 138.872 2204.3 140.863 Q2202.66 142.831 2199.63 143.594 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip890)" d="M2219.65 158.293 L2235.97 158.293 L2235.97 162.229 L2214.02 162.229 L2214.02 158.293 Q2216.69 155.539 2221.27 150.909 Q2225.88 146.256 2227.06 144.914 Q2229.3 142.391 2230.18 140.655 Q2231.08 138.895 2231.08 137.206 Q2231.08 134.451 2229.14 132.715 Q2227.22 130.979 2224.12 130.979 Q2221.92 130.979 2219.46 131.743 Q2217.03 132.507 2214.26 134.057 L2214.26 129.335 Q2217.08 128.201 2219.53 127.622 Q2221.99 127.044 2224.02 127.044 Q2229.39 127.044 2232.59 129.729 Q2235.78 132.414 2235.78 136.905 Q2235.78 139.034 2234.97 140.956 Q2234.19 142.854 2232.08 145.446 Q2231.5 146.118 2228.4 149.335 Q2225.3 152.53 2219.65 158.293 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip890)" d="M2244.51 126.257 L2248.21 126.257 Q2251.69 131.72 2253.4 136.951 Q2255.13 142.182 2255.13 147.344 Q2255.13 152.53 2253.4 157.784 Q2251.69 163.039 2248.21 168.479 L2244.51 168.479 Q2247.59 163.178 2249.09 157.946 Q2250.62 152.692 2250.62 147.344 Q2250.62 141.997 2249.09 136.789 Q2247.59 131.581 2244.51 126.257 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /></svg>




After plotting the data, the FMU is unloaded and all unpacked data on disc is removed.


```julia
unloadFMU(fmu)
```

### Custom Simulation

In the following type of simulation a more advanced variant is presented, which allows intervening more in the simulation process. Analogous to the simple variant, an FMU model must be loaded.


```julia
fmu = loadFMU("SpringFrictionPendulum1D", "Dymola", "2022x")
```




    Model name:	SpringFrictionPendulum1D
    Type:		1



Next, it is necessary to create an instance of the FMU, this is achieved by the command `fmi2Instantiate!()`.  


```julia
instanceFMU = fmi2Instantiate!(fmu)
```




    FMU:            SpringFrictionPendulum1D
        InstanceName:   SpringFrictionPendulum1D
        Address:        Ptr{Nothing} @0x000001bb78171a50
        State:          0
        Logging:        false
        FMU time:       -Inf
        FMU states:     nothing



In the following code block, start and end time for the simulation is set by the `fmi2SetupExperiment()` command. Next, the FMU is initialized by the calls of `fmi2EnterInitializationMode()` and `fmi2ExitInitializationMode()`. It would also be possible to set initial states, parameters or inputs at this place in code.


```julia
fmi2SetupExperiment(instanceFMU, tStart, tStop)
# set initial model states
fmi2EnterInitializationMode(instanceFMU)
# get initial model states
fmi2ExitInitializationMode(instanceFMU)
```




    0x00000000



The actual simulation loop is shown in the following block. Here a simulation step `fmi2DoStep()` with the fixed step size `tStep` is executed. As indicated in the code by the comments, the input values and output values of the FMU could be changed in the simulation loop as desired, whereby the higher possibility of adjustments arises.


```julia
values = []

for t in tSave
    # set model inputs if any
    # ...

    fmi2DoStep(instanceFMU, tStep)
    
    # get model outputs
    value = fmi2GetReal(instanceFMU, "mass.s")
    push!(values, value)
end

plot(tSave, values)
```




<?xml version="1.0" encoding="utf-8"?>
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="600" height="400" viewBox="0 0 2400 1600">
<defs>
  <clipPath id="clip980">
    <rect x="0" y="0" width="2400" height="1600"/>
  </clipPath>
</defs>
<path clip-path="url(#clip980)" d="M0 1600 L2400 1600 L2400 0 L0 0  Z" fill="#ffffff" fill-rule="evenodd" fill-opacity="1"/>
<defs>
  <clipPath id="clip981">
    <rect x="480" y="0" width="1681" height="1600"/>
  </clipPath>
</defs>
<path clip-path="url(#clip980)" d="M156.274 1486.45 L2352.76 1486.45 L2352.76 47.2441 L156.274 47.2441  Z" fill="#ffffff" fill-rule="evenodd" fill-opacity="1"/>
<defs>
  <clipPath id="clip982">
    <rect x="156" y="47" width="2197" height="1440"/>
  </clipPath>
</defs>
<polyline clip-path="url(#clip982)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:2; stroke-opacity:0.1; fill:none" points="218.439,1486.45 218.439,47.2441 "/>
<polyline clip-path="url(#clip982)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:2; stroke-opacity:0.1; fill:none" points="736.477,1486.45 736.477,47.2441 "/>
<polyline clip-path="url(#clip982)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:2; stroke-opacity:0.1; fill:none" points="1254.52,1486.45 1254.52,47.2441 "/>
<polyline clip-path="url(#clip982)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:2; stroke-opacity:0.1; fill:none" points="1772.55,1486.45 1772.55,47.2441 "/>
<polyline clip-path="url(#clip982)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:2; stroke-opacity:0.1; fill:none" points="2290.59,1486.45 2290.59,47.2441 "/>
<polyline clip-path="url(#clip982)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:2; stroke-opacity:0.1; fill:none" points="156.274,1327.97 2352.76,1327.97 "/>
<polyline clip-path="url(#clip982)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:2; stroke-opacity:0.1; fill:none" points="156.274,1027.52 2352.76,1027.52 "/>
<polyline clip-path="url(#clip982)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:2; stroke-opacity:0.1; fill:none" points="156.274,727.065 2352.76,727.065 "/>
<polyline clip-path="url(#clip982)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:2; stroke-opacity:0.1; fill:none" points="156.274,426.615 2352.76,426.615 "/>
<polyline clip-path="url(#clip982)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:2; stroke-opacity:0.1; fill:none" points="156.274,126.164 2352.76,126.164 "/>
<polyline clip-path="url(#clip980)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="156.274,1486.45 2352.76,1486.45 "/>
<polyline clip-path="url(#clip980)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="218.439,1486.45 218.439,1467.55 "/>
<polyline clip-path="url(#clip980)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="736.477,1486.45 736.477,1467.55 "/>
<polyline clip-path="url(#clip980)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="1254.52,1486.45 1254.52,1467.55 "/>
<polyline clip-path="url(#clip980)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="1772.55,1486.45 1772.55,1467.55 "/>
<polyline clip-path="url(#clip980)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="2290.59,1486.45 2290.59,1467.55 "/>
<path clip-path="url(#clip980)" d="M218.439 1517.37 Q214.828 1517.37 212.999 1520.93 Q211.193 1524.47 211.193 1531.6 Q211.193 1538.71 212.999 1542.27 Q214.828 1545.82 218.439 1545.82 Q222.073 1545.82 223.879 1542.27 Q225.707 1538.71 225.707 1531.6 Q225.707 1524.47 223.879 1520.93 Q222.073 1517.37 218.439 1517.37 M218.439 1513.66 Q224.249 1513.66 227.304 1518.27 Q230.383 1522.85 230.383 1531.6 Q230.383 1540.33 227.304 1544.94 Q224.249 1549.52 218.439 1549.52 Q212.629 1549.52 209.55 1544.94 Q206.494 1540.33 206.494 1531.6 Q206.494 1522.85 209.55 1518.27 Q212.629 1513.66 218.439 1513.66 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip980)" d="M731.13 1544.91 L747.449 1544.91 L747.449 1548.85 L725.505 1548.85 L725.505 1544.91 Q728.167 1542.16 732.75 1537.53 Q737.357 1532.88 738.537 1531.53 Q740.782 1529.01 741.662 1527.27 Q742.565 1525.51 742.565 1523.82 Q742.565 1521.07 740.62 1519.33 Q738.699 1517.6 735.597 1517.6 Q733.398 1517.6 730.945 1518.36 Q728.514 1519.13 725.736 1520.68 L725.736 1515.95 Q728.56 1514.82 731.014 1514.24 Q733.468 1513.66 735.505 1513.66 Q740.875 1513.66 744.069 1516.35 Q747.264 1519.03 747.264 1523.52 Q747.264 1525.65 746.454 1527.57 Q745.667 1529.47 743.56 1532.07 Q742.981 1532.74 739.88 1535.95 Q736.778 1539.15 731.13 1544.91 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip980)" d="M1257.52 1518.36 L1245.72 1536.81 L1257.52 1536.81 L1257.52 1518.36 M1256.3 1514.29 L1262.18 1514.29 L1262.18 1536.81 L1267.11 1536.81 L1267.11 1540.7 L1262.18 1540.7 L1262.18 1548.85 L1257.52 1548.85 L1257.52 1540.7 L1241.92 1540.7 L1241.92 1536.19 L1256.3 1514.29 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip980)" d="M1772.96 1529.7 Q1769.81 1529.7 1767.96 1531.86 Q1766.13 1534.01 1766.13 1537.76 Q1766.13 1541.49 1767.96 1543.66 Q1769.81 1545.82 1772.96 1545.82 Q1776.11 1545.82 1777.94 1543.66 Q1779.79 1541.49 1779.79 1537.76 Q1779.79 1534.01 1777.94 1531.86 Q1776.11 1529.7 1772.96 1529.7 M1782.24 1515.05 L1782.24 1519.31 Q1780.48 1518.48 1778.68 1518.04 Q1776.89 1517.6 1775.13 1517.6 Q1770.5 1517.6 1768.05 1520.72 Q1765.62 1523.85 1765.27 1530.17 Q1766.64 1528.15 1768.7 1527.09 Q1770.76 1526 1773.24 1526 Q1778.44 1526 1781.45 1529.17 Q1784.49 1532.32 1784.49 1537.76 Q1784.49 1543.08 1781.34 1546.3 Q1778.19 1549.52 1772.96 1549.52 Q1766.96 1549.52 1763.79 1544.94 Q1760.62 1540.33 1760.62 1531.6 Q1760.62 1523.41 1764.51 1518.55 Q1768.4 1513.66 1774.95 1513.66 Q1776.71 1513.66 1778.49 1514.01 Q1780.3 1514.36 1782.24 1515.05 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip980)" d="M2290.59 1532.44 Q2287.26 1532.44 2285.34 1534.22 Q2283.44 1536 2283.44 1539.13 Q2283.44 1542.25 2285.34 1544.03 Q2287.26 1545.82 2290.59 1545.82 Q2293.92 1545.82 2295.85 1544.03 Q2297.77 1542.23 2297.77 1539.13 Q2297.77 1536 2295.85 1534.22 Q2293.95 1532.44 2290.59 1532.44 M2285.92 1530.45 Q2282.91 1529.7 2281.22 1527.64 Q2279.55 1525.58 2279.55 1522.62 Q2279.55 1518.48 2282.49 1516.07 Q2285.45 1513.66 2290.59 1513.66 Q2295.75 1513.66 2298.69 1516.07 Q2301.63 1518.48 2301.63 1522.62 Q2301.63 1525.58 2299.94 1527.64 Q2298.28 1529.7 2295.29 1530.45 Q2298.67 1531.23 2300.54 1533.52 Q2302.44 1535.82 2302.44 1539.13 Q2302.44 1544.15 2299.36 1546.83 Q2296.31 1549.52 2290.59 1549.52 Q2284.87 1549.52 2281.8 1546.83 Q2278.74 1544.15 2278.74 1539.13 Q2278.74 1535.82 2280.64 1533.52 Q2282.54 1531.23 2285.92 1530.45 M2284.2 1523.06 Q2284.2 1525.75 2285.87 1527.25 Q2287.56 1528.76 2290.59 1528.76 Q2293.6 1528.76 2295.29 1527.25 Q2297 1525.75 2297 1523.06 Q2297 1520.38 2295.29 1518.87 Q2293.6 1517.37 2290.59 1517.37 Q2287.56 1517.37 2285.87 1518.87 Q2284.2 1520.38 2284.2 1523.06 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><polyline clip-path="url(#clip980)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="156.274,1486.45 156.274,47.2441 "/>
<polyline clip-path="url(#clip980)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="156.274,1327.97 175.172,1327.97 "/>
<polyline clip-path="url(#clip980)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="156.274,1027.52 175.172,1027.52 "/>
<polyline clip-path="url(#clip980)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="156.274,727.065 175.172,727.065 "/>
<polyline clip-path="url(#clip980)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="156.274,426.615 175.172,426.615 "/>
<polyline clip-path="url(#clip980)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="156.274,126.164 175.172,126.164 "/>
<path clip-path="url(#clip980)" d="M62.9365 1313.76 Q59.3254 1313.76 57.4967 1317.33 Q55.6912 1320.87 55.6912 1328 Q55.6912 1335.11 57.4967 1338.67 Q59.3254 1342.21 62.9365 1342.21 Q66.5707 1342.21 68.3763 1338.67 Q70.205 1335.11 70.205 1328 Q70.205 1320.87 68.3763 1317.33 Q66.5707 1313.76 62.9365 1313.76 M62.9365 1310.06 Q68.7467 1310.06 71.8022 1314.67 Q74.8809 1319.25 74.8809 1328 Q74.8809 1336.73 71.8022 1341.33 Q68.7467 1345.92 62.9365 1345.92 Q57.1264 1345.92 54.0477 1341.33 Q50.9921 1336.73 50.9921 1328 Q50.9921 1319.25 54.0477 1314.67 Q57.1264 1310.06 62.9365 1310.06 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip980)" d="M83.0984 1339.37 L87.9827 1339.37 L87.9827 1345.25 L83.0984 1345.25 L83.0984 1339.37 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip980)" d="M108.746 1326.1 Q105.598 1326.1 103.746 1328.26 Q101.918 1330.41 101.918 1334.16 Q101.918 1337.88 103.746 1340.06 Q105.598 1342.21 108.746 1342.21 Q111.895 1342.21 113.723 1340.06 Q115.575 1337.88 115.575 1334.16 Q115.575 1330.41 113.723 1328.26 Q111.895 1326.1 108.746 1326.1 M118.029 1311.45 L118.029 1315.71 Q116.27 1314.88 114.464 1314.44 Q112.682 1314 110.922 1314 Q106.293 1314 103.839 1317.12 Q101.409 1320.25 101.061 1326.57 Q102.427 1324.55 104.487 1323.49 Q106.547 1322.4 109.024 1322.4 Q114.233 1322.4 117.242 1325.57 Q120.274 1328.72 120.274 1334.16 Q120.274 1339.48 117.126 1342.7 Q113.978 1345.92 108.746 1345.92 Q102.751 1345.92 99.5798 1341.33 Q96.4085 1336.73 96.4085 1328 Q96.4085 1319.81 100.297 1314.95 Q104.186 1310.06 110.737 1310.06 Q112.496 1310.06 114.279 1310.41 Q116.084 1310.76 118.029 1311.45 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip980)" d="M63.1911 1013.31 Q59.58 1013.31 57.7513 1016.88 Q55.9458 1020.42 55.9458 1027.55 Q55.9458 1034.66 57.7513 1038.22 Q59.58 1041.76 63.1911 1041.76 Q66.8254 1041.76 68.6309 1038.22 Q70.4596 1034.66 70.4596 1027.55 Q70.4596 1020.42 68.6309 1016.88 Q66.8254 1013.31 63.1911 1013.31 M63.1911 1009.61 Q69.0013 1009.61 72.0568 1014.22 Q75.1355 1018.8 75.1355 1027.55 Q75.1355 1036.28 72.0568 1040.88 Q69.0013 1045.47 63.1911 1045.47 Q57.381 1045.47 54.3023 1040.88 Q51.2468 1036.28 51.2468 1027.55 Q51.2468 1018.8 54.3023 1014.22 Q57.381 1009.61 63.1911 1009.61 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip980)" d="M83.3531 1038.92 L88.2373 1038.92 L88.2373 1044.8 L83.3531 1044.8 L83.3531 1038.92 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip980)" d="M108.422 1028.38 Q105.089 1028.38 103.168 1030.17 Q101.27 1031.95 101.27 1035.07 Q101.27 1038.2 103.168 1039.98 Q105.089 1041.76 108.422 1041.76 Q111.756 1041.76 113.677 1039.98 Q115.598 1038.18 115.598 1035.07 Q115.598 1031.95 113.677 1030.17 Q111.779 1028.38 108.422 1028.38 M103.746 1026.39 Q100.737 1025.65 99.0474 1023.59 Q97.3808 1021.53 97.3808 1018.57 Q97.3808 1014.43 100.321 1012.02 Q103.284 1009.61 108.422 1009.61 Q113.584 1009.61 116.524 1012.02 Q119.464 1014.43 119.464 1018.57 Q119.464 1021.53 117.774 1023.59 Q116.108 1025.65 113.121 1026.39 Q116.501 1027.18 118.376 1029.47 Q120.274 1031.76 120.274 1035.07 Q120.274 1040.1 117.195 1042.78 Q114.14 1045.47 108.422 1045.47 Q102.705 1045.47 99.6261 1042.78 Q96.5706 1040.1 96.5706 1035.07 Q96.5706 1031.76 98.4687 1029.47 Q100.367 1027.18 103.746 1026.39 M102.034 1019.01 Q102.034 1021.69 103.7 1023.2 Q105.39 1024.7 108.422 1024.7 Q111.432 1024.7 113.121 1023.2 Q114.834 1021.69 114.834 1019.01 Q114.834 1016.32 113.121 1014.82 Q111.432 1013.31 108.422 1013.31 Q105.39 1013.31 103.7 1014.82 Q102.034 1016.32 102.034 1019.01 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip980)" d="M53.9088 740.41 L61.5476 740.41 L61.5476 714.044 L53.2375 715.711 L53.2375 711.452 L61.5013 709.785 L66.1772 709.785 L66.1772 740.41 L73.8161 740.41 L73.8161 744.345 L53.9088 744.345 L53.9088 740.41 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip980)" d="M83.2605 738.465 L88.1447 738.465 L88.1447 744.345 L83.2605 744.345 L83.2605 738.465 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip980)" d="M108.33 712.864 Q104.719 712.864 102.89 716.428 Q101.084 719.97 101.084 727.1 Q101.084 734.206 102.89 737.771 Q104.719 741.313 108.33 741.313 Q111.964 741.313 113.77 737.771 Q115.598 734.206 115.598 727.1 Q115.598 719.97 113.77 716.428 Q111.964 712.864 108.33 712.864 M108.33 709.16 Q114.14 709.16 117.195 713.766 Q120.274 718.35 120.274 727.1 Q120.274 735.827 117.195 740.433 Q114.14 745.016 108.33 745.016 Q102.52 745.016 99.4409 740.433 Q96.3854 735.827 96.3854 727.1 Q96.3854 718.35 99.4409 713.766 Q102.52 709.16 108.33 709.16 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip980)" d="M55.506 439.959 L63.1448 439.959 L63.1448 413.594 L54.8347 415.26 L54.8347 411.001 L63.0985 409.335 L67.7744 409.335 L67.7744 439.959 L75.4133 439.959 L75.4133 443.895 L55.506 443.895 L55.506 439.959 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip980)" d="M84.8577 438.015 L89.7419 438.015 L89.7419 443.895 L84.8577 443.895 L84.8577 438.015 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip980)" d="M103.955 439.959 L120.274 439.959 L120.274 443.895 L98.3298 443.895 L98.3298 439.959 Q100.992 437.205 105.575 432.575 Q110.182 427.922 111.362 426.58 Q113.608 424.057 114.487 422.321 Q115.39 420.561 115.39 418.871 Q115.39 416.117 113.445 414.381 Q111.524 412.645 108.422 412.645 Q106.223 412.645 103.77 413.409 Q101.339 414.172 98.5613 415.723 L98.5613 411.001 Q101.385 409.867 103.839 409.288 Q106.293 408.71 108.33 408.71 Q113.7 408.71 116.895 411.395 Q120.089 414.08 120.089 418.571 Q120.089 420.7 119.279 422.621 Q118.492 424.52 116.385 427.112 Q115.807 427.783 112.705 431.001 Q109.603 434.195 103.955 439.959 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip980)" d="M53.4227 139.509 L61.0615 139.509 L61.0615 113.143 L52.7514 114.81 L52.7514 110.551 L61.0152 108.884 L65.6911 108.884 L65.6911 139.509 L73.33 139.509 L73.33 143.444 L53.4227 143.444 L53.4227 139.509 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip980)" d="M82.7744 137.564 L87.6586 137.564 L87.6586 143.444 L82.7744 143.444 L82.7744 137.564 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip980)" d="M110.691 112.958 L98.8854 131.407 L110.691 131.407 L110.691 112.958 M109.464 108.884 L115.344 108.884 L115.344 131.407 L120.274 131.407 L120.274 135.296 L115.344 135.296 L115.344 143.444 L110.691 143.444 L110.691 135.296 L95.0891 135.296 L95.0891 130.782 L109.464 108.884 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><polyline clip-path="url(#clip982)" style="stroke:#009af9; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="218.439,1445.72 244.341,1349.39 270.243,1197.15 296.144,1003.61 322.046,787.832 347.948,571.242 373.85,375.335 399.752,219.638 425.654,120.048 451.556,87.9763 477.458,118.029 503.36,200.796 529.262,329.644 555.164,492.422 581.065,673.144 606.967,853.949 632.869,1016.84 658.771,1145.46 684.673,1226.42 710.575,1250.14 736.477,1224.25 762.379,1157.13 788.281,1053.92 814.183,924.157 840.085,780.439 865.986,636.952 891.888,508.012 917.79,406.711 943.692,343.857 969.594,327.128 995.496,346.838 1021.4,395.564 1047.3,469.83 1073.2,563.073 1099.1,666.381 1125.01,769.601 1150.91,862.385 1176.81,935.14 1202.71,979.811 1228.61,990.827 1254.52,979.774 1280.42,953.314 1306.32,913.092 1332.22,862.418 1358.12,805.928 1384.02,749.089 1409.93,697.654 1435.83,657.115 1461.73,632.214 1487.63,626.035 1513.53,626.035 1539.44,626.035 1565.34,626.035 1591.24,626.035 1617.14,626.035 1643.04,626.035 1668.95,626.035 1694.85,626.035 1720.75,626.035 1746.65,626.035 1772.55,626.035 1798.46,626.035 1824.36,626.035 1850.26,626.035 1876.16,626.035 1902.06,626.035 1927.96,626.035 1953.87,626.035 1979.77,626.035 2005.67,626.035 2031.57,626.035 2057.47,626.035 2083.38,626.035 2109.28,626.035 2135.18,626.035 2161.08,626.035 2186.98,626.035 2212.89,626.035 2238.79,626.035 2264.69,626.035 2290.59,626.035 "/>
<path clip-path="url(#clip980)" d="M2007.46 198.898 L2279.54 198.898 L2279.54 95.2176 L2007.46 95.2176  Z" fill="#ffffff" fill-rule="evenodd" fill-opacity="1"/>
<polyline clip-path="url(#clip980)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="2007.46,198.898 2279.54,198.898 2279.54,95.2176 2007.46,95.2176 2007.46,198.898 "/>
<polyline clip-path="url(#clip980)" style="stroke:#009af9; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="2031.87,147.058 2178.3,147.058 "/>
<path clip-path="url(#clip980)" d="M2216.55 166.745 Q2214.74 171.375 2213.03 172.787 Q2211.32 174.199 2208.44 174.199 L2205.04 174.199 L2205.04 170.634 L2207.54 170.634 Q2209.3 170.634 2210.27 169.8 Q2211.25 168.967 2212.43 165.865 L2213.19 163.921 L2202.7 138.412 L2207.22 138.412 L2215.32 158.689 L2223.42 138.412 L2227.94 138.412 L2216.55 166.745 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip980)" d="M2235.23 160.402 L2242.87 160.402 L2242.87 134.037 L2234.56 135.703 L2234.56 131.444 L2242.82 129.778 L2247.5 129.778 L2247.5 160.402 L2255.13 160.402 L2255.13 164.338 L2235.23 164.338 L2235.23 160.402 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /></svg>




The instantiated FMU must be terminated and then the memory area for the instance can also be deallocated. The last step is to unload the FMU to remove all unpacked data on disc. 


```julia
fmi2Terminate(instanceFMU)
fmi2FreeInstance!(instanceFMU)
unloadFMU(fmu)
```

### Summary

The tutorial has shown how to use the default simulation command and how to deploy a custom simulation loop.