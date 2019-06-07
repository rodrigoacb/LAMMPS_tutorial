# Tutorial de LAMMPS

## LAMMPS: Download e instalação

O Large-scale Atomic/Molecular Massively Parallel Simulator [(LAMMPS)](https://lammps.sandia.gov/) não enconta-se disponível no
repositório base do ubuntu, por isso faz-se necessário a adição via ppa a partir dos seguintes comandos:

```
    sudo add-apt-repository ppa:gladky-anton/lammps
    sudo apt-get update
    sudo apt-get install lammps-daily
```
Como alternativa também é possível instala-lo a partir do código fonte:

```
    wget https://lammps.sandia.gov/tars/lammps-9Nov18.tar.gz
    tar -xzvf lammps*.tar.gz
    cd lammps*
    mkdir build
    cd build 
    cmake ../cmake
    make
    make install
```

Pronto! agora provavelmente você já pode verificar se o lammps esta instalado no seu sistema. Se optou pela primeira instalação pode
digitar `lmp-daily` e verificar se o LAMMPS foi instalado, caso tenha escolhido a segunda tente `lammps` ou `lmp`. Para liberar o
terminal pressione `Ctrl+c`

#### Input file

Neste tutorial faremos o input com três arquivos, primeiro um com extensão `.data` que irá conter informações sobre a topologia dos
átomos. Como estudo de caso faremos uma molécula de butano.

Primeiro define-se quantos átomos, ligações, angulos e diedros são necessários para definir a molécula assim como quantos "tipos" de
cada.

```
Data file for TraPPE -UA Butane

    4 atoms
    3 bonds
    2 angles
    1 dihedrals

    2 atom types
    1 bond types
    1 angle types
    1 dihedral types

```

No caso do butano tem-se `4 atoms` que representam 4 sítios *coarse-grain* (2 CH<sub>3</sub> e 2 CH<sub>2</sub>) 3 ligações, 1 ângulo e
1 diedro. o próximo passo é descrever as dimensões da caixa em que representaremos a molécula.

```
    0.0 6.0 xlo xhi
    0.0 6.0 ylo yhi
    0.0 6.0 zlo zhi
```

Então definem-se as massas:
```
    Masses
    1 15.035
    2 14.027
```

A posição espacial dos átomos e suas ligações ângulos e dihedros. Uma descrição detalhada do que significa cada coluna do
[.data](https://lammps.sandia.gov/doc/2001/data_format.html) já está documentada.

```
Atoms

1 1 1 0.0 1.000  1.000  1.000
2 1 2 0.0 2.540  1.000  1.000
3 1 2 0.0 3.090  2.440  1.000
4 1 1 0.0 4.640  2.440  1.000

Bonds

1 1 1 2
2 1 2 3
3 1 3 4

Angles

1 1 1 2 3
2 1 2 3 4

Dihedrals

1 1 1 2 3 4
```

O segundo arquivo irá conter dados do potencial inter e intramolecular. O potencial intermolécular utilizado será o de Lennard-Jones
com raio de corte  e correção de longo alcance. Os parâmetros de coeficiente podem ser retirados do artigo original do TraPPE.

```
    pair_style  lj/cut 14.0
    pair_modify tail yes mix arithmetic

    pair_coeff  1 1 0.195 3.75
    pair_coeff  2 2 0.091 3.95
```

O mesmo pode ser feito pode ser feito para as ligações, ângulos e diedros:
```
    bond_style  harmonic
    bond_coeff  1 268.0 1.54

    angle_style harmonic
    angle_coeff 1 62.1 114.0

    dihedral_style  opls
    dihedral_coeff  1 1.411 -0.2710 3.145 0.000
```
Por último, agora que temos a configuração completa, precisamos estabelecer parâmetros da simulação em si:

```
# TraPPE UA Ethane

# Initialization
units       real
atom_style  full

variable    NRUN    equal 10000
variable    VPRES   equal 100.0
variable    VTEMP   equal 300.0
variable    AVOGAD  equal 6.0221409E23
variable    A3TOM3  equal 1E-30
variable    GTOKG   equal 1E-3
variable    DCONV   equal ${GTOKG}/${AVOGAD}/${A3TOM3}
variable    DENSITY equal mass(all)*${DCONV}/vol

# Setup simulation box
boundary    p p p
read_data   but.data
include     but.ff
replicate   10 10 10

# Setup atoms
velocity    all create ${VTEMP} 352450618

# Settings
timestep    2.0
neighbor    2.0 bin
neigh_modify    every 1 delay 0 check yes

# Operations
fix     integrate all npt temp ${VTEMP} ${VTEMP} 100.0 iso ${VPRES} ${VPRES} 1000.0
fix     dens all ave/time 1 1 1 v_DENSITY file dens.dat

# Output
thermo      10
thermo_style    custom step temp press v_DENSITY etotal
dump        myDump all xyz 10 out.xyz

# Actions
run     ${NRUN}
```

