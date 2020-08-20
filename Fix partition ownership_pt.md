# Como modificar o proprietário de uma partição

## Este é um breve tutorial de como modificar o proprietário, configuração de acesso e permissões a determinada partição.

_Read this article in english [here](https://github.com/synini/SFTP-Drive/blob/master/Fix%20partition%20ownership_en.md)._

Este tutorial é pensado para quem utiliza armazenamento em máquina remota linux _(nesse exemplo, ubuntu, mas certamente se aplica a diversas outras distribuições)_ e gostaria de acessá-lo via SSHFS como dispositivo remoto de armazenamento.

Por padrão, dispositivos de armazenamento são montados automaticamente e ficam disponíveis para todos os usuários quando em um desktop com GUI _(graphical user interface)_. Todavia, quando utilizando o armazenamento em um servidor e, via de regra, montando-o manualmente ao menos uma vez, é preciso configurar as permissões para outros usuários além do _root_.

Antes de começar, o diretório em que eu havia montado a partição teve as permissões modificadas sem meu conhecimento, o que me forçou a verificá-lo e corrigi-lo. Inicialmente tentei modificar o proprietário do diretório com o comando `sudo chown -R <user> <folder>` e o modo com `sudo chmod a+rwx <folder>`, porém, apesar da ausência de erro, não modificaram o diretório, então continuei procurando e também tentei modificar o atributo com `sudo chattr -i <folder>`, que retornou o seguinte erro:
>chattr: Inappropriate ioctl for device while reading flags on \<folder\>.

Dito isso, vamos ao tutorial:

### Passo 1
Se montado, é preciso desmontar o armazenamento. Execute o comando abaixo para verificar numa lista simples quais discos e partições estão disponíveis:
>`lsblk -o name,mountpoint,label,size,fstype,uuid | egrep -v "^loop"`

O comando tem como saída algo semelhante a isso:
```
NAME   MOUNTPOINT                LABEL   SIZE FSTYPE   UUID
sda                                      2.7T          
├─sda1 /data/part1               part1   1.2T vfat     5633-529C
└─sda2 /data/part2               part2   1.2T vfat     9792-26A7
```
Também é possível usar o comando mais simples `lsblk -f` que retorna saída semelhante.

Para desmontar o armazenamento, usando o exemplo, execute o comando `sudo unmount /data/part1` ou `sudo unmount /dev/sda1`.

### Passo 2
Crie o diretório onde será montada a partição:
>exemplo: `mkdir -p /data/part1`

_O diretório /mnt/sda1 também é muito comum._

### Passo 3
Verifique seu número de identificação do usuário `uid` (normalmente é 1000, 1001 ou 1002):
> `grep ^"$USER" /etc/group`

Utilize o número no  comando abaixo:

> - `sudo mount -o rw,user,uid=1000,umask=007,exec /dev/sda1 /data/part1` # com permissão de execução
> - `sudo mount -o rw,user,uid=1000,dmask=007,fmask=117 /dev/sda1 /data/part1` # sem permissão de execução
> - `sudo mount -o rw,users,umask=000,exec /dev/sda1 /data.part1` # com permissão para todos os usuários - Conveniente, porém menos seguro

_Nota: é possível executar comando similar utilizando a identificação do grupo de usuários `gid`_.

### **Et voilà**
```
$ls -l data/
total 512
drwxrwx--- 4 home root 262144 Dec 31  1969 part1
```
Assim, o armazenamento é acessível remotamente para leitura e escrita, nesse exemplo, para o usuário `home`.

Clique aqui para ver como [Configurar o acesso a um Armazenamento compartilhado por SSH/SFTP no Windows 10](https://medium.com/@huvirgilio/configura%C3%A7%C3%A3o-de-armazenamento-compartilhado-por-ssh-sftp-no-windows-10-aeea0e6b64a3).

#### Notas:
> - Esse tutorial é baseado nessa [resposta](https://askubuntu.com/questions/11840/how-do-i-use-chmod-on-an-ntfs-or-fat32-partition/956072#956072) do [sudodus](https://askubuntu.com/users/55537/sudodus).
> - Artigo revisado por Carolina Brandão em 2020-08-20.