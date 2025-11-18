---
title: "Fluid Substitution"
date: 2025-11-14
permalink: /posts/fluid-substitution/
tags:
  - rock physics
  - CO2 storage
  - Suphanburi Basin
---
 

**Kdry/Kmin vs. Porosity** cross plot showing an inverse relationship between porosity and the dry bulk modulus ratio, indicating decreasing rock stiffness with increasing porosity.

### Reference
<figure style="text-align:center;">
  <br/><img src='/images/crossplot1.png'> 
  <figcaption>Kdry/Kmin – Porosity (Abe et al., 2018)</figcaption>
</figure>

### Result
<figure style="text-align:center;">
  <br/><img src='/images/crossplot2.png'> 
  <figcaption>Applied code from <code>10_subplot_03.py</code></figcaption>
</figure>
``` python
import pandas as pd
import matplotlib.pyplot as plt

#read data from CSV file
df = pd.read_csv("Kdry_Kmin_vs_Porosity.csv")

#calculate Kdry/Kmin ratio
df["K_ratio"] = df["Kdry"] / df["Kmin"]

#create scatter plot
fig, ax = plt.subplots (figsize=(6,5))
sc = ax.scatter(df["Porosity"], df["K_ratio"],s=40, c=df["RHOB"], cmap="turbo", edgecolors="none")
ax.set_xlabel("Porosity (fraction)")
ax.set_ylabel("Kdry / Kmin")
ax.set_title("Cross Plot: Kdry/Kmin vs. Porosity")
ax.grid(True)

#add color bar
cbar = plt.colorbar(sc, ax=ax)
cbar.set_label("RHOB (g/cc)")

plt.tight_layout()
plt.show()
```
