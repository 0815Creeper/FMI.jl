# FMU Parameter Optimization
Tutorial by Tobias Thummerer

## License


```julia
# Copyright (c) 2021 Tobias Thummerer, Lars Mikelsons
# Licensed under the MIT license. 
# See LICENSE (https://github.com/thummeto/FMI.jl/blob/main/LICENSE) file in the project root for details.
```

## Introduction to the example
This example shows how a parameter optimization can be set up for a FMU. The goal is to fit FMU parameters (and initial states), so that a reference trajectory is fit as good as possible.

Note, that this tutorial covers optimization *without* gradient information. Basically, *FMI.jl* supports gradient based optimization, too.

## Other formats
Besides, this [Jupyter Notebook](https://github.com/thummeto/FMI.jl/blob/examples/examples/src/parameter_optimization.ipynb) there is also a [Julia file](https://github.com/thummeto/FMI.jl/blob/examples/examples/src/parameter_optimization.jl) with the same name, which contains only the code cells and for the documentation there is a [Markdown file](https://github.com/thummeto/FMI.jl/blob/examples/examples/src/parameter_optimization.md) corresponding to the notebook.  

## Getting started

### Installation prerequisites
|     | Description                       | Command                   |
|:----|:----------------------------------|:--------------------------|
| 1.  | Enter Package Manager via         | ]                         |
| 2.  | Install FMI via                   | add FMI                   | 
| 3.  | Install FMIZoo via                | add FMIZoo                | 
| 4.  | Install Optim  via                | add Optim                 | 
| 5.  | Install Plots  via                | add Plots                 | 

## Code section

To run the example, the previously installed packages must be included. 


```julia
# imports
using FMI
using FMIZoo
using Optim
using Plots
using DifferentialEquations
```

### Simulation setup

Next, the start time and end time of the simulation are set.


```julia
tStart = 0.0
tStop = 5.0
tStep = 0.1
tSave = tStart:tStep:tStop
```




    0.0:0.1:5.0



### Import FMU

In the next lines of code the FMU model from *FMIZoo.jl* is loaded and the information about the FMU is shown.


```julia
# we use an FMU from the FMIZoo.jl
fmu = loadFMU("SpringPendulum1D", "Dymola", "2022x"; type=:ME)
info(fmu)
```

    #################### Begin information for FMU ####################
    	Model name:			SpringPendulum1D
    	FMI-Version:			2.0
    	GUID:				{fc15d8c4-758b-48e6-b00e-5bf47b8b14e5}
    	Generation tool:		Dymola Version 2022x (64-bit), 2021-10-08
    	Generation time:		2022-05-19T06:54:23Z
    	Var. naming conv.:		structured
    	Event indicators:		0
    	Inputs:				0
    	Outputs:			0
    	States:				2
    

    		33554432 ["mass.s"]
    		33554433 ["mass.v"]
    	Parameters:			7
    		16777216 ["mass_s0"]
    		16777217 ["mass_v0"]
    		16777218 ["fixed.s0"]
    		16777219 ["spring.c"]
    		16777220 ["spring.s_rel0"]
    		16777221 ["mass.m"]
    		16777222 ["mass.L"]
    	Supports Co-Simulation:		true
    		Model identifier:	SpringPendulum1D
    		Get/Set State:		true
    		Serialize State:	true
    		Dir. Derivatives:	true
    		Var. com. steps:	true
    		Input interpol.:	true
    		Max order out. der.:	1
    	Supports Model-Exchange:	true
    		Model identifier:	SpringPendulum1D
    		Get/Set State:		true
    		Serialize State:	true
    		Dir. Derivatives:	true
    ##################### End information for FMU #####################
    

Now, the optimization objective (the function to minimize) needs to be defined. In this case, we just want to do a simulation and compare it to a regular `sin` wave.


```julia
s_tar = 1.0 .+ sin.(tSave)

# a function to simulate the FMU for given parameters
function simulateFMU(p)
    s0, v0, c, m = p # unpack parameters: s0 (start position), v0 (start velocity), c (spring constant) and m (pendulum mass)

    # pack the parameters into a dictionary
    paramDict = Dict{String, Any}()
    paramDict["spring.c"] = c 
    paramDict["mass.m"] = m

    # pack the start state
    x0 = [s0, v0]

    # simulate with given start stae and parameters
    sol = simulate(fmu, (tStart, tStop); x0=x0, parameters=paramDict, saveat=tSave)

    # get state with index 1 (the position) from the solution
    s_res = getState(sol, 1; isIndex=true) 

    return s_res
end

# the optimization objective
function objective(p)
    s_res = simulateFMU(p)

    # return the position error sum between FMU simulation (s_res) and target (s_tar)
    return sum(abs.(s_tar .- s_res))    
end
```




    objective (generic function with 1 method)



Now let's see how far we are away for our guess parameters:


```julia
s0 = 0.0 
v0 = 0.0
c = 1.0
m = 1.0 
p = [s0, v0, c, m]

obj_before = objective(p) # not really good!
```




    54.432324541060666



Let's have a look on the differences:


```julia
s_fmu = simulateFMU(p); # simulate the position

plot(tSave, s_fmu; label="FMU")
plot!(tSave, s_tar; label="Optimization target")
```




<?xml version="1.0" encoding="utf-8"?>
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="600" height="400" viewBox="0 0 2400 1600">
<defs>
  <clipPath id="clip780">
    <rect x="0" y="0" width="2400" height="1600"/>
  </clipPath>
</defs>
<path clip-path="url(#clip780)" d="M0 1600 L2400 1600 L2400 0 L0 0  Z" fill="#ffffff" fill-rule="evenodd" fill-opacity="1"/>
<defs>
  <clipPath id="clip781">
    <rect x="480" y="0" width="1681" height="1600"/>
  </clipPath>
</defs>
<path clip-path="url(#clip780)" d="M156.112 1486.45 L2352.76 1486.45 L2352.76 47.2441 L156.112 47.2441  Z" fill="#ffffff" fill-rule="evenodd" fill-opacity="1"/>
<defs>
  <clipPath id="clip782">
    <rect x="156" y="47" width="2198" height="1440"/>
  </clipPath>
