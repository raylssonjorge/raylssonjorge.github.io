---
  title: "Tutorial Gromacs (Cluster)"
  date: 2020-05-27
  tags: [Chemistry]
  header:
  #  image: "/images/1234.jpg"

  #excerpt: "Physics, Chemistry"
  mathjax: "true"
  classes: wide

---




O cluster Sdumont é operado via terminal em  que você pode logar para  administrar e rodar seus cálculos.



Para fazer o login no sdumont você vai precisar de apenas uma vpn para se conectar a rede privada de lá.Com isso fica melhor para mandar os arquivos de forma remota apenas tendo o acesso a VPN.Segue em anexo um arquivo em pdf de como configurar e se conectar na vpn.

Para logar no sdumont basta se conectar na VPN e  digitar no terminal o seguinte comando :


```
~ ssh -YC -v nome.sobrenome@login.sdumont.lncc.br
```

Ao digitar sua senha do sdumont (senha de usuário passada pela coordenação do cluster) você deve cair no seu diretório Home /prj/NOME_DO_PROJETO/username


-prj é onde todos os projetos do sdumont se encontram
-NOME_DO_PROJETO é o nome do seu grupo
-username  é seu usuário




Nesse diretório do seu usuário você pode manter seus arquivos seguros sempre tendo um backup para prevenir problemas.


O seu diretório do seu usuário é onde você deve manter seus arquivos seguros pois o outro diretório importante (conhecido como scratch) serve para rodar seus cálculos .



### Agora vamos a parte de rodar cálculos.

Digite o seguinte comando no seu terminal (conectado ao sdumont):

```
~ cd /scratch/NOME_DO_PROJETO/username/
```


Para não ficar perdido no cluster recomendo usar o comando pwd (que indica em qual diretório você se encontra).



No /scratch do seu usuário é o lugar onde se rodam seus cálculos.Uma forma simples de fazer isso é usar um script executável em  bash.Se vocẽ quiser pode pesquisar a fundo sobre os scripts em bash mas não irei me aprofundar nesse tutorial.




Os scripts seguem essa forma geral:


```
#!/bin/bash
#SBATCH --nodes=N                  #Numero de Nós
#SBATCH --ntasks-per-node=TPN      #Numero de tarefas por Nó
#SBATCH --ntasks=T                 #Numero total de tarefas MPI
#SBATCH -p FILA                    #Fila (partition) a ser utilizada
#SBATCH -J JOB			           #Nome job
#SBATCH --exclusive                #Utilização exclusiva dos nós durante a execução do job

module purge all		            #Remove todos os módulos ativos para o seu usuário
module load 		            	#carrega um módulo  _do_ seu software
module list		                	#mostra todos os módulos que estão ativos para o seu usuário

exec=/scratch/app/nome_do_software/caminho até o executável

srun $exec  comandos_adicionais
```

O número de nós  é um jeito de dividir os processos para computação. A imagem abaixo mostra as possíveis filas do cluster:



<img src="{{ site.url }}{{ site.baseurl }}/images/filas.png" alt="filas disponiveis no cluster">

As filas são algo importante para saber onde se deve colocar seu Job.Basta ficar atento quanto tempo irá precisar e quais filas são rápidas para rodar seu job.Os módulos são um jeito de você carregar os softwares para serem usados.
Para ver quais softwares estão disponíveis para você usar basta digitar " $  module avail " no seu terminal










## Gromacs


Nota: É necessário estar conectado a VPN do cluster para interagir com ele.

Suponhamos que você tenha um arquivo .tpr do gromacs e queira rodar no sdumont.A primeira coisa que se deve fazer (com a vpn conectada) é  enviar o arquivo para o diretório que quiser rodar o cálculo no /scratch.Isso pode ser feito usando a opção scp.


(esteja no diretorio do arquivo)


```
~ scp file.tpr   nome.sobrenome@login.sdumont.lncc.br:/scratch/NOMEDOPROJETO/username/
```


Agora é necessário entrar com seu usuário no sdumont :


```
~ ssh -YC -v nome.sobrenome@login.sdumont.lncc.br
```



Chegou a hora de ir onde você enviou o arquivo:



```
~ cd /scratch/nomedoprojeto/username/
```

