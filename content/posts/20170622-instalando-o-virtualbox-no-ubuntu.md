---
date: "2017-06-22"
categories:
  - ops
tags:
  - ubuntu
  - virtualbox
title: "Instalando o virtualbox 5.1 no ubuntu"
---

Um dos maiores desafios atuais dos desenvolvedores é a implementação agnóstica aos ambientes disponíveis. Mas isso, infelizmente, nem sempre é a realidade, principalmente se você utiliza o Linux em seu desktop ou notebook, e necessita fazer testes em ambientes Windows. Para resolver tais problemas diversos são disponíveis, e grande parte delas envolvem tecnologias de virtualizações de ambientes. Dois são os aplicativos principais no mercado atualmente, o VMWare e o Virtualbox, e por motivos de sinergia entre as ferramentas disponíveis(Vagrant), utilizaremos o VirtualBox.

Nosso primeiro passo será adicionar o repositório do VirtualBox à nossa lista de repositórios, executando o comando abaixo.

```bash
echo "deb http://download.virtualbox.org/virtualbox/debian $(lsb_release -cs) contrib" > /etc/apt/sources.list.d/virtualbox.list
```

Adicionado o repositório, também precisamos adicionar sua chave, validando sua confiabilidade. Para tal, execute o comando abaixo.

```bash
wget -q https://www.virtualbox.org/download/oracle_vbox_2016.asc -O- | sudo apt-key add -
wget -q -O - http://download.virtualbox.org/virtualbox/debian/oracle_vbox.asc | apt-key add -
```

Incluído o repositório e sua chave, é hora de avisar ao apt que existe um repositório novo a ser lido, atualizando sua lista, e instalar o pacote:

```bash
sudo apt update --fix-missing && apt install virtualbox-5.1
```

O pacote do VirtualBox encontra-se instalado, porém é necessário instalar suas extensões para ter acesso a diversos recursos disponíveis do pacote, como habilitar dispositivos USB nas máquinas convidadas, diretórios compartilhados, melhorias nos dispositivos gráficos, entre outros.

Para tal, execute a linha de comando abaixo, que baixará a última versão disponível neste data, no site do VirtualBox.

```bash
http://download.virtualbox.org/virtualbox/5.1.22/Oracle_VM_VirtualBox_Extension_Pack-5.1.22-115126.vbox-extpack
```

Baixado o arquivo, execute o comando abaixo para instalá-lo!

```bash
sudo VBoxManage extpack install Oracle_VM_VirtualBox_Extension_Pack-5.1.22-115126.vbox-extpack --replace
```
