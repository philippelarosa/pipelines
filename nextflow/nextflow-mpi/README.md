---

title: "MPI via nextflow"
author: "Philippe La Rosa"
team: "HPC benchmark/conception"
date: "02 May 2019"
output: html_document

---

Quick start guide
-----------------

Project description
-------------------

- Type : "Proof of concept" 
- Title : "Intégration de codes MPI dans un pipeline sous nextflow ou choix de la meilleure stratégie à adopter"
- Descriptions : "Design des stratégies de tests, maquettage et lancement des tests du workflow".

- Bioinformaticians contacts: Philippe La Rosa (philippe.larosa@curie.fr), Frédéric Jarlier (frederic.jarlier@curie.fr)

Dependencies
------------

- nextflow (/bioinfo/local/build/Centos/nextflow/nextflow-19.04.0.5069/nextflow)
- singularity (at least the version 2.6.1)
- mpi : mpirun
- mpiBWA project (Frederic Jarlier) : https://github.com/fredjarlier/mpiBWA (branch:Experimental)


Prerequisites
------------
- Pour l'ensemble des tools sous MPI
    - le lancement s'effectue via mpirun qui est un tool openmpi sous Centos, mpirun est installé sous : /bioinfo/local/build/Centos/openmpi/openmpi-3.0.0/bin

- installation outils mpiBWA project (Frederic Jarlier) sous Centos
    - la première étape : installer les tools BWA sous mpi (pbwa7, pidx) selon :  https://github.com/fredjarlier/mpiBWA (branch:Experimental)
    créér par Frederic en local c.a.d dans le répertoire "bin" accessible du futur pipeline nextflow de benchmark .
    - la deuxième étape : faire installer les tools BWA sous mpi selon :  https://github.com/fredjarlier/mpiBWA (branch:Experimental)
    créér par Frederic sous Centos dans répertoire canonique c.a.d /bioinfo/local/build/Centos/mpiBWA. A noter que d'anciennes installation existent, mais il est préférable de faire cette mise à jours installation; accessiblilité du futur pipeline nextflow de benchmark via la configuration.

- installation annotation (fa.map) pour mpiBWA project (Frederic Jarlier) sous Centos
    - pour fonctionner pbwa7 (BWA sous mpi) à besoin d'un fichier d'annotation sous la forme d'une map pour une utilisation en mémoire partagée. Création via le tool pidx; il est souhaitable de faire générer cette ressource d'annotation automatiquement dans le pipeline nextflow via un process.

- installation outils mpiBWA project sous Centos dans une image singularity
    - une image singularity est à créer pour les test via les containers, voir fichier de recette deja créé par Frederic.


Strategies
-------------------

- 1) Soit c'est nextflow qui appel MPI via les process nextflow pour les cas ci-dessous :

    - 1.1) dans un simple pipeline sous nextflow qui appel directement un des outils spécifiques déjà implémentés sous mpi (ex : mpiBWA).
        - avec et sans images singularity

    - 1.2) à partir d'un pipeline existant (nfcore) sous nextflow plus complexe, avec appel d'un ou plusieurs outils spécifique déjà implémentés sous mpi (ex : mpiBWA)
      
    - 1.3)  à partir du pipeline EWOK sous nextflow beaucoup plus complexe, avec appel d'un ou plusieurs outils spécifique déjà implémentés sous mpi (ex : mpiBWA)
    
Dans ces 3 cas 

        - implique, le calcul des ressources nécessaires à réserver à torque (mémoire, procs, ...)
        - appel tools sous MPI directement installés sous Centos ou dans une image singularity: utilisation env spécifique
        - permet la comparaison perfs pures avec ou sans MPI : implémentaion option spécifique
        - permet la comparaison perfs pures versus ressources (trouver le bon compromis)


- 2) Soit c'est MPI qui lance nextflow qui appel ensuite directement des outils spécifique déjà implémentés sous mpi (ex : mpiBWA):

    - 2.1) dans un simple pipeline sous nextflow qui appel directement des outils spécifique déjà implémentés sous mpi (ex : mpiBWA).
    
Ce cas doit permettre de bien apprehender comment sont allouées les ressources nécessaires à réserver à torque (mémoire, procs, ...).

Pour ce cas d'usage, a noter que selon la documentation de nextflow, des exemples de lancements existent seulement pour les gestionnaires de ressources : LSF, Grid Engine, SLURM
A tester sous torque et/ou sous slurm lorsque nous aurons accès au nouveau cluster
mpirun --pernode nextflow run <your-project-name> -with-mpi [pipeline parameters]
 


How to install it ?
-------------------

- Clone singularity_via_nextflow repository

```shell
git clone ssh://git@gitlab.curie.fr:2222/plarosa/nextflow-mpi.git
```

How to use it ?
---------------


## Run the pipeline nextflow/MPI directly or with singularity and all on CentOS cluster:
### case 1

```shell
ssh calcsub 
cd ${your_local_repo_GIT}/nextflow-mpi/strategie_1.1

echo "cd $(pwd); bash run_nextflow-mpi_strategie_1.1.bash" | qsub -q diag -N nextflow-mpi_strategie_1.1

```

## Run the pipeline MPI/nextflow directly or with singularity and all on CentOS cluster:
### case 2

```shell
ssh calcsub 
cd ${your_local_repo_GIT}/nextflow-mpi/strategie_2.1

echo "cd $(pwd); bash run_nextflow-mpi_strategie_2.1.bash" | qsub -q diag -N nextflow-mpi_strategie_2.1

```

Understanding the results
-------------------------

- results/pipeline
    - strategiexx-nf-flowchart.pdf
    - strategiexx-nf-report.html 
    - strategiexx-nf-timeline.html 
    - strategiexx-nf-trace.csv
- results/tools/bwa
	- strategiexx-bwa-nf_stats.csv