Vamos precisar criar um arquivo para o seu script. Para isso deixe copiado esse código aqui no seu clipboard (use CTRL+C ):


```
#!/bin/bash -l
#SBATCH --nodes=1                      #Numero de Nós
#SBATCH --ntasks-per-node=1            #Numero de tarefas por Nó
#SBATCH --ntasks=1                     #Numero total de tarefas MPI
#SBATCH --cpus-per-task=24      
#SBATCH -p nvidia_dev                  #Fila (partition) a ser utilizada
#SBATCH -J teste                       #Nome job
#SBATCH --error=teste.err
##SBATCH --output=teste.out
#SBATCH --exclusive                    #Utilização exclusiva dos nós durante a execução do job


module purge all
module load gromacs/2018.7_openmpi_gnu
module list

exec=/scratch/app/gromacs/2018.7_openmpi_gnu/bin/gmx_mpi_gpu

srun $exec mdrun -v -deffnm md -nb gpu
```


Esse script usa 24 cpus no total  e usa placa de vídeo dedicada para ajudar nos cálculos.Pode ver que eu usei uma fila que contém gpu ( nvidia_dev).Essa fila serve para testar se o cálculo vai rodar antes de usar as outras filas (a cpu_dev tem o mesmo objetivo) mas o tempo limite para o cálculo é de apenas 20 min 


Crie um arquivo script_gromacs para rodar a simulação no seu diretório /scratch/nome_do_projeto/username/  usando o seguinte comando:



```
~ vim script_gromacs 
```


Ao entrar no editor vim  aperte a letra  i   do seu teclado para entrar no modo de edição do vim no terminal e aperte  ctrl + shift + v  para colar o texto que você copiou.Agora saia do  modo de INSERÇÃO  no vim  ( apertando Ctrl + C ) e use o comando :wq!  (inclua os dois pontos no comando)   para  SALVAR E SAIR  o arquivo. Você pode entrar outra vez no arquivo (usando vim script_gromacs)  para ver se salvou tudo certinho :) .



Tendo os arquivos md.tpr e o script_gromacs no MESMO diretório basta usar sbatch para submeter seu job para as máquinas.


```
~ sbatch script_gromacs
```

Você pode usar o comando $  squeue  --job job_idnumber (No meu caso seria $ squeue --job 474174)  para ver se o seu cálculo ja está rodando ou se houve algum problema.Ou você pode apenas usar $ squeue -u username para ver todos os cálculos que estão rodando utilizando seu usuário.Se houve algum problema você pode tentar diagnosticar usando vim teste.err.Para cancelar o job use $ scancel job_id_number.



Para ver como está andando seu cálculo basta usar :

```
~ tail -f  gromacs_md_name_file.log  
```

Também tem o test.err que você pode ver se ocorreu algum erro quando foi rodar o seu arquivo.tpr. (Use vim teste.err )


Agora basta  mudar para uma fila que tem um tempo maior pois o tempo limite das filas _dev é de 0 :20 (em horas).



## Orca

Suponhamos que você tenha um arquivo.inp (ORCA) e queira rodar no sdumont.Basta seguir os mesmos processos:
1. envie o arquivo usando scp para o /scratch do seu usuário.
2. crie um script para o orca (vim script_orca) .
3. copie e cole um script padronizado para o orca: 



```
#!/bin/bash -l
#SBATCH --nodes=1                   #Numero de Nós
#SBATCH --ntasks-per-node=1         #Numero de tarefas por Nó
#SBATCH --ntasks=1                  #Numero total de tarefas MPI
#SBATCH --cpus-per-task=20    
#SBATCH -p cpu_dev                  #Fila (partition) a ser utilizada
#SBATCH -J sds                      #Nome job
#SBATCH --error=teste.err
##SBATCH --output=teste.out
#SBATCH --exclusive                 #Utilização exclusiva dos nós durante a execução do job

module purge all
module load orca/4.0.1              #Seleciona a versão do orca desejada e carrega o módulo
module list		                    #Mostra os módulos ativos para o seu usuário 

EXEC=/scratch/app/orca/4.0.1/orca


srun $EXEC SDS_linear.inp  > SDS_linear.out
```


