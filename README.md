# Restoration Function

Authors:  
Felipe Sodré Mendes Barros  
Renato Crouzeilles  
Julia Niemeyer  

Function in development in R to simulate restoration in different properties maximizing or minimizing different themes (e.g.: opportunity cost).

## 1. preAnalysis function  
  
Function developed to estimate amout of forest debt/credit in each property analysed, according to Brazilian Native Vegetation Law [12.651/12](http://www.planalto.gov.br/ccivil_03/_ato2011-2014/2012/lei/l12651.htm).
  
```
> source("restoration.function.R")
> args(preAnalysis)
function (forest_t = forest_t, 
appt = appt, 
car = carList[[4]],
restYears = 10)
```
  
#### preAnalysis input:  
  
* forest_t = Actual forest remnants (raster layer: 1 = forest; 0 = non forest; NA = non forest and non restorable sites [e.g.: Urban Areas] & areas outside the site of study);  
* appt = Area of Riparian Protection (Área de Proteção Permanente, in portuguese) (raster layer: 1 = Riparian areas; 0 = non riparian areas; NA = areas outside the site of study);  
* car = Rural Environmental Registry (Cadastro Ambiental Rural - CAR, in portuguese) (vector layer with property limits, with "ID" and "Car_modulo_fiscal" *fields*)  
* restYears = Number of years to be simulated in the analysis. This will change the amount to be restored yearly

#### preAnalysis output:  
  
Data frame:  

* car.area: Area of property
* f.total: Area of forest remnant
* app.area: Area of APP
* debt.app: Debit of APP (i.e.: Amount of pixels to be restored)
* rl.area: Area of Legal Reserve inside the property (discounting APP)
* debt.rl: Debit of Legal Reserve (in case of)
* credit.rl: Credit of Legal Reserve (in case of)
* TotalToRestore: Total amount to be restored (in pixels)
* ToRest2years: Total to be restored  (in pixels) considering 20 years of restoration and that the restoration will be done according to parameter `restYears`.
* Car.ID: Property ID
* Modu_Fisc: Property modulo fiscal
* obs: Observation about property: wether it is all forested, unforested, not_analized, ...  
  
```
head(debito)


---------------------------------------------------------------------------
 X   Car_ID   Car_modulo_fiscal   car.area   f.total   app.area   debt.app 
--- -------- ------------------- ---------- --------- ---------- ----------
 1     34            50             683        240        25         16    

 2     42            89             1212       546        49         15    

 3     43            75             1020       177       144        113    

 4     49            68             931        269        27         26    

 5     76             9             122        58         0          0     

 6     79            27             374        224        0          0     
---------------------------------------------------------------------------

Table: Table continues below

 
------------------------------------------------------------------------
          obs            rl.area   debt.rl   credit.rl   TotalToRestore 
----------------------- --------- --------- ----------- ----------------
Property with RL credit   111.6       0        119.4           16       

Property with RL credit   193.4       0        318.6           15       

Property with RL credit    60         0         86            113       

Property with RL credit   159.2       0        108.8           26       

Property with RL credit   24.4        0        33.6            0        

Property with RL credit   74.8        0        149.2           0        
------------------------------------------------------------------------

Table: Table continues below

 
-----------------
 ToRestore2years 
-----------------
        1        

        1        

       11        

        2        

        0        

        0        
-----------------
```
  
## 2. run.conefor function  
  
Function developed to run the [conefor](http://www.conefor.org/) analysis after each year of restoration;
  
```
> args(run.conefor)
function (dispersal.dist = c(100, 1000, 5000), 
nodes = "nodes_, 
dists = "dists_", 
landArea = 1e+05, 
outFolder = ".", 
windows = FALSE, 
conefor.exe = "/home/felipe/Projetos/CONEFOR/conefor2.7.3Linux", 
mean.dispersal = 0.5, metric = c("-IIC", "-PC")[2],
AddNodes = FALSE) 
```  
  
#### run.conefor input:  
  
* dispersal.dist = Dispersan ditances to be used in the connectivity simulation;  
* nodes = List of files with nodes ID and patch parameter (area) (tow columns data: ID, Parameter);  
* dists = List of files with distances between nodes (tree columns data: ID origin, ID destiny, Distance);  
* landArea = Landscape area;
* outFolder = Folder name where the result will be saved in;
* windows = TRUE/FALSE indicating if using windows O.S. (**Not working now**);
* conefor.exe = Path to conefor executable;
* mean.dispersal = Mean dispersal capacity;
* AddNodes = TRUE/FALSE indicates when conefor should estimates its index based on nodes added (see coneforPrep function for more infos);

#### run.conefor output:  
  
##### Overall_indices table  
  
```
----------------------
     V1         V2    
------------ ---------
   IICnum    6.562e+16

  EC(IIC)    256156000

    IIC       0.5605  

IICintra(%)    96.88  

IICdirect(%)   2.198  

 IICstep(%)   0.9259  
----------------------
```
  
##### results_all_EC(IIC) or results_all_EC(PC)  
  
```
-----------------------------------
    Prefix      Distance   EC.IIC. 
-------------- ---------- ---------
10__run_1.txt      10     252122000

10__run_10.txt     10     252123000

10__run_2.txt      10     252122000

10__run_3.txt      10     252122000

10__run_4.txt      10     252122000

10__run_5.txt      10     252123000
-----------------------------------

```
  
##### results_all_overall_indices  
  
```
------------------------------------------
     V1        V2       V3          V4    
------------- ---- ------------ ----------
10__run_1.txt  10     IICnum    6.357e+16 

10__run_1.txt  10    EC(IIC)    252122000 

10__run_1.txt  10      IIC        0.543   

10__run_1.txt  10  IICintra(%)     100    

10__run_1.txt  10  IICdirect(%)     0     

10__run_1.txt  10   IICstep(%)  -2.853e-09
------------------------------------------
```
  
## 3. restore function  
  
Function used to alocate the amout of area to be restored.
  
```
> args(restore)
function (sorted = sorted, carRestCost = carRestCost, totalcost = totalcost, s.value = ToRest2years[a, ]) 
```  
  
#### restore input:  

* sorted = The pixels sorted in the function, according to the amount necesssary to be restored;
* carRestCost = raster created in the script with opportunity cost (or any other theme) in the area to be restored;
* totalCost = Raster datainput necessary to run the function. This layer must contain the opportunity cost (or any other theme) for each pixel;
* s.value = Amount to be restored (sampled) by function;  

#### restore output:
  
* restoration = Raster data with the forest remnant input plus pixels restored with the simulation;
* RestSampled = Raster data with only the pixels restored with the simulation;
* TotalRestCost = Sum of pixels values sampled in the simulation process (the information of this values can change according to the theme used in the simulation. e.g.: opportunity cost);
    

## change.dir function  
  
Small function to manage output files and directories. In `from` argument must be inserted path where the files will be change from whereas `to` should be the path to the folder where the files must be changed to.  
  
```
> args(change.dir)
function (from, to) 
NULL
```
  
## coneforPrep function  
  
Function to create the nodes and distances files before running conefor (run.conefor()).
This function extract from a raster file the reamnants (nodes) ID and area saving them as nodes_\*.txt, and estimates the distances between them, which will be saved as dists_\*.txt;
 
```
> args(coneforPrep)
function (forest = forest_t, AddNodes = FALSE, onlyAdded = TRUE, 
    i = 1)
```
  
#### coneforPrep input:  

* forest = The spatial raster file with remants (values = 1);
* AddNodes = Logical - This function can be used to identify nodes to be added. If this is the case, set as TRUE (default = FALSE);
* onlyAdded = Logical - Indicates whther or not actual forest (with values 1) must have its importances calculated togheter with non forest (nodes added). If TRUE, forest pixels will have -1 values which won't be analysed on run.conefor(AddNodes = TRUE);
* i = Integer - As the function can be called inside restoration iteraction, the index *i* will be used to diferenciate the iteraction number in the file names;

#### coneforPrep output:
  
* nodes_ = Files containing remants ID and Area. In case of AddNodes = TRUE, a third column will indicate 1 = existing remnants and 0 = potential remnants;
* dists_ = Files containing distances between all nodes. First column origin ID, second column destiny ID third column distance;
    

## readImportance function

Function created to read all node importance created with run.conefor (Addnodes=TRUE) for one or several dispersal distances and produce spatial raster with node importance as pixel value.


```
> args(readImportance)
function (rasterTemplate = forest_t, nodes = readOGR("/home/felipe/Projetos/Funcao_Restauracao/", 
    layer = "forest_pol_AddNodes_run_1"), dispersal.dist = c(100, 
    1000, 5000), Indice = c("dIIC", "dPC")[1])
```

## readImportance input

* rasterTemplate = Raster data to be used as a template in the creation of node importance raster (extent, resolution, CRS);
* nodes = Shape file created with run.conefor(Addnodes="TRUE") which will recieve the importance estimated and will be rasterized;
* dispersal.dist = Dispersal distances used to estimate the nodes importante in run.conefor function;
* Indice = Indice used to estimate nodes importances in run.conefor function;

## readImportance output

* PC_importance_ = Raster with nodes importance for each pixel, indicating the dispersal distance in the name (sufix);

