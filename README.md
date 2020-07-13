
# pySingleCellNet Vignette

## Introduction 
This is a python package for singleCellNet that accomplishes the same tasks as the original version written in R and can be found at https://github.com/pcahan1/SingleCellNet/.

PySingleCellNet makes use of the following packages:

- pandas
- numpy
- sklearn
- scanpy
- sklearn
- statsmodels
- scipy
- matplotlib
- seaborn
- umap-learn

Make sure these packages are installed on your machine prior to using PySingleCellNet.

## Installing pySCN


```python
!pip install git+https://github.com/pcahan1/PySingleCellNet/
```

## Importing pySCN, pandas and numpy


```python
import pySingleCellNet as pySCN
import pandas as pd
import numpy as np
```

## Load in training data 
#### (subset of Tabula Muris data is used here, available at https://github.com/pcahan1/SingleCellNet/)
#### *can also extract data from AnnData object using makeExpMat and makeSampMat functions


```python
tmData=pd.read_csv("tm10xSubExpDat.csv", index_col=0)
tmSampTab=pd.read_csv("tm10xSubSampTab.csv", index_col=0)
tmSampTab.newAnn= tmSampTab.newAnn.astype(str)
```

## Split data into training and validation


```python
expTrain, expVal = pySCN.splitCommon(expData=tmData, ncells=100,sampTab=tmSampTab, dLevel="newAnn")
stTrain=tmSampTab.loc[expTrain.index,:]
stVal=tmSampTab.loc[expVal.index,:]
```    

# Train pySCN


```python
[cgenesA, xpairs, tspRF]= pySCN.scn_train(stTrain = stTrain, expTrain = expTrain,
                   nTopGenes = 10, nRand = 70, nTrees = 1000,nTopGenePairs = 25, dLevel = "newAnn", stratify=True)
```

## Apply to held-out Data


```python
classResVal= pySCN.scn_predict(cgenesA, xpairs, tspRF, expVal, nrand = 0)
```

## Assess Classifier


```python
tm_heldoutassessment = pySCN.assess_comm(classResVal, stTrain, stVal, resolution = 0.005, nRand = 0, dLevelSID = "cell", classTrain = "newAnn", classQuery = "newAnn")
```

## Generate classifier assessment plots


```python
pySCN.plot_PRs(tm_heldoutassessment)
```


![png](md_img/output_17_0.png)



```python
pySCN.plot_metrics(tm_heldoutassessment)
```


![png](md_img/output_18_0.png)


## Plot the classification clustermap, violin plot and attribution plot 


```python
pySCN.sc_hmClass(classResVal.loc[stVal.index,:], stVal["newAnn"],cRow=False,cCol=False)
```


![png](md_img/output_20_0.png)



```python
pySCN.sc_violinClass(stVal, classResVal, threshold=0.5, dLevel="newAnn", ncol=4 )
```


![png](md_img/output_21_0.png)



```python
pySCN.plot_attr(classResVal, stVal, "newAnn")
```


![png](md_img/output_22_0.png)


## Assign classification from pySCN, add it to sample table and plot UMAP of validation data colored by classifcation category 


```python
stVal = pySCN.get_cate(classResVal, stVal, cThresh = 0)
pySCN.plot_umap(expVal, stVal, dLevel="category")
```


![png](md_img/output_24_0.png)



```python

```