Veja que você PRECISA  editar o nome do arquivo que o orca vai executar.(No meu caso estou querendo rodar o SDS_linear.inp gerando um SDS_linear.out.)



4. Execute o comando:

```
~ sbatch script_orca
```

5. edite o nome da fila que quer usar (uma fila que usa apenas cpu (cpu , cpu_long, cpu_shared )

6. use $ squeue -u username para verificar se está rodando  / verifique o .out usando $  tail -f file_name.out 


Pronto! Agora é aguardar e verificar se o cálculo acabou (ou se entrou em problemas rs ).




## SCRIPTS

Nota: Eu gosto de usar a fila nvidia_long e cpu_shared pq geralmente são rapidas para o cálculo começar a rodar.




#### Gromacs (Gpu)

```
#!/bin/bash -l
#SBATCH --nodes=1                      #Numero de Nós
#SBATCH --ntasks-per-node=1            #Numero de tarefas por Nó
#SBATCH --ntasks=1                     #Numero total de tarefas MPI
#SBATCH --cpus-per-task=24   
#SBATCH -p nvidia_long                 #Fila (partition) a ser utilizada
#SBATCH -J ctac_l_oct                  #Nome job
#SBATCH --error=teste.err
##SBATCH --output=teste.out
#SBATCH --exclusive                    #Utilização exclusiva dos nós durante a execução do job

module purge all
module load gromacs/2019.1_openmpi_gnu
module list

exec=/scratch/app/gromacs/2019.1_openmpi_gnu/bin/gmx_mpi_gpu

srun $exec mdrun -v -deffnm md_ctac_oc3t -nb gpu
```

#### Gromacs (cpu)


```
#!/bin/bash -l
#SBATCH --nodes=1                      #Numero de Nós
#SBATCH --ntasks-per-node=1            #Numero de tarefas por Nó
#SBATCH --ntasks=1                     #Numero total de tarefas MPI
#SBATCH --cpus-per-task=24    
#SBATCH -p cpu_shared                  #Fila (partition) a ser utilizada
#SBATCH -J md_sds_oct                  #Nome job
#SBATCH --error=teste.err
##SBATCH --output=teste.out
#SBATCH --exclusive                    #Utilização exclusiva dos nós durante a execução do job
 
module purge all
module load gromacs/2019.3_openmpi_gnu
module list
 
exec=/scratch/app/gromacs/2019.3_openmpi_gnu/bin/gmx_mpi
srun $exec mdrun  -v -deffnm md_sds_l_oct
```




#### Gromacs (restart from checkpoint)




```
#!/bin/bash -l
#SBATCH --nodes=1                      #Numero de Nós
#SBATCH --ntasks-per-node=1            #Numero de tarefas por Nó
#SBATCH --ntasks=1                     #Numero total de tarefas MPI
#SBATCH --cpus-per-task=24      
#SBATCH -p nvidia_long                 #Fila (partition) a ser utilizada
#SBATCH -J md_1_ctac_l_oct             #Nome job
#SBATCH --error=teste.err
##SBATCH --output=teste.out
#SBATCH --exclusive                    #Utilização exclusiva dos nós durante a execução do job


module purge all
module load gromacs/2019.3_openmpi_gnu
module list

exec=/scratch/app/gromacs/2019.3_openmpi_gnu/bin/gmx_mpi_gpu
srun $exec mdrun  -v -deffnm md_1_ctac_l_oct -cpi md_1_ctac_l_oct.cpt -append -nb gpu
```


#### Orca (cpu)


```
#!/bin/bash -l
#SBATCH --nodes=1                      #Numero de Nós
#SBATCH --ntasks-per-node=1            #Numero de tarefas por Nó
#SBATCH --ntasks=1                     #Numero total de tarefas MPI
#SBATCH --cpus-per-task=24    
#SBATCH -p cpu_shared                  #Fila (partition) a ser utilizada
#SBATCH -J sds                         #Nome job
#SBATCH --error=teste.err
##SBATCH --output=teste.out
#SBATCH --exclusive                    #Utilização exclusiva dos nós durante a execução do job

module purge all
module load orca/4.0.1
module list

EXEC=/scratch/app/orca/4.0.1/orca
srun $EXEC SDS_linear.inp  > SDS_linear.out
```




