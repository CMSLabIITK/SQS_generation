<p align="justify">
A special quasirandom structure (SQS) is a mathematical model to generate structures for disordered alloys. They exhibit certain statistical properties similar to those of a randomly generated alloy but with some underlying regularity or pattern. The SQS can mimic the disordered behavior in a single configuration in a small number of atoms (~100 atoms for five components). SQSs are often used to model disordered alloys because they can provide a good approximation of real alloys while being computationally less expensive to simulate than truly random structures. Apart from this, it can also capture the effect of the local environment and hence can be used to study local lattice distortions in multi-principal element alloys.
</p>

The following tutorial covers the basics of generating SQS structures using the Alloy Theoretic Automated Toolkit (ATAT).<br>


## Step 1:
Create the smallest unit cell and name it as `rndstr.in` . Its format should be of the following form

First we need to specify the shape of the cell which can be done by:
```
[a] [b] [c] [alpha] [beta] [gamma] 
```
Here a,b,c are length, breadth and height of cell while the alpha,beta and gamma defines the angles between them. 
Another way we can define cell shape by using unit cell cell vectors as
```
[ax] [ay] [az]
[bx] [by] [bz] 
[cx] [cy] [cz] 
```
Then we need to specify the lattice vectors u,v,w of the unit cell which are expressed as:
```
[ua] [ub] [uc] 
[va] [vb] [vc] 
[wa] [wb] [wc]
```
Finally, atom positions and types are given, expressed in the same coordinate system as the lattice vectors:
```
[atom1a] [atom1b] [atom1c] [atomtype11]=[occupation11],[atomtype12]=[occupation12],... 
[atom2a] [atom2b] [atom2c] [atomtype21]=[occupation21],[atomtype22]=[occupation22],.. 
```
<br>
<br>

An example of the same for FCC structure for binary alloy considering lattice parameter as one and taking fraction of Fe and Mn as 0.96875 and 0.03125, respectively is shown below. <br>

```
1.0 1.0 1.0 90 90 90 
0.5 0.5 -0.5 
0.5 0.5 0.5 
0.5 -0.5 0.5 
0.0 0.0 0.0 Fe=0.96875,Mn=0.03125
```
<br>
<br>


## Step 2:
Then we need to generate the cluster information. For this we can use the "CORRelation DUMPer" command `corrdump`. It takes `rndstr.in` as an input which can be specified as `-l=rndstr.in`. Cluster information from pairs triplets... upto sextuplet can be generated using `-2 -3` ... upto `-6`, and for each cluster, maximum distance between two points needs to be specified. For example, if we use `-2=1.1`, then here `2` denotes the pair cluster while 1.1 denotes the maximun neighbour distance which means that if in the `rndstr.in` the lattice is specified as FCC and lattice parameter as 1, it will consider FCC first and second nearest neighbor distance ie. 0.707 and 1.0 respectively, as 1.1 is the maximum neighbor distance. Usually, for most of the alloys, clusters upto quadruplet(`-4`) and upto third nerest neighour distance is sufficient for most calculations. Basic usage of the command is given below. <br>

<code>corrdump -l=rndstr.in -ro -noe -nop -clus -2=1.1 -3=1.1 -4=1.1 </code>
<br>

The above example shows the generation of cluster taking `rndstr.in` as input. `-ro` asks them to read occupancies from the file. The `-noe` and `-nop` asks the program not to include empty and point clusters, respectively. The `-clus` command is used to generate the cluster. 
<br>
<br>


## Step 3:
For generating SQS of specific size and shape, we need to create `sqscell.out`. The format for this is given below <br>

```
[No. of structures] 

[str1_a1] [str1_a2] [str1_a3] 
[str1_b1] [str1_b2] [str1_b3] 
[str1_c1] [str1_c2] [str1_c3] 

[str2_a1] [str2_a2] [str2_a3] 
[str2_b1] [str2_b2] [str2_b3] 
[str2_c1] [str2_c2] [str2_c3] 
.... 
```

Here the first row takes the number of structures. For the single phase, it will be 1, while for multiphase, it will be >1. The next three lines give the transformation matrix for structure 1; the further three lines give the same for structure 2, and so on. An example of a single FCC phase is given below, which creates the supercell of size 4x5x2 using this transformation. 

```
1 

4 0 0
0 5 0
0 0 2
```
<br>
<br>


## Step 4:
 The last step is the generation of SQS, which is carried out using `mcsqs` command. This generates the file `bestsqs.out` containing the structure information and the `bestcorr.out` containing information about the correlation function. The correlation function can be used to judge how much disordering is present in the structure. A simple usage of the command is given below.

```
mcsqs -rc
```

Here `-rc` tells the command to read supercells from file `sqscell.out`. 

Usually, SQS generation takes a fairly large amount of time. In order to speed up its generation, several instances can be run using `-ip=[integer]`. An example of the same is given below.

```
mcsqs -rc -ip=1 &
mcsqs -rc -ip=2 &
...
```

This above command will create two instances with id 1, 2... and `&` command is used for running it in background. Running the above command gives several bestsqs.out files like `bestsqs1.out`, `bestsqs2.out`... Each structure has a different correlation function given in `bestcorr1.out`, `bestcorr2.out`... In order the select the best structure among them we can use

```
mcsqs -best
```
<br>

This will create a file `bestsqs.out` containing the most disordered structure among the list of generated structures.
<br>
<br>

## Step 5:
The structure generated can now be used for the DFT run. However, before this, it needs to be converted, which can be read in the DFT software; for example, for VASP, we need to convert this in the POSCAR format. For this, I have written a small tool that converts the bestsqs.out to POSCAR. Here is the <a href="https://github.com/albert-hzbn/sqs_to_poscar" target="_blank">link</a> for the same. Don't forget to scale the lattice parameters close to the alloy before calculation as the lattice parameter is considered in the above example as 1. 
