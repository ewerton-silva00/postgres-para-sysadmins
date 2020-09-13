## PostgreSQL para SysAdmins

Neste material utilizo Vagrant para provisionar as VMs necessárias para estudar. O ```Vagrantfile``` disponível na raíz deste repositório me permite provisionar VMs em loop, ou seja, consigo subir quantas VMs eu precisar apenas incrementando a variável ```servers```.

Se faz necessário apenas informar um nome para a VM e definir os recursos. Como se trata de um ambiente para estudo do PostgreSQL, o default no Vagrantfile é provisionar um disco secundário para inicializar o postgres numa partição separada da raíz do sistema operacional.

Abaixo dois exemplos.

**Exemplo 01**: subir uma VM apenas (default).
```
servers = {
  'primario' => { 'ip' => '10.10.10.10', 'memory' => '2048', 'cpus' => '2', 'disk' => '/tmp/primario_disk.vdi' }
}
```

**Exemplo 02**: Subir duas VMs.
```
servers = {
  'primario' => { 'ip' => '10.10.10.10', 'memory' => '2048', 'cpus' => '2', 'disk' => '/tmp/primario_disk.vdi' },
  'replica-01' => { 'ip' => '10.10.10.11', 'memory' => '2048', 'cpus' => '2', 'disk' => '/tmp/replica-01_disk.vdi' }
}
```

Diante dos exemplos acima, concluímos que é possível provisionar quantas VMs forem necessárias, respeitando os critérios de unicidade do ```nome``` da VM, ```IP``` atribuído e nome do arquivo de referência do disco secundário, setado na variável ```disk```. Esses 3 argumentos precisam ser únicos para cada VM.

Observações:

O ```Vagrantfile``` deste repositório utiliza o [Virtualbox](https://www.virtualbox.org/) como provider, a ```box``` utilizada é de minha autoria, um CentOS 7 disponível no meu repositório do [Vagrant Cloud](https://app.vagrantup.com/ewerton_silva00/boxes/centos-7-x86_64-minimal-2003), além de utilizar 2 plugins do Vagrant.

```vagrant-vbguest``` para atualizar sempre que necessário o ```VirtualBox Guest Additions```. Caso não queira ficar verificando se precisa ou não atualizar basta informar no Vagrantfile ```vbguest.auto_update = false```.

```vagrant-hosts``` para configurar o ```/etc/hosts``` das VMs automaticamente, pois dessa forma elas poderão se conversar pelo nome sem precisar de muitos ajustes.

Os dois plugins acima são recursos opcionais. Caso não queira utilizar basta remover do Vagrantfile. A box também pode ser alterada para outra de sua escolha, contanto que você faça os ajustes necessários no Vagrantfile.


**Conteúdo**:

1. [Instalação do PostgreSQL](instalacao/README.md)