</defs>
<polyline clip-path="url(#clip782)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:2; stroke-opacity:0.1; fill:none" points="218.281,1486.45 218.281,47.2441 "/>
<polyline clip-path="url(#clip782)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:2; stroke-opacity:0.1; fill:none" points="632.742,1486.45 632.742,47.2441 "/>
<polyline clip-path="url(#clip782)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:2; stroke-opacity:0.1; fill:none" points="1047.2,1486.45 1047.2,47.2441 "/>
<polyline clip-path="url(#clip782)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:2; stroke-opacity:0.1; fill:none" points="1461.66,1486.45 1461.66,47.2441 "/>
<polyline clip-path="url(#clip782)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:2; stroke-opacity:0.1; fill:none" points="1876.13,1486.45 1876.13,47.2441 "/>
<polyline clip-path="url(#clip782)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:2; stroke-opacity:0.1; fill:none" points="2290.59,1486.45 2290.59,47.2441 "/>
<polyline clip-path="url(#clip782)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:2; stroke-opacity:0.1; fill:none" points="156.112,1445.72 2352.76,1445.72 "/>
<polyline clip-path="url(#clip782)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:2; stroke-opacity:0.1; fill:none" points="156.112,1137.01 2352.76,1137.01 "/>
<polyline clip-path="url(#clip782)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:2; stroke-opacity:0.1; fill:none" points="156.112,828.294 2352.76,828.294 "/>
<polyline clip-path="url(#clip782)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:2; stroke-opacity:0.1; fill:none" points="156.112,519.584 2352.76,519.584 "/>
<polyline clip-path="url(#clip782)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:2; stroke-opacity:0.1; fill:none" points="156.112,210.873 2352.76,210.873 "/>
<polyline clip-path="url(#clip780)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="156.112,1486.45 2352.76,1486.45 "/>
<polyline clip-path="url(#clip780)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="218.281,1486.45 218.281,1467.55 "/>
<polyline clip-path="url(#clip780)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="632.742,1486.45 632.742,1467.55 "/>
<polyline clip-path="url(#clip780)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="1047.2,1486.45 1047.2,1467.55 "/>
<polyline clip-path="url(#clip780)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="1461.66,1486.45 1461.66,1467.55 "/>
<polyline clip-path="url(#clip780)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="1876.13,1486.45 1876.13,1467.55 "/>
<polyline clip-path="url(#clip780)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="2290.59,1486.45 2290.59,1467.55 "/>
<path clip-path="url(#clip780)" d="M218.281 1517.37 Q214.67 1517.37 212.842 1520.93 Q211.036 1524.47 211.036 1531.6 Q211.036 1538.71 212.842 1542.27 Q214.67 1545.82 218.281 1545.82 Q221.916 1545.82 223.721 1542.27 Q225.55 1538.71 225.55 1531.6 Q225.55 1524.47 223.721 1520.93 Q221.916 1517.37 218.281 1517.37 M218.281 1513.66 Q224.091 1513.66 227.147 1518.27 Q230.226 1522.85 230.226 1531.6 Q230.226 1540.33 227.147 1544.94 Q224.091 1549.52 218.281 1549.52 Q212.471 1549.52 209.392 1544.94 Q206.337 1540.33 206.337 1531.6 Q206.337 1522.85 209.392 1518.27 Q212.471 1513.66 218.281 1513.66 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip780)" d="M623.124 1544.91 L630.763 1544.91 L630.763 1518.55 L622.453 1520.21 L622.453 1515.95 L630.717 1514.29 L635.393 1514.29 L635.393 1544.91 L643.032 1544.91 L643.032 1548.85 L623.124 1548.85 L623.124 1544.91 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip780)" d="M1041.86 1544.91 L1058.18 1544.91 L1058.18 1548.85 L1036.23 1548.85 L1036.23 1544.91 Q1038.89 1542.16 1043.48 1537.53 Q1048.08 1532.88 1049.26 1531.53 Q1051.51 1529.01 1052.39 1527.27 Q1053.29 1525.51 1053.29 1523.82 Q1053.29 1521.07 1051.35 1519.33 Q1049.43 1517.6 1046.32 1517.6 Q1044.12 1517.6 1041.67 1518.36 Q1039.24 1519.13 1036.46 1520.68 L1036.46 1515.95 Q1039.29 1514.82 1041.74 1514.24 Q1044.19 1513.66 1046.23 1513.66 Q1051.6 1513.66 1054.8 1516.35 Q1057.99 1519.03 1057.99 1523.52 Q1057.99 1525.65 1057.18 1527.57 Q1056.39 1529.47 1054.29 1532.07 Q1053.71 1532.74 1050.61 1535.95 Q1047.5 1539.15 1041.86 1544.91 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip780)" d="M1465.91 1530.21 Q1469.27 1530.93 1471.14 1533.2 Q1473.04 1535.47 1473.04 1538.8 Q1473.04 1543.92 1469.52 1546.72 Q1466 1549.52 1459.52 1549.52 Q1457.35 1549.52 1455.03 1549.08 Q1452.74 1548.66 1450.29 1547.81 L1450.29 1543.29 Q1452.23 1544.43 1454.55 1545.01 Q1456.86 1545.58 1459.38 1545.58 Q1463.78 1545.58 1466.07 1543.85 Q1468.39 1542.11 1468.39 1538.8 Q1468.39 1535.75 1466.24 1534.03 Q1464.11 1532.3 1460.29 1532.3 L1456.26 1532.3 L1456.26 1528.45 L1460.47 1528.45 Q1463.92 1528.45 1465.75 1527.09 Q1467.58 1525.7 1467.58 1523.11 Q1467.58 1520.45 1465.68 1519.03 Q1463.81 1517.6 1460.29 1517.6 Q1458.37 1517.6 1456.17 1518.01 Q1453.97 1518.43 1451.33 1519.31 L1451.33 1515.14 Q1453.99 1514.4 1456.31 1514.03 Q1458.64 1513.66 1460.7 1513.66 Q1466.03 1513.66 1469.13 1516.09 Q1472.23 1518.5 1472.23 1522.62 Q1472.23 1525.49 1470.59 1527.48 Q1468.94 1529.45 1465.91 1530.21 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip780)" d="M1879.13 1518.36 L1867.33 1536.81 L1879.13 1536.81 L1879.13 1518.36 M1877.91 1514.29 L1883.79 1514.29 L1883.79 1536.81 L1888.72 1536.81 L1888.72 1540.7 L1883.79 1540.7 L1883.79 1548.85 L1879.13 1548.85 L1879.13 1540.7 L1863.53 1540.7 L1863.53 1536.19 L1877.91 1514.29 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip780)" d="M2280.86 1514.29 L2299.22 1514.29 L2299.22 1518.22 L2285.15 1518.22 L2285.15 1526.7 Q2286.17 1526.35 2287.18 1526.19 Q2288.2 1526 2289.22 1526 Q2295.01 1526 2298.39 1529.17 Q2301.77 1532.34 2301.77 1537.76 Q2301.77 1543.34 2298.3 1546.44 Q2294.82 1549.52 2288.5 1549.52 Q2286.33 1549.52 2284.06 1549.15 Q2281.81 1548.78 2279.41 1548.04 L2279.41 1543.34 Q2281.49 1544.47 2283.71 1545.03 Q2285.93 1545.58 2288.41 1545.58 Q2292.42 1545.58 2294.75 1543.48 Q2297.09 1541.37 2297.09 1537.76 Q2297.09 1534.15 2294.75 1532.04 Q2292.42 1529.94 2288.41 1529.94 Q2286.54 1529.94 2284.66 1530.35 Q2282.81 1530.77 2280.86 1531.65 L2280.86 1514.29 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><polyline clip-path="url(#clip780)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="156.112,1486.45 156.112,47.2441 "/>
<polyline clip-path="url(#clip780)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="156.112,1445.72 175.01,1445.72 "/>
<polyline clip-path="url(#clip780)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="156.112,1137.01 175.01,1137.01 "/>
<polyline clip-path="url(#clip780)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="156.112,828.294 175.01,828.294 "/>
<polyline clip-path="url(#clip780)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="156.112,519.584 175.01,519.584 "/>
<polyline clip-path="url(#clip780)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="156.112,210.873 175.01,210.873 "/>
<path clip-path="url(#clip780)" d="M62.9365 1431.51 Q59.3254 1431.51 57.4967 1435.08 Q55.6912 1438.62 55.6912 1445.75 Q55.6912 1452.86 57.4967 1456.42 Q59.3254 1459.96 62.9365 1459.96 Q66.5707 1459.96 68.3763 1456.42 Q70.205 1452.86 70.205 1445.75 Q70.205 1438.62 68.3763 1435.08 Q66.5707 1431.51 62.9365 1431.51 M62.9365 1427.81 Q68.7467 1427.81 71.8022 1432.42 Q74.8809 1437 74.8809 1445.75 Q74.8809 1454.48 71.8022 1459.08 Q68.7467 1463.67 62.9365 1463.67 Q57.1264 1463.67 54.0477 1459.08 Q50.9921 1454.48 50.9921 1445.75 Q50.9921 1437 54.0477 1432.42 Q57.1264 1427.81 62.9365 1427.81 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip780)" d="M83.0984 1457.12 L87.9827 1457.12 L87.9827 1463 L83.0984 1463 L83.0984 1457.12 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip780)" d="M108.168 1431.51 Q104.557 1431.51 102.728 1435.08 Q100.922 1438.62 100.922 1445.75 Q100.922 1452.86 102.728 1456.42 Q104.557 1459.96 108.168 1459.96 Q111.802 1459.96 113.608 1456.42 Q115.436 1452.86 115.436 1445.75 Q115.436 1438.62 113.608 1435.08 Q111.802 1431.51 108.168 1431.51 M108.168 1427.81 Q113.978 1427.81 117.033 1432.42 Q120.112 1437 120.112 1445.75 Q120.112 1454.48 117.033 1459.08 Q113.978 1463.67 108.168 1463.67 Q102.358 1463.67 99.2789 1459.08 Q96.2234 1454.48 96.2234 1445.75 Q96.2234 1437 99.2789 1432.42 Q102.358 1427.81 108.168 1427.81 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip780)" d="M63.9319 1122.8 Q60.3208 1122.8 58.4921 1126.37 Q56.6865 1129.91 56.6865 1137.04 Q56.6865 1144.15 58.4921 1147.71 Q60.3208 1151.25 63.9319 1151.25 Q67.5661 1151.25 69.3717 1147.71 Q71.2004 1144.15 71.2004 1137.04 Q71.2004 1129.91 69.3717 1126.37 Q67.5661 1122.8 63.9319 1122.8 M63.9319 1119.1 Q69.742 1119.1 72.7976 1123.71 Q75.8763 1128.29 75.8763 1137.04 Q75.8763 1145.77 72.7976 1150.37 Q69.742 1154.96 63.9319 1154.96 Q58.1217 1154.96 55.043 1150.37 Q51.9875 1145.77 51.9875 1137.04 Q51.9875 1128.29 55.043 1123.71 Q58.1217 1119.1 63.9319 1119.1 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip780)" d="M84.0938 1148.41 L88.978 1148.41 L88.978 1154.29 L84.0938 1154.29 L84.0938 1148.41 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip780)" d="M99.2095 1119.73 L117.566 1119.73 L117.566 1123.66 L103.492 1123.66 L103.492 1132.13 Q104.51 1131.79 105.529 1131.62 Q106.547 1131.44 107.566 1131.44 Q113.353 1131.44 116.733 1134.61 Q120.112 1137.78 120.112 1143.2 Q120.112 1148.78 116.64 1151.88 Q113.168 1154.96 106.848 1154.96 Q104.672 1154.96 102.404 1154.59 Q100.159 1154.22 97.7511 1153.47 L97.7511 1148.78 Q99.8345 1149.91 102.057 1150.47 Q104.279 1151.02 106.756 1151.02 Q110.76 1151.02 113.098 1148.91 Q115.436 1146.81 115.436 1143.2 Q115.436 1139.59 113.098 1137.48 Q110.76 1135.37 106.756 1135.37 Q104.881 1135.37 103.006 1135.79 Q101.154 1136.21 99.2095 1137.09 L99.2095 1119.73 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip780)" d="M53.7467 841.639 L61.3856 841.639 L61.3856 815.274 L53.0754 816.94 L53.0754 812.681 L61.3393 811.014 L66.0152 811.014 L66.0152 841.639 L73.654 841.639 L73.654 845.574 L53.7467 845.574 L53.7467 841.639 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip780)" d="M83.0984 839.695 L87.9827 839.695 L87.9827 845.574 L83.0984 845.574 L83.0984 839.695 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip780)" d="M108.168 814.093 Q104.557 814.093 102.728 817.658 Q100.922 821.199 100.922 828.329 Q100.922 835.436 102.728 839 Q104.557 842.542 108.168 842.542 Q111.802 842.542 113.608 839 Q115.436 835.436 115.436 828.329 Q115.436 821.199 113.608 817.658 Q111.802 814.093 108.168 814.093 M108.168 810.389 Q113.978 810.389 117.033 814.996 Q120.112 819.579 120.112 828.329 Q120.112 837.056 117.033 841.662 Q113.978 846.246 108.168 846.246 Q102.358 846.246 99.2789 841.662 Q96.2234 837.056 96.2234 828.329 Q96.2234 819.579 99.2789 814.996 Q102.358 810.389 108.168 810.389 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip780)" d="M54.7421 532.929 L62.381 532.929 L62.381 506.563 L54.0708 508.23 L54.0708 503.97 L62.3347 502.304 L67.0106 502.304 L67.0106 532.929 L74.6494 532.929 L74.6494 536.864 L54.7421 536.864 L54.7421 532.929 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip780)" d="M84.0938 530.984 L88.978 530.984 L88.978 536.864 L84.0938 536.864 L84.0938 530.984 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip780)" d="M99.2095 502.304 L117.566 502.304 L117.566 506.239 L103.492 506.239 L103.492 514.711 Q104.51 514.364 105.529 514.202 Q106.547 514.017 107.566 514.017 Q113.353 514.017 116.733 517.188 Q120.112 520.359 120.112 525.776 Q120.112 531.354 116.64 534.456 Q113.168 537.535 106.848 537.535 Q104.672 537.535 102.404 537.165 Q100.159 536.794 97.7511 536.054 L97.7511 531.354 Q99.8345 532.489 102.057 533.044 Q104.279 533.6 106.756 533.6 Q110.76 533.6 113.098 531.493 Q115.436 529.387 115.436 525.776 Q115.436 522.165 113.098 520.058 Q110.76 517.952 106.756 517.952 Q104.881 517.952 103.006 518.368 Q101.154 518.785 99.2095 519.665 L99.2095 502.304 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip780)" d="M56.9643 224.218 L73.2837 224.218 L73.2837 228.153 L51.3393 228.153 L51.3393 224.218 Q54.0014 221.463 58.5847 216.834 Q63.1911 212.181 64.3717 210.838 Q66.617 208.315 67.4967 206.579 Q68.3994 204.82 68.3994 203.13 Q68.3994 200.375 66.455 198.639 Q64.5337 196.903 61.4319 196.903 Q59.2328 196.903 56.7791 197.667 Q54.3486 198.431 51.5708 199.982 L51.5708 195.26 Q54.3949 194.125 56.8486 193.547 Q59.3023 192.968 61.3393 192.968 Q66.7096 192.968 69.9041 195.653 Q73.0985 198.338 73.0985 202.829 Q73.0985 204.959 72.2883 206.88 Q71.5013 208.778 69.3948 211.371 Q68.8161 212.042 65.7143 215.26 Q62.6124 218.454 56.9643 224.218 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip780)" d="M83.0984 222.273 L87.9827 222.273 L87.9827 228.153 L83.0984 228.153 L83.0984 222.273 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip780)" d="M108.168 196.672 Q104.557 196.672 102.728 200.237 Q100.922 203.778 100.922 210.908 Q100.922 218.014 102.728 221.579 Q104.557 225.121 108.168 225.121 Q111.802 225.121 113.608 221.579 Q115.436 218.014 115.436 210.908 Q115.436 203.778 113.608 200.237 Q111.802 196.672 108.168 196.672 M108.168 192.968 Q113.978 192.968 117.033 197.575 Q120.112 202.158 120.112 210.908 Q120.112 219.635 117.033 224.241 Q113.978 228.824 108.168 228.824 Q102.358 228.824 99.2789 224.241 Q96.2234 219.635 96.2234 210.908 Q96.2234 202.158 99.2789 197.575 Q102.358 192.968 108.168 192.968 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><polyline clip-path="url(#clip782)" style="stroke:#009af9; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="218.281,1445.72 259.727,1442.32 301.174,1432.18 342.62,1415.38 384.066,1392.1 425.512,1362.58 466.958,1327.09 508.404,1286.01 549.85,1239.73 591.296,1188.73 632.742,1133.51 674.188,1074.62 715.635,1012.66 757.081,948.231 798.527,881.991 839.973,814.597 881.419,746.724 922.865,679.049 964.311,612.248 1005.76,546.989 1047.2,483.923 1088.65,423.682 1130.1,366.866 1171.54,314.044 1212.99,265.743 1254.43,222.447 1295.88,184.586 1337.33,152.541 1378.77,126.63 1420.22,107.114 1461.66,94.1859 1503.11,87.9763 1544.56,88.5467 1586,95.8916 1627.45,109.937 1668.9,130.544 1710.34,157.505 1751.79,190.552 1793.23,229.354 1834.68,273.523 1876.13,322.619 1917.57,376.15 1959.02,433.582 2000.46,494.341 2041.91,557.82 2083.36,623.385 2124.8,690.379 2166.25,758.135 2207.69,825.975 2249.14,893.222 2290.59,959.202 "/>
<polyline clip-path="url(#clip782)" style="stroke:#e26f46; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="218.281,828.294 259.727,766.655 301.174,705.632 342.62,645.834 384.066,587.859 425.512,532.287 466.958,479.672 508.404,430.541 549.85,385.383 591.296,344.652 632.742,308.752 674.188,278.044 715.635,252.834 757.081,233.373 798.527,219.857 839.973,212.42 881.419,211.136 922.865,216.019 964.311,227.02 1005.76,244.029 1047.2,266.875 1088.65,295.33 1130.1,329.111 1171.54,367.88 1212.99,411.249 1254.43,458.785 1295.88,510.013 1337.33,564.421 1378.77,621.466 1420.22,680.577 1461.66,741.164 1503.11,802.622 1544.56,864.336 1586,925.69 1627.45,986.071 1668.9,1044.88 1710.34,1101.52 1751.79,1155.43 1793.23,1206.07 1834.68,1252.94 1876.13,1295.56 1917.57,1333.52 1959.02,1366.42 2000.46,1393.95 2041.91,1415.83 2083.36,1431.84 2124.8,1441.82 2166.25,1445.67 2207.69,1443.35 2249.14,1434.88 2290.59,1420.35 "/>
<path clip-path="url(#clip780)" d="M1601.19 250.738 L2279.53 250.738 L2279.53 95.2176 L1601.19 95.2176  Z" fill="#ffffff" fill-rule="evenodd" fill-opacity="1"/>
<polyline clip-path="url(#clip780)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="1601.19,250.738 2279.53,250.738 2279.53,95.2176 1601.19,95.2176 1601.19,250.738 "/>
<polyline clip-path="url(#clip780)" style="stroke:#009af9; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="1625.6,147.058 1772.04,147.058 "/>
<path clip-path="url(#clip780)" d="M1796.45 129.778 L1816.31 129.778 L1816.31 133.713 L1801.13 133.713 L1801.13 143.898 L1814.83 143.898 L1814.83 147.833 L1801.13 147.833 L1801.13 164.338 L1796.45 164.338 L1796.45 129.778 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip780)" d="M1823.72 129.778 L1830.69 129.778 L1839.5 153.296 L1848.37 129.778 L1855.34 129.778 L1855.34 164.338 L1850.78 164.338 L1850.78 133.99 L1841.87 157.694 L1837.17 157.694 L1828.25 133.99 L1828.25 164.338 L1823.72 164.338 L1823.72 129.778 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip780)" d="M1864.09 129.778 L1868.79 129.778 L1868.79 150.773 Q1868.79 156.328 1870.8 158.782 Q1872.81 161.213 1877.33 161.213 Q1881.82 161.213 1883.83 158.782 Q1885.85 156.328 1885.85 150.773 L1885.85 129.778 L1890.55 129.778 L1890.55 151.352 Q1890.55 158.111 1887.19 161.56 Q1883.86 165.009 1877.33 165.009 Q1870.78 165.009 1867.42 161.56 Q1864.09 158.111 1864.09 151.352 L1864.09 129.778 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><polyline clip-path="url(#clip780)" style="stroke:#e26f46; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="1625.6,198.898 1772.04,198.898 "/>
<path clip-path="url(#clip780)" d="M1812.47 184.789 Q1807.38 184.789 1804.37 188.585 Q1801.38 192.381 1801.38 198.932 Q1801.38 205.46 1804.37 209.256 Q1807.38 213.053 1812.47 213.053 Q1817.56 213.053 1820.52 209.256 Q1823.51 205.46 1823.51 198.932 Q1823.51 192.381 1820.52 188.585 Q1817.56 184.789 1812.47 184.789 M1812.47 180.993 Q1819.74 180.993 1824.09 185.877 Q1828.44 190.738 1828.44 198.932 Q1828.44 207.104 1824.09 211.988 Q1819.74 216.849 1812.47 216.849 Q1805.18 216.849 1800.8 211.988 Q1796.45 207.127 1796.45 198.932 Q1796.45 190.738 1800.8 185.877 Q1805.18 180.993 1812.47 180.993 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip780)" d="M1839.69 212.289 L1839.69 226.039 L1835.41 226.039 L1835.41 190.252 L1839.69 190.252 L1839.69 194.187 Q1841.03 191.872 1843.07 190.761 Q1845.13 189.627 1847.98 189.627 Q1852.7 189.627 1855.64 193.377 Q1858.6 197.127 1858.6 203.238 Q1858.6 209.349 1855.64 213.099 Q1852.7 216.849 1847.98 216.849 Q1845.13 216.849 1843.07 215.738 Q1841.03 214.603 1839.69 212.289 M1854.18 203.238 Q1854.18 198.539 1852.24 195.877 Q1850.31 193.192 1846.94 193.192 Q1843.56 193.192 1841.61 195.877 Q1839.69 198.539 1839.69 203.238 Q1839.69 207.937 1841.61 210.622 Q1843.56 213.284 1846.94 213.284 Q1850.31 213.284 1852.24 210.622 Q1854.18 207.937 1854.18 203.238 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip780)" d="M1869.87 182.891 L1869.87 190.252 L1878.65 190.252 L1878.65 193.562 L1869.87 193.562 L1869.87 207.636 Q1869.87 210.807 1870.73 211.71 Q1871.61 212.613 1874.27 212.613 L1878.65 212.613 L1878.65 216.178 L1874.27 216.178 Q1869.34 216.178 1867.47 214.349 Q1865.59 212.497 1865.59 207.636 L1865.59 193.562 L1862.47 193.562 L1862.47 190.252 L1865.59 190.252 L1865.59 182.891 L1869.87 182.891 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip780)" d="M1884.25 190.252 L1888.51 190.252 L1888.51 216.178 L1884.25 216.178 L1884.25 190.252 M1884.25 180.159 L1888.51 180.159 L1888.51 185.553 L1884.25 185.553 L1884.25 180.159 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip780)" d="M1917.61 195.229 Q1919.2 192.358 1921.43 190.993 Q1923.65 189.627 1926.66 189.627 Q1930.71 189.627 1932.91 192.474 Q1935.11 195.298 1935.11 200.529 L1935.11 216.178 L1930.82 216.178 L1930.82 200.668 Q1930.82 196.942 1929.5 195.136 Q1928.18 193.33 1925.48 193.33 Q1922.17 193.33 1920.24 195.53 Q1918.32 197.729 1918.32 201.525 L1918.32 216.178 L1914.04 216.178 L1914.04 200.668 Q1914.04 196.918 1912.72 195.136 Q1911.4 193.33 1908.65 193.33 Q1905.38 193.33 1903.46 195.553 Q1901.54 197.752 1901.54 201.525 L1901.54 216.178 L1897.26 216.178 L1897.26 190.252 L1901.54 190.252 L1901.54 194.28 Q1903 191.895 1905.04 190.761 Q1907.07 189.627 1909.87 189.627 Q1912.7 189.627 1914.67 191.062 Q1916.66 192.497 1917.61 195.229 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip780)" d="M1943.6 190.252 L1947.86 190.252 L1947.86 216.178 L1943.6 216.178 L1943.6 190.252 M1943.6 180.159 L1947.86 180.159 L1947.86 185.553 L1943.6 185.553 L1943.6 180.159 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip780)" d="M1954.92 190.252 L1975.15 190.252 L1975.15 194.141 L1959.13 212.775 L1975.15 212.775 L1975.15 216.178 L1954.34 216.178 L1954.34 212.289 L1970.36 193.655 L1954.92 193.655 L1954.92 190.252 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip780)" d="M1993.44 203.145 Q1988.28 203.145 1986.29 204.326 Q1984.3 205.506 1984.3 208.354 Q1984.3 210.622 1985.78 211.965 Q1987.28 213.284 1989.85 213.284 Q1993.39 213.284 1995.52 210.784 Q1997.67 208.261 1997.67 204.094 L1997.67 203.145 L1993.44 203.145 M2001.93 201.386 L2001.93 216.178 L1997.67 216.178 L1997.67 212.242 Q1996.22 214.603 1994.04 215.738 Q1991.86 216.849 1988.72 216.849 Q1984.74 216.849 1982.37 214.627 Q1980.04 212.381 1980.04 208.631 Q1980.04 204.256 1982.95 202.034 Q1985.89 199.812 1991.7 199.812 L1997.67 199.812 L1997.67 199.395 Q1997.67 196.455 1995.73 194.858 Q1993.81 193.238 1990.31 193.238 Q1988.09 193.238 1985.99 193.77 Q1983.88 194.303 1981.93 195.367 L1981.93 191.432 Q1984.27 190.53 1986.47 190.09 Q1988.67 189.627 1990.75 189.627 Q1996.38 189.627 1999.16 192.543 Q2001.93 195.46 2001.93 201.386 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip780)" d="M2014.92 182.891 L2014.92 190.252 L2023.69 190.252 L2023.69 193.562 L2014.92 193.562 L2014.92 207.636 Q2014.92 210.807 2015.78 211.71 Q2016.66 212.613 2019.32 212.613 L2023.69 212.613 L2023.69 216.178 L2019.32 216.178 Q2014.39 216.178 2012.51 214.349 Q2010.64 212.497 2010.64 207.636 L2010.64 193.562 L2007.51 193.562 L2007.51 190.252 L2010.64 190.252 L2010.64 182.891 L2014.92 182.891 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip780)" d="M2029.3 190.252 L2033.55 190.252 L2033.55 216.178 L2029.3 216.178 L2029.3 190.252 M2029.3 180.159 L2033.55 180.159 L2033.55 185.553 L2029.3 185.553 L2029.3 180.159 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip780)" d="M2052.51 193.238 Q2049.09 193.238 2047.1 195.923 Q2045.11 198.585 2045.11 203.238 Q2045.11 207.891 2047.07 210.576 Q2049.06 213.238 2052.51 213.238 Q2055.92 213.238 2057.91 210.553 Q2059.9 207.867 2059.9 203.238 Q2059.9 198.631 2057.91 195.946 Q2055.92 193.238 2052.51 193.238 M2052.51 189.627 Q2058.07 189.627 2061.24 193.238 Q2064.41 196.849 2064.41 203.238 Q2064.41 209.604 2061.24 213.238 Q2058.07 216.849 2052.51 216.849 Q2046.93 216.849 2043.76 213.238 Q2040.61 209.604 2040.61 203.238 Q2040.61 196.849 2043.76 193.238 Q2046.93 189.627 2052.51 189.627 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip780)" d="M2093.02 200.529 L2093.02 216.178 L2088.76 216.178 L2088.76 200.668 Q2088.76 196.988 2087.33 195.159 Q2085.89 193.33 2083.02 193.33 Q2079.57 193.33 2077.58 195.53 Q2075.59 197.729 2075.59 201.525 L2075.59 216.178 L2071.31 216.178 L2071.31 190.252 L2075.59 190.252 L2075.59 194.28 Q2077.12 191.942 2079.18 190.784 Q2081.26 189.627 2083.97 189.627 Q2088.44 189.627 2090.73 192.405 Q2093.02 195.159 2093.02 200.529 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip780)" d="M2120.8 182.891 L2120.8 190.252 L2129.57 190.252 L2129.57 193.562 L2120.8 193.562 L2120.8 207.636 Q2120.8 210.807 2121.66 211.71 Q2122.54 212.613 2125.2 212.613 L2129.57 212.613 L2129.57 216.178 L2125.2 216.178 Q2120.27 216.178 2118.39 214.349 Q2116.52 212.497 2116.52 207.636 L2116.52 193.562 L2113.39 193.562 L2113.39 190.252 L2116.52 190.252 L2116.52 182.891 L2120.8 182.891 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip780)" d="M2146.96 203.145 Q2141.79 203.145 2139.8 204.326 Q2137.81 205.506 2137.81 208.354 Q2137.81 210.622 2139.29 211.965 Q2140.8 213.284 2143.37 213.284 Q2146.91 213.284 2149.04 210.784 Q2151.19 208.261 2151.19 204.094 L2151.19 203.145 L2146.96 203.145 M2155.45 201.386 L2155.45 216.178 L2151.19 216.178 L2151.19 212.242 Q2149.73 214.603 2147.56 215.738 Q2145.38 216.849 2142.23 216.849 Q2138.25 216.849 2135.89 214.627 Q2133.55 212.381 2133.55 208.631 Q2133.55 204.256 2136.47 202.034 Q2139.41 199.812 2145.22 199.812 L2151.19 199.812 L2151.19 199.395 Q2151.19 196.455 2149.25 194.858 Q2147.33 193.238 2143.83 193.238 Q2141.61 193.238 2139.5 193.77 Q2137.4 194.303 2135.45 195.367 L2135.45 191.432 Q2137.79 190.53 2139.99 190.09 Q2142.19 189.627 2144.27 189.627 Q2149.9 189.627 2152.67 192.543 Q2155.45 195.46 2155.45 201.386 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip780)" d="M2179.25 194.233 Q2178.53 193.817 2177.67 193.631 Q2176.84 193.423 2175.82 193.423 Q2172.21 193.423 2170.27 195.784 Q2168.35 198.122 2168.35 202.52 L2168.35 216.178 L2164.06 216.178 L2164.06 190.252 L2168.35 190.252 L2168.35 194.28 Q2169.69 191.918 2171.84 190.784 Q2173.99 189.627 2177.07 189.627 Q2177.51 189.627 2178.04 189.696 Q2178.58 189.743 2179.22 189.858 L2179.25 194.233 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip780)" d="M2199.94 202.914 Q2199.94 198.284 2198.02 195.738 Q2196.12 193.192 2192.67 193.192 Q2189.25 193.192 2187.33 195.738 Q2185.43 198.284 2185.43 202.914 Q2185.43 207.52 2187.33 210.066 Q2189.25 212.613 2192.67 212.613 Q2196.12 212.613 2198.02 210.066 Q2199.94 207.52 2199.94 202.914 M2204.2 212.96 Q2204.2 219.58 2201.26 222.798 Q2198.32 226.039 2192.26 226.039 Q2190.01 226.039 2188.02 225.691 Q2186.03 225.367 2184.16 224.673 L2184.16 220.529 Q2186.03 221.548 2187.86 222.034 Q2189.69 222.52 2191.59 222.52 Q2195.78 222.52 2197.86 220.321 Q2199.94 218.145 2199.94 213.724 L2199.94 211.617 Q2198.62 213.909 2196.56 215.043 Q2194.5 216.178 2191.63 216.178 Q2186.86 216.178 2183.95 212.543 Q2181.03 208.909 2181.03 202.914 Q2181.03 196.895 2183.95 193.261 Q2186.86 189.627 2191.63 189.627 Q2194.5 189.627 2196.56 190.761 Q2198.62 191.895 2199.94 194.187 L2199.94 190.252 L2204.2 190.252 L2204.2 212.96 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip780)" d="M2235.15 202.15 L2235.15 204.233 L2215.57 204.233 Q2215.85 208.631 2218.21 210.946 Q2220.59 213.238 2224.83 213.238 Q2227.28 213.238 2229.57 212.636 Q2231.89 212.034 2234.16 210.83 L2234.16 214.858 Q2231.86 215.83 2229.46 216.34 Q2227.05 216.849 2224.57 216.849 Q2218.37 216.849 2214.73 213.238 Q2211.12 209.627 2211.12 203.469 Q2211.12 197.104 2214.55 193.377 Q2218 189.627 2223.83 189.627 Q2229.06 189.627 2232.1 193.006 Q2235.15 196.363 2235.15 202.15 M2230.89 200.9 Q2230.85 197.405 2228.92 195.321 Q2227.03 193.238 2223.88 193.238 Q2220.31 193.238 2218.16 195.252 Q2216.03 197.266 2215.71 200.923 L2230.89 200.9 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip780)" d="M2246.35 182.891 L2246.35 190.252 L2255.13 190.252 L2255.13 193.562 L2246.35 193.562 L2246.35 207.636 Q2246.35 210.807 2247.21 211.71 Q2248.09 212.613 2250.75 212.613 L2255.13 212.613 L2255.13 216.178 L2250.75 216.178 Q2245.82 216.178 2243.95 214.349 Q2242.07 212.497 2242.07 207.636 L2242.07 193.562 L2238.95 193.562 L2238.95 190.252 L2242.07 190.252 L2242.07 182.891 L2246.35 182.891 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /></svg>




Not that good. So let's do a bit of optimization!


```julia
opt = Optim.optimize(objective, p; iterations=250) # do max. 250 iterations
obj_after = opt.minimum # much better!
p_res = opt.minimizer # the optimized parameters
```




    4-element Vector{Float64}:
     1.000927423495889
     0.9780437253734777
     0.11212447550094248
     0.09761028417513656



Looks promising, let's have a look on the results plot:


```julia
s_fmu = simulateFMU(p_res); # simulate the position

plot(tSave, s_fmu; label="FMU")
plot!(tSave, s_tar; label="Optimization target")
```




<?xml version="1.0" encoding="utf-8"?>
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="600" height="400" viewBox="0 0 2400 1600">
<defs>
  <clipPath id="clip870">
    <rect x="0" y="0" width="2400" height="1600"/>
  </clipPath>
</defs>
<path clip-path="url(#clip870)" d="M0 1600 L2400 1600 L2400 0 L0 0  Z" fill="#ffffff" fill-rule="evenodd" fill-opacity="1"/>
<defs>
  <clipPath id="clip871">
    <rect x="480" y="0" width="1681" height="1600"/>
  </clipPath>
</defs>
<path clip-path="url(#clip870)" d="M156.112 1486.45 L2352.76 1486.45 L2352.76 47.2441 L156.112 47.2441  Z" fill="#ffffff" fill-rule="evenodd" fill-opacity="1"/>
<defs>
  <clipPath id="clip872">
    <rect x="156" y="47" width="2198" height="1440"/>
  </clipPath>
</defs>
<polyline clip-path="url(#clip872)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:2; stroke-opacity:0.1; fill:none" points="218.281,1486.45 218.281,47.2441 "/>
<polyline clip-path="url(#clip872)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:2; stroke-opacity:0.1; fill:none" points="632.742,1486.45 632.742,47.2441 "/>
<polyline clip-path="url(#clip872)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:2; stroke-opacity:0.1; fill:none" points="1047.2,1486.45 1047.2,47.2441 "/>
<polyline clip-path="url(#clip872)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:2; stroke-opacity:0.1; fill:none" points="1461.66,1486.45 1461.66,47.2441 "/>
<polyline clip-path="url(#clip872)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:2; stroke-opacity:0.1; fill:none" points="1876.13,1486.45 1876.13,47.2441 "/>
<polyline clip-path="url(#clip872)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:2; stroke-opacity:0.1; fill:none" points="2290.59,1486.45 2290.59,47.2441 "/>
<polyline clip-path="url(#clip872)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:2; stroke-opacity:0.1; fill:none" points="156.112,1445.77 2352.76,1445.77 "/>
<polyline clip-path="url(#clip872)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:2; stroke-opacity:0.1; fill:none" points="156.112,1109.56 2352.76,1109.56 "/>
<polyline clip-path="url(#clip872)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:2; stroke-opacity:0.1; fill:none" points="156.112,773.351 2352.76,773.351 "/>
<polyline clip-path="url(#clip872)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:2; stroke-opacity:0.1; fill:none" points="156.112,437.143 2352.76,437.143 "/>
<polyline clip-path="url(#clip872)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:2; stroke-opacity:0.1; fill:none" points="156.112,100.935 2352.76,100.935 "/>
<polyline clip-path="url(#clip870)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="156.112,1486.45 2352.76,1486.45 "/>
<polyline clip-path="url(#clip870)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="218.281,1486.45 218.281,1467.55 "/>
<polyline clip-path="url(#clip870)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="632.742,1486.45 632.742,1467.55 "/>
<polyline clip-path="url(#clip870)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="1047.2,1486.45 1047.2,1467.55 "/>
<polyline clip-path="url(#clip870)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="1461.66,1486.45 1461.66,1467.55 "/>
<polyline clip-path="url(#clip870)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="1876.13,1486.45 1876.13,1467.55 "/>
<polyline clip-path="url(#clip870)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="2290.59,1486.45 2290.59,1467.55 "/>
<path clip-path="url(#clip870)" d="M218.281 1517.37 Q214.67 1517.37 212.842 1520.93 Q211.036 1524.47 211.036 1531.6 Q211.036 1538.71 212.842 1542.27 Q214.67 1545.82 218.281 1545.82 Q221.916 1545.82 223.721 1542.27 Q225.55 1538.71 225.55 1531.6 Q225.55 1524.47 223.721 1520.93 Q221.916 1517.37 218.281 1517.37 M218.281 1513.66 Q224.091 1513.66 227.147 1518.27 Q230.226 1522.85 230.226 1531.6 Q230.226 1540.33 227.147 1544.94 Q224.091 1549.52 218.281 1549.52 Q212.471 1549.52 209.392 1544.94 Q206.337 1540.33 206.337 1531.6 Q206.337 1522.85 209.392 1518.27 Q212.471 1513.66 218.281 1513.66 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip870)" d="M623.124 1544.91 L630.763 1544.91 L630.763 1518.55 L622.453 1520.21 L622.453 1515.95 L630.717 1514.29 L635.393 1514.29 L635.393 1544.91 L643.032 1544.91 L643.032 1548.85 L623.124 1548.85 L623.124 1544.91 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip870)" d="M1041.86 1544.91 L1058.18 1544.91 L1058.18 1548.85 L1036.23 1548.85 L1036.23 1544.91 Q1038.89 1542.16 1043.48 1537.53 Q1048.08 1532.88 1049.26 1531.53 Q1051.51 1529.01 1052.39 1527.27 Q1053.29 1525.51 1053.29 1523.82 Q1053.29 1521.07 1051.35 1519.33 Q1049.43 1517.6 1046.32 1517.6 Q1044.12 1517.6 1041.67 1518.36 Q1039.24 1519.13 1036.46 1520.68 L1036.46 1515.95 Q1039.29 1514.82 1041.74 1514.24 Q1044.19 1513.66 1046.23 1513.66 Q1051.6 1513.66 1054.8 1516.35 Q1057.99 1519.03 1057.99 1523.52 Q1057.99 1525.65 1057.18 1527.57 Q1056.39 1529.47 1054.29 1532.07 Q1053.71 1532.74 1050.61 1535.95 Q1047.5 1539.15 1041.86 1544.91 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip870)" d="M1465.91 1530.21 Q1469.27 1530.93 1471.14 1533.2 Q1473.04 1535.47 1473.04 1538.8 Q1473.04 1543.92 1469.52 1546.72 Q1466 1549.52 1459.52 1549.52 Q1457.35 1549.52 1455.03 1549.08 Q1452.74 1548.66 1450.29 1547.81 L1450.29 1543.29 Q1452.23 1544.43 1454.55 1545.01 Q1456.86 1545.58 1459.38 1545.58 Q1463.78 1545.58 1466.07 1543.85 Q1468.39 1542.11 1468.39 1538.8 Q1468.39 1535.75 1466.24 1534.03 Q1464.11 1532.3 1460.29 1532.3 L1456.26 1532.3 L1456.26 1528.45 L1460.47 1528.45 Q1463.92 1528.45 1465.75 1527.09 Q1467.58 1525.7 1467.58 1523.11 Q1467.58 1520.45 1465.68 1519.03 Q1463.81 1517.6 1460.29 1517.6 Q1458.37 1517.6 1456.17 1518.01 Q1453.97 1518.43 1451.33 1519.31 L1451.33 1515.14 Q1453.99 1514.4 1456.31 1514.03 Q1458.64 1513.66 1460.7 1513.66 Q1466.03 1513.66 1469.13 1516.09 Q1472.23 1518.5 1472.23 1522.62 Q1472.23 1525.49 1470.59 1527.48 Q1468.94 1529.45 1465.91 1530.21 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip870)" d="M1879.13 1518.36 L1867.33 1536.81 L1879.13 1536.81 L1879.13 1518.36 M1877.91 1514.29 L1883.79 1514.29 L1883.79 1536.81 L1888.72 1536.81 L1888.72 1540.7 L1883.79 1540.7 L1883.79 1548.85 L1879.13 1548.85 L1879.13 1540.7 L1863.53 1540.7 L1863.53 1536.19 L1877.91 1514.29 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip870)" d="M2280.86 1514.29 L2299.22 1514.29 L2299.22 1518.22 L2285.15 1518.22 L2285.15 1526.7 Q2286.17 1526.35 2287.18 1526.19 Q2288.2 1526 2289.22 1526 Q2295.01 1526 2298.39 1529.17 Q2301.77 1532.34 2301.77 1537.76 Q2301.77 1543.34 2298.3 1546.44 Q2294.82 1549.52 2288.5 1549.52 Q2286.33 1549.52 2284.06 1549.15 Q2281.81 1548.78 2279.41 1548.04 L2279.41 1543.34 Q2281.49 1544.47 2283.71 1545.03 Q2285.93 1545.58 2288.41 1545.58 Q2292.42 1545.58 2294.75 1543.48 Q2297.09 1541.37 2297.09 1537.76 Q2297.09 1534.15 2294.75 1532.04 Q2292.42 1529.94 2288.41 1529.94 Q2286.54 1529.94 2284.66 1530.35 Q2282.81 1530.77 2280.86 1531.65 L2280.86 1514.29 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><polyline clip-path="url(#clip870)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="156.112,1486.45 156.112,47.2441 "/>
<polyline clip-path="url(#clip870)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="156.112,1445.77 175.01,1445.77 "/>
<polyline clip-path="url(#clip870)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="156.112,1109.56 175.01,1109.56 "/>
<polyline clip-path="url(#clip870)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="156.112,773.351 175.01,773.351 "/>
<polyline clip-path="url(#clip870)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="156.112,437.143 175.01,437.143 "/>
<polyline clip-path="url(#clip870)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="156.112,100.935 175.01,100.935 "/>
<path clip-path="url(#clip870)" d="M62.9365 1431.57 Q59.3254 1431.57 57.4967 1435.13 Q55.6912 1438.67 55.6912 1445.8 Q55.6912 1452.91 57.4967 1456.47 Q59.3254 1460.01 62.9365 1460.01 Q66.5707 1460.01 68.3763 1456.47 Q70.205 1452.91 70.205 1445.8 Q70.205 1438.67 68.3763 1435.13 Q66.5707 1431.57 62.9365 1431.57 M62.9365 1427.86 Q68.7467 1427.86 71.8022 1432.47 Q74.8809 1437.05 74.8809 1445.8 Q74.8809 1454.53 71.8022 1459.14 Q68.7467 1463.72 62.9365 1463.72 Q57.1264 1463.72 54.0477 1459.14 Q50.9921 1454.53 50.9921 1445.8 Q50.9921 1437.05 54.0477 1432.47 Q57.1264 1427.86 62.9365 1427.86 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip870)" d="M83.0984 1457.17 L87.9827 1457.17 L87.9827 1463.05 L83.0984 1463.05 L83.0984 1457.17 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip870)" d="M108.168 1431.57 Q104.557 1431.57 102.728 1435.13 Q100.922 1438.67 100.922 1445.8 Q100.922 1452.91 102.728 1456.47 Q104.557 1460.01 108.168 1460.01 Q111.802 1460.01 113.608 1456.47 Q115.436 1452.91 115.436 1445.8 Q115.436 1438.67 113.608 1435.13 Q111.802 1431.57 108.168 1431.57 M108.168 1427.86 Q113.978 1427.86 117.033 1432.47 Q120.112 1437.05 120.112 1445.8 Q120.112 1454.53 117.033 1459.14 Q113.978 1463.72 108.168 1463.72 Q102.358 1463.72 99.2789 1459.14 Q96.2234 1454.53 96.2234 1445.8 Q96.2234 1437.05 99.2789 1432.47 Q102.358 1427.86 108.168 1427.86 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip870)" d="M63.9319 1095.36 Q60.3208 1095.36 58.4921 1098.92 Q56.6865 1102.46 56.6865 1109.59 Q56.6865 1116.7 58.4921 1120.27 Q60.3208 1123.81 63.9319 1123.81 Q67.5661 1123.81 69.3717 1120.27 Q71.2004 1116.7 71.2004 1109.59 Q71.2004 1102.46 69.3717 1098.92 Q67.5661 1095.36 63.9319 1095.36 M63.9319 1091.65 Q69.742 1091.65 72.7976 1096.26 Q75.8763 1100.84 75.8763 1109.59 Q75.8763 1118.32 72.7976 1122.93 Q69.742 1127.51 63.9319 1127.51 Q58.1217 1127.51 55.043 1122.93 Q51.9875 1118.32 51.9875 1109.59 Q51.9875 1100.84 55.043 1096.26 Q58.1217 1091.65 63.9319 1091.65 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip870)" d="M84.0938 1120.96 L88.978 1120.96 L88.978 1126.84 L84.0938 1126.84 L84.0938 1120.96 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip870)" d="M99.2095 1092.28 L117.566 1092.28 L117.566 1096.21 L103.492 1096.21 L103.492 1104.69 Q104.51 1104.34 105.529 1104.18 Q106.547 1103.99 107.566 1103.99 Q113.353 1103.99 116.733 1107.16 Q120.112 1110.33 120.112 1115.75 Q120.112 1121.33 116.64 1124.43 Q113.168 1127.51 106.848 1127.51 Q104.672 1127.51 102.404 1127.14 Q100.159 1126.77 97.7511 1126.03 L97.7511 1121.33 Q99.8345 1122.46 102.057 1123.02 Q104.279 1123.58 106.756 1123.58 Q110.76 1123.58 113.098 1121.47 Q115.436 1119.36 115.436 1115.75 Q115.436 1112.14 113.098 1110.03 Q110.76 1107.93 106.756 1107.93 Q104.881 1107.93 103.006 1108.34 Q101.154 1108.76 99.2095 1109.64 L99.2095 1092.28 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip870)" d="M53.7467 786.696 L61.3856 786.696 L61.3856 760.33 L53.0754 761.997 L53.0754 757.738 L61.3393 756.071 L66.0152 756.071 L66.0152 786.696 L73.654 786.696 L73.654 790.631 L53.7467 790.631 L53.7467 786.696 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip870)" d="M83.0984 784.752 L87.9827 784.752 L87.9827 790.631 L83.0984 790.631 L83.0984 784.752 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip870)" d="M108.168 759.15 Q104.557 759.15 102.728 762.715 Q100.922 766.256 100.922 773.386 Q100.922 780.492 102.728 784.057 Q104.557 787.599 108.168 787.599 Q111.802 787.599 113.608 784.057 Q115.436 780.492 115.436 773.386 Q115.436 766.256 113.608 762.715 Q111.802 759.15 108.168 759.15 M108.168 755.446 Q113.978 755.446 117.033 760.053 Q120.112 764.636 120.112 773.386 Q120.112 782.113 117.033 786.719 Q113.978 791.303 108.168 791.303 Q102.358 791.303 99.2789 786.719 Q96.2234 782.113 96.2234 773.386 Q96.2234 764.636 99.2789 760.053 Q102.358 755.446 108.168 755.446 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip870)" d="M54.7421 450.488 L62.381 450.488 L62.381 424.122 L54.0708 425.789 L54.0708 421.53 L62.3347 419.863 L67.0106 419.863 L67.0106 450.488 L74.6494 450.488 L74.6494 454.423 L54.7421 454.423 L54.7421 450.488 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip870)" d="M84.0938 448.544 L88.978 448.544 L88.978 454.423 L84.0938 454.423 L84.0938 448.544 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip870)" d="M99.2095 419.863 L117.566 419.863 L117.566 423.798 L103.492 423.798 L103.492 432.271 Q104.51 431.923 105.529 431.761 Q106.547 431.576 107.566 431.576 Q113.353 431.576 116.733 434.747 Q120.112 437.919 120.112 443.335 Q120.112 448.914 116.64 452.016 Q113.168 455.095 106.848 455.095 Q104.672 455.095 102.404 454.724 Q100.159 454.354 97.7511 453.613 L97.7511 448.914 Q99.8345 450.048 102.057 450.604 Q104.279 451.159 106.756 451.159 Q110.76 451.159 113.098 449.053 Q115.436 446.946 115.436 443.335 Q115.436 439.724 113.098 437.618 Q110.76 435.511 106.756 435.511 Q104.881 435.511 103.006 435.928 Q101.154 436.345 99.2095 437.224 L99.2095 419.863 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip870)" d="M56.9643 114.28 L73.2837 114.28 L73.2837 118.215 L51.3393 118.215 L51.3393 114.28 Q54.0014 111.525 58.5847 106.896 Q63.1911 102.243 64.3717 100.9 Q66.617 98.3773 67.4967 96.6412 Q68.3994 94.882 68.3994 93.1922 Q68.3994 90.4376 66.455 88.7015 Q64.5337 86.9654 61.4319 86.9654 Q59.2328 86.9654 56.7791 87.7293 Q54.3486 88.4931 51.5708 90.0441 L51.5708 85.3219 Q54.3949 84.1876 56.8486 83.6089 Q59.3023 83.0302 61.3393 83.0302 Q66.7096 83.0302 69.9041 85.7154 Q73.0985 88.4006 73.0985 92.8913 Q73.0985 95.0209 72.2883 96.9422 Q71.5013 98.8403 69.3948 101.433 Q68.8161 102.104 65.7143 105.322 Q62.6124 108.516 56.9643 114.28 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip870)" d="M83.0984 112.336 L87.9827 112.336 L87.9827 118.215 L83.0984 118.215 L83.0984 112.336 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip870)" d="M108.168 86.7339 Q104.557 86.7339 102.728 90.2987 Q100.922 93.8403 100.922 100.97 Q100.922 108.076 102.728 111.641 Q104.557 115.183 108.168 115.183 Q111.802 115.183 113.608 111.641 Q115.436 108.076 115.436 100.97 Q115.436 93.8403 113.608 90.2987 Q111.802 86.7339 108.168 86.7339 M108.168 83.0302 Q113.978 83.0302 117.033 87.6367 Q120.112 92.22 120.112 100.97 Q120.112 109.697 117.033 114.303 Q113.978 118.886 108.168 118.886 Q102.358 118.886 99.2789 114.303 Q96.2234 109.697 96.2234 100.97 Q96.2234 92.22 99.2789 87.6367 Q102.358 83.0302 108.168 83.0302 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><polyline clip-path="url(#clip872)" style="stroke:#009af9; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="218.281,772.728 259.727,708.495 301.174,642.323 342.62,576.882 384.066,512.925 425.512,451.185 466.958,392.37 508.404,337.156 549.85,286.176 591.296,240.015 632.742,199.202 674.188,164.207 715.635,135.431 757.081,113.204 798.527,97.7816 839.973,89.3399 881.419,87.9763 922.865,93.7063 964.311,106.464 1005.76,126.104 1047.2,152.399 1088.65,185.049 1130.1,223.679 1171.54,267.845 1212.99,317.04 1254.43,370.701 1295.88,428.21 1337.33,488.909 1378.77,552.1 1420.22,617.059 1461.66,683.04 1503.11,749.285 1544.56,815.035 1586,879.535 1627.45,942.045 1668.9,1001.85 1710.34,1058.25 1751.79,1110.62 1793.23,1158.35 1834.68,1200.88 1876.13,1237.74 1917.57,1268.49 1959.02,1292.8 2000.46,1310.37 2041.91,1321 2083.36,1324.58 2124.8,1321.06 2166.25,1310.48 2207.69,1292.97 2249.14,1268.73 2290.59,1238.02 "/>
<polyline clip-path="url(#clip872)" style="stroke:#e26f46; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="218.281,773.351 259.727,706.222 301.174,639.763 342.62,574.639 384.066,511.5 425.512,450.978 466.958,393.677 508.404,340.169 549.85,290.99 591.296,246.63 632.742,207.533 674.188,174.089 715.635,146.633 757.081,125.439 798.527,110.719 839.973,102.62 881.419,101.222 922.865,106.54 964.311,118.52 1005.76,137.044 1047.2,161.925 1088.65,192.915 1130.1,229.705 1171.54,271.927 1212.99,319.159 1254.43,370.929 1295.88,426.72 1337.33,485.974 1378.77,548.1 1420.22,612.476 1461.66,678.46 1503.11,745.392 1544.56,812.603 1586,879.422 1627.45,945.181 1668.9,1009.22 1710.34,1070.91 1751.79,1129.62 1793.23,1184.77 1834.68,1235.82 1876.13,1282.24 1917.57,1323.57 1959.02,1359.41 2000.46,1389.4 2041.91,1413.22 2083.36,1430.66 2124.8,1441.53 2166.25,1445.72 2207.69,1443.19 2249.14,1433.97 2290.59,1418.15 "/>
<path clip-path="url(#clip870)" d="M1601.19 250.738 L2279.53 250.738 L2279.53 95.2176 L1601.19 95.2176  Z" fill="#ffffff" fill-rule="evenodd" fill-opacity="1"/>
<polyline clip-path="url(#clip870)" style="stroke:#000000; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="1601.19,250.738 2279.53,250.738 2279.53,95.2176 1601.19,95.2176 1601.19,250.738 "/>
<polyline clip-path="url(#clip870)" style="stroke:#009af9; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="1625.6,147.058 1772.04,147.058 "/>
<path clip-path="url(#clip870)" d="M1796.45 129.778 L1816.31 129.778 L1816.31 133.713 L1801.13 133.713 L1801.13 143.898 L1814.83 143.898 L1814.83 147.833 L1801.13 147.833 L1801.13 164.338 L1796.45 164.338 L1796.45 129.778 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip870)" d="M1823.72 129.778 L1830.69 129.778 L1839.5 153.296 L1848.37 129.778 L1855.34 129.778 L1855.34 164.338 L1850.78 164.338 L1850.78 133.99 L1841.87 157.694 L1837.17 157.694 L1828.25 133.99 L1828.25 164.338 L1823.72 164.338 L1823.72 129.778 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip870)" d="M1864.09 129.778 L1868.79 129.778 L1868.79 150.773 Q1868.79 156.328 1870.8 158.782 Q1872.81 161.213 1877.33 161.213 Q1881.82 161.213 1883.83 158.782 Q1885.85 156.328 1885.85 150.773 L1885.85 129.778 L1890.55 129.778 L1890.55 151.352 Q1890.55 158.111 1887.19 161.56 Q1883.86 165.009 1877.33 165.009 Q1870.78 165.009 1867.42 161.56 Q1864.09 158.111 1864.09 151.352 L1864.09 129.778 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><polyline clip-path="url(#clip870)" style="stroke:#e26f46; stroke-linecap:round; stroke-linejoin:round; stroke-width:4; stroke-opacity:1; fill:none" points="1625.6,198.898 1772.04,198.898 "/>
<path clip-path="url(#clip870)" d="M1812.47 184.789 Q1807.38 184.789 1804.37 188.585 Q1801.38 192.381 1801.38 198.932 Q1801.38 205.46 1804.37 209.256 Q1807.38 213.053 1812.47 213.053 Q1817.56 213.053 1820.52 209.256 Q1823.51 205.46 1823.51 198.932 Q1823.51 192.381 1820.52 188.585 Q1817.56 184.789 1812.47 184.789 M1812.47 180.993 Q1819.74 180.993 1824.09 185.877 Q1828.44 190.738 1828.44 198.932 Q1828.44 207.104 1824.09 211.988 Q1819.74 216.849 1812.47 216.849 Q1805.18 216.849 1800.8 211.988 Q1796.45 207.127 1796.45 198.932 Q1796.45 190.738 1800.8 185.877 Q1805.18 180.993 1812.47 180.993 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip870)" d="M1839.69 212.289 L1839.69 226.039 L1835.41 226.039 L1835.41 190.252 L1839.69 190.252 L1839.69 194.187 Q1841.03 191.872 1843.07 190.761 Q1845.13 189.627 1847.98 189.627 Q1852.7 189.627 1855.64 193.377 Q1858.6 197.127 1858.6 203.238 Q1858.6 209.349 1855.64 213.099 Q1852.7 216.849 1847.98 216.849 Q1845.13 216.849 1843.07 215.738 Q1841.03 214.603 1839.69 212.289 M1854.18 203.238 Q1854.18 198.539 1852.24 195.877 Q1850.31 193.192 1846.94 193.192 Q1843.56 193.192 1841.61 195.877 Q1839.69 198.539 1839.69 203.238 Q1839.69 207.937 1841.61 210.622 Q1843.56 213.284 1846.94 213.284 Q1850.31 213.284 1852.24 210.622 Q1854.18 207.937 1854.18 203.238 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip870)" d="M1869.87 182.891 L1869.87 190.252 L1878.65 190.252 L1878.65 193.562 L1869.87 193.562 L1869.87 207.636 Q1869.87 210.807 1870.73 211.71 Q1871.61 212.613 1874.27 212.613 L1878.65 212.613 L1878.65 216.178 L1874.27 216.178 Q1869.34 216.178 1867.47 214.349 Q1865.59 212.497 1865.59 207.636 L1865.59 193.562 L1862.47 193.562 L1862.47 190.252 L1865.59 190.252 L1865.59 182.891 L1869.87 182.891 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip870)" d="M1884.25 190.252 L1888.51 190.252 L1888.51 216.178 L1884.25 216.178 L1884.25 190.252 M1884.25 180.159 L1888.51 180.159 L1888.51 185.553 L1884.25 185.553 L1884.25 180.159 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip870)" d="M1917.61 195.229 Q1919.2 192.358 1921.43 190.993 Q1923.65 189.627 1926.66 189.627 Q1930.71 189.627 1932.91 192.474 Q1935.11 195.298 1935.11 200.529 L1935.11 216.178 L1930.82 216.178 L1930.82 200.668 Q1930.82 196.942 1929.5 195.136 Q1928.18 193.33 1925.48 193.33 Q1922.17 193.33 1920.24 195.53 Q1918.32 197.729 1918.32 201.525 L1918.32 216.178 L1914.04 216.178 L1914.04 200.668 Q1914.04 196.918 1912.72 195.136 Q1911.4 193.33 1908.65 193.33 Q1905.38 193.33 1903.46 195.553 Q1901.54 197.752 1901.54 201.525 L1901.54 216.178 L1897.26 216.178 L1897.26 190.252 L1901.54 190.252 L1901.54 194.28 Q1903 191.895 1905.04 190.761 Q1907.07 189.627 1909.87 189.627 Q1912.7 189.627 1914.67 191.062 Q1916.66 192.497 1917.61 195.229 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip870)" d="M1943.6 190.252 L1947.86 190.252 L1947.86 216.178 L1943.6 216.178 L1943.6 190.252 M1943.6 180.159 L1947.86 180.159 L1947.86 185.553 L1943.6 185.553 L1943.6 180.159 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip870)" d="M1954.92 190.252 L1975.15 190.252 L1975.15 194.141 L1959.13 212.775 L1975.15 212.775 L1975.15 216.178 L1954.34 216.178 L1954.34 212.289 L1970.36 193.655 L1954.92 193.655 L1954.92 190.252 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip870)" d="M1993.44 203.145 Q1988.28 203.145 1986.29 204.326 Q1984.3 205.506 1984.3 208.354 Q1984.3 210.622 1985.78 211.965 Q1987.28 213.284 1989.85 213.284 Q1993.39 213.284 1995.52 210.784 Q1997.67 208.261 1997.67 204.094 L1997.67 203.145 L1993.44 203.145 M2001.93 201.386 L2001.93 216.178 L1997.67 216.178 L1997.67 212.242 Q1996.22 214.603 1994.04 215.738 Q1991.86 216.849 1988.72 216.849 Q1984.74 216.849 1982.37 214.627 Q1980.04 212.381 1980.04 208.631 Q1980.04 204.256 1982.95 202.034 Q1985.89 199.812 1991.7 199.812 L1997.67 199.812 L1997.67 199.395 Q1997.67 196.455 1995.73 194.858 Q1993.81 193.238 1990.31 193.238 Q1988.09 193.238 1985.99 193.77 Q1983.88 194.303 1981.93 195.367 L1981.93 191.432 Q1984.27 190.53 1986.47 190.09 Q1988.67 189.627 1990.75 189.627 Q1996.38 189.627 1999.16 192.543 Q2001.93 195.46 2001.93 201.386 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip870)" d="M2014.92 182.891 L2014.92 190.252 L2023.69 190.252 L2023.69 193.562 L2014.92 193.562 L2014.92 207.636 Q2014.92 210.807 2015.78 211.71 Q2016.66 212.613 2019.32 212.613 L2023.69 212.613 L2023.69 216.178 L2019.32 216.178 Q2014.39 216.178 2012.51 214.349 Q2010.64 212.497 2010.64 207.636 L2010.64 193.562 L2007.51 193.562 L2007.51 190.252 L2010.64 190.252 L2010.64 182.891 L2014.92 182.891 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip870)" d="M2029.3 190.252 L2033.55 190.252 L2033.55 216.178 L2029.3 216.178 L2029.3 190.252 M2029.3 180.159 L2033.55 180.159 L2033.55 185.553 L2029.3 185.553 L2029.3 180.159 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip870)" d="M2052.51 193.238 Q2049.09 193.238 2047.1 195.923 Q2045.11 198.585 2045.11 203.238 Q2045.11 207.891 2047.07 210.576 Q2049.06 213.238 2052.51 213.238 Q2055.92 213.238 2057.91 210.553 Q2059.9 207.867 2059.9 203.238 Q2059.9 198.631 2057.91 195.946 Q2055.92 193.238 2052.51 193.238 M2052.51 189.627 Q2058.07 189.627 2061.24 193.238 Q2064.41 196.849 2064.41 203.238 Q2064.41 209.604 2061.24 213.238 Q2058.07 216.849 2052.51 216.849 Q2046.93 216.849 2043.76 213.238 Q2040.61 209.604 2040.61 203.238 Q2040.61 196.849 2043.76 193.238 Q2046.93 189.627 2052.51 189.627 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip870)" d="M2093.02 200.529 L2093.02 216.178 L2088.76 216.178 L2088.76 200.668 Q2088.76 196.988 2087.33 195.159 Q2085.89 193.33 2083.02 193.33 Q2079.57 193.33 2077.58 195.53 Q2075.59 197.729 2075.59 201.525 L2075.59 216.178 L2071.31 216.178 L2071.31 190.252 L2075.59 190.252 L2075.59 194.28 Q2077.12 191.942 2079.18 190.784 Q2081.26 189.627 2083.97 189.627 Q2088.44 189.627 2090.73 192.405 Q2093.02 195.159 2093.02 200.529 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip870)" d="M2120.8 182.891 L2120.8 190.252 L2129.57 190.252 L2129.57 193.562 L2120.8 193.562 L2120.8 207.636 Q2120.8 210.807 2121.66 211.71 Q2122.54 212.613 2125.2 212.613 L2129.57 212.613 L2129.57 216.178 L2125.2 216.178 Q2120.27 216.178 2118.39 214.349 Q2116.52 212.497 2116.52 207.636 L2116.52 193.562 L2113.39 193.562 L2113.39 190.252 L2116.52 190.252 L2116.52 182.891 L2120.8 182.891 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip870)" d="M2146.96 203.145 Q2141.79 203.145 2139.8 204.326 Q2137.81 205.506 2137.81 208.354 Q2137.81 210.622 2139.29 211.965 Q2140.8 213.284 2143.37 213.284 Q2146.91 213.284 2149.04 210.784 Q2151.19 208.261 2151.19 204.094 L2151.19 203.145 L2146.96 203.145 M2155.45 201.386 L2155.45 216.178 L2151.19 216.178 L2151.19 212.242 Q2149.73 214.603 2147.56 215.738 Q2145.38 216.849 2142.23 216.849 Q2138.25 216.849 2135.89 214.627 Q2133.55 212.381 2133.55 208.631 Q2133.55 204.256 2136.47 202.034 Q2139.41 199.812 2145.22 199.812 L2151.19 199.812 L2151.19 199.395 Q2151.19 196.455 2149.25 194.858 Q2147.33 193.238 2143.83 193.238 Q2141.61 193.238 2139.5 193.77 Q2137.4 194.303 2135.45 195.367 L2135.45 191.432 Q2137.79 190.53 2139.99 190.09 Q2142.19 189.627 2144.27 189.627 Q2149.9 189.627 2152.67 192.543 Q2155.45 195.46 2155.45 201.386 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip870)" d="M2179.25 194.233 Q2178.53 193.817 2177.67 193.631 Q2176.84 193.423 2175.82 193.423 Q2172.21 193.423 2170.27 195.784 Q2168.35 198.122 2168.35 202.52 L2168.35 216.178 L2164.06 216.178 L2164.06 190.252 L2168.35 190.252 L2168.35 194.28 Q2169.69 191.918 2171.84 190.784 Q2173.99 189.627 2177.07 189.627 Q2177.51 189.627 2178.04 189.696 Q2178.58 189.743 2179.22 189.858 L2179.25 194.233 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip870)" d="M2199.94 202.914 Q2199.94 198.284 2198.02 195.738 Q2196.12 193.192 2192.67 193.192 Q2189.25 193.192 2187.33 195.738 Q2185.43 198.284 2185.43 202.914 Q2185.43 207.52 2187.33 210.066 Q2189.25 212.613 2192.67 212.613 Q2196.12 212.613 2198.02 210.066 Q2199.94 207.52 2199.94 202.914 M2204.2 212.96 Q2204.2 219.58 2201.26 222.798 Q2198.32 226.039 2192.26 226.039 Q2190.01 226.039 2188.02 225.691 Q2186.03 225.367 2184.16 224.673 L2184.16 220.529 Q2186.03 221.548 2187.86 222.034 Q2189.69 222.52 2191.59 222.52 Q2195.78 222.52 2197.86 220.321 Q2199.94 218.145 2199.94 213.724 L2199.94 211.617 Q2198.62 213.909 2196.56 215.043 Q2194.5 216.178 2191.63 216.178 Q2186.86 216.178 2183.95 212.543 Q2181.03 208.909 2181.03 202.914 Q2181.03 196.895 2183.95 193.261 Q2186.86 189.627 2191.63 189.627 Q2194.5 189.627 2196.56 190.761 Q2198.62 191.895 2199.94 194.187 L2199.94 190.252 L2204.2 190.252 L2204.2 212.96 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip870)" d="M2235.15 202.15 L2235.15 204.233 L2215.57 204.233 Q2215.85 208.631 2218.21 210.946 Q2220.59 213.238 2224.83 213.238 Q2227.28 213.238 2229.57 212.636 Q2231.89 212.034 2234.16 210.83 L2234.16 214.858 Q2231.86 215.83 2229.46 216.34 Q2227.05 216.849 2224.57 216.849 Q2218.37 216.849 2214.73 213.238 Q2211.12 209.627 2211.12 203.469 Q2211.12 197.104 2214.55 193.377 Q2218 189.627 2223.83 189.627 Q2229.06 189.627 2232.1 193.006 Q2235.15 196.363 2235.15 202.15 M2230.89 200.9 Q2230.85 197.405 2228.92 195.321 Q2227.03 193.238 2223.88 193.238 Q2220.31 193.238 2218.16 195.252 Q2216.03 197.266 2215.71 200.923 L2230.89 200.9 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /><path clip-path="url(#clip870)" d="M2246.35 182.891 L2246.35 190.252 L2255.13 190.252 L2255.13 193.562 L2246.35 193.562 L2246.35 207.636 Q2246.35 210.807 2247.21 211.71 Q2248.09 212.613 2250.75 212.613 L2255.13 212.613 L2255.13 216.178 L2250.75 216.178 Q2245.82 216.178 2243.95 214.349 Q2242.07 212.497 2242.07 207.636 L2242.07 193.562 L2238.95 193.562 L2238.95 190.252 L2242.07 190.252 L2242.07 182.891 L2246.35 182.891 Z" fill="#000000" fill-rule="nonzero" fill-opacity="1" /></svg>




Actually a pretty fit! If you have higher requirements, check out the *Optim.jl* library.


```julia
unloadFMU(fmu)
```

### Summary

This tutorial showed how a parameter (and start value) optimization can be performed on a FMU with a gradient free optimizer. This tutorial will be extended soon to further show how convergence for large parameter spaces can be improoved!