# Tutorial de LAMMPS

## LAMMPS: Download e instalação

O Large-scale Atomic/Molecular Massively Parallel Simulator [(LAMMPS)](https://lammps.sandia.gov/) não enconta-se disponível no repositório
base do ubuntu, por isso faz-se necessário a adição via ppa a partir dos seguintes comandos:

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

Pronto! agora provavelmente você já pode verificar se o lammps esta instalado no seu sistema. Se optou pela primeira instalação pode digitar
`lmp-daily` e verificar se o LAMMPS foi instalado, caso tenha escolhido a segunda tente `lammps` ou `lmp`. Para liberar o terminal pressione
`Ctrl+c`

#### Input file



