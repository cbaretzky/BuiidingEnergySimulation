
# BuildingEnergySimulation (BES)
<img src="docs/_IMAGES/Logo.png" width="100"> __

BuildingEnergySimulation (BES) is a OpenSource project to simulate energy flow and storage inside a building.

## Dependencies

BES requires the following python packages:

    geocoder, geopy, matplotlib, numba, numpy, pandas, pvlib, requests, scipy

## Installation

Install the dependencies and copy the package into a Python user site directory. Unfortunately it is not yet available on PyPI

## Quickstart

```python
import BuildingEnergySimulation as bes
building= bes.Building(loc="Markt, 52062 Aachen")
```
Will generate the building as a frame for all components.

The next step is the add the relevant components to the building:

Adding a wall to the Building can be done as following:
```python
bes.Wall.reg(building, bes.DEFAULT_LAYERS['Brick_Granite'], area=400)
```
the layers for walls and windows are lists of dictionary containing the relevant phyical parameters to simulate the thermal properties in one Dimensions:

The "Brick_Granite" layer from the integrated sample layers is defined as

```python
[{
'c_v': 1000.0, #Heat capacity
'lambda': 0.79, #Heat resistance
'name': 'Brick', #Name
'rho': 1800.0, #Density
'thickness': 0.3}, #Thickness of the layer
{
'c_v': 1000.0,
'lambda': 2.80,
'name': 'Granite',
'rho': 2600.0,
'thickness': 0.4}],
```

the area defines the wall area in square meters

To add a window it is neccessary to add the orientation. Azimuth = 0 means the window is facing south, 90 means the window is facing west and -90 means the window is facing east.
```python
bes.Window.reg(building, area= 20, azimuth=0)
```
Adding a Heatpump modeled after one specific groundwater based model
```python
bes.Heatpump.reg(building)
```
Adding a simplified solar-pv panel with 14 kWp and a battery with 14kWh capacity and a powergrid connection:
```python
bes.Solar_pv_simple.reg(building, kwp = 14,azimuth = 0, inclination=35)
bes.Battery.reg(building, capacity=14*3.6e6)
bes.Grid.reg(building)
```

Finally the last components need to be connected with each other. The solar panel and heatpump run through the battery, the battery is connected to the grid.

```python
battery = building.get_component('Battery')[0]
battery.connection_in.append(building.get_component('Solar_pv_simple'))
battery.connection_in.append(building.get_component('Heatpump'))
grid = building.get_component('Grid')[0]
grid.connection_in.append(battery)
```

Now the simulation form January first to January fifth of 2007 can be run as follows
```python
building.simulate('2007-1-1', '2007-1-5')
```
When the processing is finished
```python
building.sim_results
```
contains a pandas Dataframe with all relevant quantities through the simulation timeframe and can be written to disk using the functionality integrated into pandas
## How it works

The idea is to model the energy flow as a directed graph from the given environmental factors (outer temperature, solar irradiane) to the available energy sources (Grid, Fuel).

<img src="docs/_IMAGES/Graph.jpeg" width="1000"> __


Physically speaking the simulation is very inaccurate, however it is balanced around how much information about buildings is typically available and the significant influence of inhabitant habits. For example there is no radiator model. But the heat transfer from a underfloor radiator depends on the position and quantity of furniture in the room. It is much easier to assume a given temperature that the room is supposed to have, compared to of solving a differential equation which greatly depends on individual factors which are rarely accessible.

The main goal is to simulate the energy flows on a timescale that can account for the variability of renewable energies.

## Comments

This is a "rough around the edges" project, that started and still is a repository of codefragments I used.

The PVGIS data is hardcoded to the year 2007 to keep the load on the PVGIS API low. However the API provides data for the years 2005 through 2015 which can be changed in the PVGIS class .
