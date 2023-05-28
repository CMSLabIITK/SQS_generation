<p align="justify">
A special quasirandom structure (SQS) is a mathematical model to generate structures for disordered alloys. They exhibit certain statistical properties similar to those of a randomly generated alloy but with some underlying regularity or pattern. The SQS can mimic the disordered behavior in a single configuration in a small number of atoms (~100 atoms for five components). SQSs are often used to model disordered alloys because they can provide a good approximation of real alloys while being computationally less expensive to simulate than truly random structures. Apart from this, it can also capture the effect of the local environment and hence can be used to study local lattice distortions in multi-principal element alloys.
</p>

The following tutorial covers the basics of generating SQS structures using the Alloy Theoretic Automated Toolkit (ATAT).<br>


## Step 1:
Create the smallest unit cell and name it as `rndstr.in` . Its format is explained below:

First we need to specify the shape of the cell, this can be done by:
```
[a] [b] [c] [alpha] [beta] [gamma] 
```
Here a,b,c are length, breadth and height of cell while the alpha, beta and gamma defines the angles between them. 
Another way we can define cell shape is by using unit cell vectors as
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
Finally, atomic positions and types are given, they are expressed in the same coordinate system as the lattice vectors:
```
[atom1a] [atom1b] [atom1c] [atomtype11]=[occupation11],[atomtype12]=[occupation12],... 
[atom2a] [atom2b] [atom2c] [atomtype21]=[occupation21],[atomtype22]=[occupation22],.. 
```
<br>
<br>

An example of FCC structure for binary alloy considering lattice parameter as one and fractional composition of Fe and Mn as 0.5 and 0.5, respectively is shown below. <br>

```
1.0 1.0 1.0 90 90 90 
0.0 0.5 0.5
0.5 0.0 0.5
0.5 0.5 0.0
0.0 0.0 0.0 Fe=0.5,Mn=0.5
```
<br>
<br>


## Step 2:
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
In this format, the first row specifies the number of structures in the system. The following rows contain the transformation matrices for each structure. Each matrix is represented by 3 rows, with 3 values in each row, [str1_a1] [str1_a2] [str1_a3], [str1_b1] [str1_b2] [str1_b3], [str1_c1] [str1_c2] [str1_c3] in the first case. These matrices are used to create supercells of different sizes for each structure. An example of a single FCC phase is given below, which creates the supercell of size 4x5x2 using this transformation. 

```
1 

4 0 0
0 5 0
0 0 2
```
<br>
<br>




## Step 3:
The CORRelation DUMPer (corrdump) command is used to generate cluster information from a given input file, rndstr.in, which specifies the lattice structure and parameters of a material. The input file is specified using the `-l` flag followed by the input file name. The command can generate cluster information for various clusters such as pairs, triplets, quadruplets, and sextuplets by using flags such as `-2`, `-3`, `-4`, and `-6`, respectively. For each cluster, the maximum distance between two points in the cluster can also be specified.

For example, if the lattice structure is FCC and the lattice parameter is 1, and the maximum distance between two points in the cluster is specified as 1.1, then the command will only consider the first and second nearest neighbor distances (0.707 and 1.0 respectively) as valid distances within the cluster when generating the cluster information.

For most alloy calculations, clusters up to quadruplet and distance up to the third nearest neighbor are sufficient.  <br>

<code>corrdump -l=rndstr.in -ro -noe -nop -clus -2=1.1 -3=1.1 -4=1.1 </code>
<br>

The above example shows the generation of cluster taking `rndstr.in` as input. `-ro` asks them to read occupancies from the file. The `-noe` and `-nop` asks the program not to include empty and point clusters, respectively. The `-clus` command is used to generate the cluster. 
<br>
<br>





## Step 4:
The last step in the process is the generation of a SQS using the `mcsqs` command. This command generates two output files, `bestsqs.out` and `bestcorr.out`, containing information about the SQS structure and the correlation function, respectively.

The `bestsqs.out` file contains information about the structure of the SQS, including the atomic positions, coordination numbers, and other relevant structural properties. The `bestcorr.out` file, on the other hand, contains information about the correlation function, which can be used to judge how much disorder is present in the structure. The correlation function is a measure of the similarity between the pair distribution function of the SQS and the pair distribution function of the underlying random alloy. A high correlation function value indicates a high degree of order, while a low correlation function value indicates a high degree of disorder.

An simple example is given below:

```
mcsqs -rc
```

The mcsqs command has an option `-rc` that tells the command to read supercells from file `sqscell.out`.

SQS generation can be a computationally expensive process, and it may take a fairly large amount of time to generate the SQS. In order to speed up the process, the command has an option `-ip=[integer]` which allows to run multiple instances of the mcsqs command at the same time. This option takes an integer as an input, specifying the number of instances that will be run in parallel. An example of how to use this option is:

```
mcsqs -rc -ip=1 &
mcsqs -rc -ip=2 &
...
```

Above command creates multiple instances of the `mcsqs` command with different instance IDs (1, 2, etc.) and runs them in the background using the `&` command. This command creates multiple output files, such as `bestsqs1.out`, `bestsqs2.out`, etc., each containing information about a different SQS structure. The correlation function for each structure is given in the corresponding `bestcorr1.out`, `bestcorr2.out`, etc.

To select the best structure among the generated structures we can use the command

```
mcsqs -best
```

This creates a file called `bestsqs.out` containing the most disordered structure among the list of generated structures. The most disordered structure is generally considered as the best SQS because it has the highest degree of randomness and thus, is more representative of the underlying random alloy. It's also important to note that the criteria of selecting the best SQS can vary depending on the specific use case and the requirements of the calculations. It's recommended to check the correlation function, the number of symmetry inequivalent atom, and other property of the SQS to make a final decision.

<br>
<br>

## Step 5:
Once the SQS structure has been generated, it can be used for Density Functional Theory (DFT) calculations. However, before performing the DFT calculations, the structure needs to be converted into a format that can be read by the DFT software. For example, if you are using the VASP software, you will need to convert the structure into the POSCAR format.

You can use a small tool that converts the `bestsqs.out` file to `POSCAR` format, such as the one provided in this <a href="https://github.com/albert-hzbn/sqs_to_poscar" target="_blank">link</a>. It's important to note that before using the structure for DFT calculations, you should scale the lattice parameters to match the alloy you are studying. In the example provided above, the lattice parameter is set to 1, so you should scale the lattice parameters to match the alloy's lattice parameter before performing the DFT calculations.

It's also worth noting that, depending on the DFT software, you may need to use different units or conventions for the lattice parameters and the atomic positions. Make sure to check the documentation of the DFT software to ensure that the structure is in the correct format for the calculations.
