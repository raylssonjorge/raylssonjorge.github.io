---
  title: "Tutorial Gromacs (Cluster)"
  date: 2020-05-27
  tags: [Chemistryy]
  header:
  #  image: "/images/1234.jpg"

  #excerpt: "Physics, Chemistry"
  mathjax: "true"


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



### Agora vamos a parte para rodar cálculos.

Digite o seguinte comando no seu terminal (conectado ao sdumont):

```
~ cd /scratch/NOME_DO_PROJETO/username/
```


Para não ficar perdido no cluster recomendo usar o comando pwd (que indica em qual diretório você se encontra).



No /scratch do seu usuário é o lugar onde se rodam seus cálculos.Uma forma simples de fazer isso é usar um script executável em  bash.Se vocẽ quiser pode pesquisar a fundo sobre os scripts em bash mas não irei me aprofundar nesse tutorial.




Os scripts seguem essa forma geral:


```
#!/bin/bash
#SBATCH --nodes=N                      #Numero de Nós
#SBATCH --ntasks-per-node=TPN          #Numero de tarefas por Nó
#SBATCH --ntasks=T                     #Numero total de tarefas MPI
#SBATCH -p FILA                        #Fila (partition) a ser utilizada
#SBATCH -J JOB			       #Nome job
#SBATCH --exclusive                    #Utilização exclusiva dos nós durante a execução do job

module purge all		#Remove todos os módulos ativos para o seu usuário
module load 		   	#carrega um módulo  _do_ seu software
module list			#mostra todos os módulos que estão ativos para o seu usuário

exec=/scratch/app/nome_do_software/caminho até o executável

srun $exec  comandos_adicionais
```























