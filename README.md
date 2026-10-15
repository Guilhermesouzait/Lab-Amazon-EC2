# -Lab - Introdução ao Amazon EC2

# 🚀 Introdução ao Amazon EC2

Laboratório prático de como **executar, monitorar, redimensionar e encerrar** uma instância do Amazon EC2, com um servidor web Apache provisionado automaticamente via *user data*.

![AWS](https://img.shields.io/badge/AWS-EC2-FF9900?logo=amazonaws&logoColor=white)
![Amazon Linux](https://img.shields.io/badge/Amazon%20Linux-2023-232F3E)
![Duração](https://img.shields.io/badge/dura%C3%A7%C3%A3o-~45%20min-blue)

---

## 📑 Sumário

- [Sobre o laboratório](#-sobre-o-laboratório)
- [Objetivos](#-objetivos)
- [Arquitetura](#-arquitetura)
- [Pré-requisitos](#-pré-requisitos)
- [Tarefa 1: Iniciar a instância](#tarefa-1-iniciar-a-instância-do-ec2)
- [Tarefa 2: Monitorar a instância](#tarefa-2-monitorar-a-instância)
- [Tarefa 3: Atualizar o grupo de segurança](#tarefa-3-atualizar-o-grupo-de-segurança-e-acessar-o-servidor-web)
- [Tarefa 4: Redimensionar a instância](#tarefa-4-redimensionar-a-instância)
- [Tarefa 5: Testar a proteção contra encerramento](#tarefa-5-testar-a-proteção-contra-encerramento)
- [Tarefa 6: Encerrar a instância](#tarefa-6-encerrar-a-instância)
- [Conceitos-chave](#-conceitos-chave)
- [O que aprendi](#-o-que-aprendi)

---

## 📖 Sobre o laboratório

O **Amazon Elastic Compute Cloud (Amazon EC2)** é um serviço que fornece capacidade computacional redimensionável na nuvem. Com ele é possível:

- Obter e configurar capacidade com o mínimo de esforço;
- Ter controle total sobre os recursos de computação;
- Inicializar novos servidores em minutos e escalar para mais ou para menos conforme a demanda;
- **Pagar apenas pelo que usar**;
- Criar aplicações resistentes a falhas.

## 🎯 Objetivos

Ao final do laboratório, você será capaz de:

- [x] Iniciar um servidor web com **proteção contra encerramento** ativada
- [x] Monitorar a instância EC2
- [x] Modificar o **grupo de segurança** para permitir acesso HTTP
- [x] Redimensionar a instância (tipo e volume EBS)
- [x] Testar a proteção contra encerramento
- [x] Encerrar a instância

## 🏗️ Arquitetura

<!-- Adicione aqui o diagrama do laboratório -->
![Diagrama de arquitetura](images/architecture.png)

## ✅ Pré-requisitos

- Acesso ao ambiente do laboratório (AWS Academy / Escola da Nuvem)
- Navegador com **pop-ups permitidos**
- Noções básicas de redes (IP, portas, HTTP)

> ⏱️ **Duração estimada:** ~45 minutos

### Acessando o console

1. No canto superior direito das instruções, clique em **Start Lab**.
2. Aguarde o indicador ao lado de **AWS** ficar 🟢 verde:
   - 🔴 Vermelho: laboratório não iniciado
   - 🟡 Amarelo: iniciando
   - 🟢 Verde: pronto
3. Clique no círculo verde para abrir o **Console de Gerenciamento da AWS** em uma nova aba (o login é automático).

> 💡 **Dicas:** se aparecer `Access Denied`, feche o erro e clique em *Start Lab* novamente. Se a aba não abrir, permita pop-ups no navegador. **Não altere a Região** do laboratório.

---

## Tarefa 1: Iniciar a instância do EC2

Nesta tarefa você inicia uma instância com **proteção contra encerramento**, que evita a exclusão acidental, e com um script de *user data* que instala um servidor web.

**Caminho:** `Services` → `EC2` → `Dashboard` → `Launch instance`

### 1.1 Nome e tags

| Campo | Valor |
|---|---|
| Name | `Web Server` |

### 1.2 Imagem de máquina (AMI)

Uma **AMI** contém o modelo do volume raiz (SO/aplicações), as permissões de execução e o mapeamento de dispositivos de bloco.

- Mantenha a opção padrão: **Amazon Linux 2023**.

### 1.3 Tipo de instância

| Campo | Valor |
|---|---|
| Instance type | `t3.micro` (2 vCPUs, 1 GiB de memória) |

> ⚠️ Outros tipos podem estar bloqueados no ambiente do laboratório.

### 1.4 Par de chaves

Neste lab não haverá login na instância, então:

- **Key pair (login):** `Proceed without a key pair (Not recommended)`

### 1.5 Configurações de rede

Clique em **Edit** no painel *Network settings* e configure:

| Campo | Valor |
|---|---|
| VPC | `Lab VPC` |
| Security group name | `Web Server security group` |
| Description | `Security group for my web server` |
| Inbound security group rules | **Remover** a regra existente (sem SSH) |

> 🔒 Remover o acesso SSH reforça a segurança da instância. O **grupo de segurança** funciona como um firewall virtual: as regras podem ser alteradas a qualquer momento e são aplicadas automaticamente às instâncias associadas.

### 1.6 Armazenamento

- Mantenha o padrão: volume raiz **EBS de 8 GiB**.

### 1.7 Detalhes avançados

1. Expanda **Advanced details**.
2. Em **Termination protection**, selecione **Enable**.
3. No campo **User data**, cole o script abaixo:

```bash
#!/bin/bash
yum -y install httpd
systemctl enable httpd
systemctl start httpd
echo '<html><h1>Hello From Your Web Server!</h1></html>' > /var/www/html/index.html
```

**O que o script faz:**

1. Instala o servidor web Apache (`httpd`);
2. Habilita o serviço para iniciar junto com o sistema;
3. Inicia o servidor web;
4. Cria uma página HTML simples.

### 1.8 Executar

1. Clique em **Launch instance** e depois em **View all instances**.
2. O estado muda de `Pending` para `Running`.
3. Selecione o **Web Server** e explore as abas **Details**, **Security** e **Networking**.
4. Aguarde até:
   - **Instance state:** `Running`
   - **Status checks:** `2/2 checks passed`

---

## Tarefa 2: Monitorar a instância

O monitoramento é essencial para manter a confiabilidade, a disponibilidade e o desempenho das instâncias.

1. Selecione a instância e abra a aba **Status checks**. As verificações **System reachability** e **Instance reachability** devem estar aprovadas.
2. Abra a aba **Monitoring** para ver as métricas do **Amazon CloudWatch**.
   - Por padrão o monitoramento **básico** (a cada 5 minutos) está ativo; o **detalhado** (1 minuto) pode ser habilitado.
   - Como a instância é recente, ainda há poucas métricas.
3. Vá em `Actions` → `Monitor and troubleshoot` → `Get instance screenshot`.
   - Mostra como seria a tela do console da instância; útil quando não é possível acessar via SSH/RDP.
4. Clique em **Cancel** para fechar.

---

## Tarefa 3: Atualizar o grupo de segurança e acessar o servidor web

### 3.1 Testar o acesso (vai falhar)

1. Na aba **Details**, copie o **Public IPv4 address**.
2. Cole o IP em uma nova aba do navegador.

❓ **Você consegue acessar o servidor? Por quê?**

> ❌ **Não.** O grupo de segurança não permite tráfego de entrada na **porta 80** (HTTP). Isso demonstra o uso do security group como firewall.

### 3.2 Liberar o tráfego HTTP

1. No console EC2, vá em `Network & Security` → **Security Groups**.
2. Selecione **Web Server security group** → aba **Inbound rules** (hoje sem regras).
3. Clique em **Edit inbound rules** → **Add rule**:

| Campo | Valor |
|---|---|
| Type | `HTTP` |
| Source | `Anywhere-IPv4` |

4. Clique em **Save rules**.
5. Atualize a aba do navegador. Você deve ver:

> **Hello From Your Web Server!** ✅

---

## Tarefa 4: Redimensionar a instância

Se a instância estiver superutilizada (pequena demais) ou subutilizada (grande demais), é possível alterar o **tipo de instância** e o **tamanho do disco**. Exemplo: de `t3.micro` para `m5.large`.

### 4.1 Interromper a instância

> ℹ️ Uma instância interrompida **não gera cobrança de computação**, mas os volumes EBS anexados continuam sendo cobrados.

1. Em **Instances**, selecione o Web Server.
2. `Instance state` → **Stop instance** → **Stop**.
3. Aguarde o estado `Stopped`.

### 4.2 Alterar o tipo de instância

1. `Actions` → `Instance settings` → **Change instance type**.
2. Selecione `t3.small` (o dobro de memória do `t3.micro`) e confirme.

### 4.3 Aumentar o volume EBS

1. Vá em `Elastic Block Store` → **Volumes**.
2. Selecione o volume → `Actions` → **Modify volume**.
3. Altere o tamanho de **8** para **10 GiB** e confirme em **Modify**.

> ⚠️ A criação de volumes grandes pode ser restrita no laboratório.

### 4.4 Iniciar a instância redimensionada

1. Em **Instances**, selecione o Web Server.
2. `Instance state` → **Start instance**.

✅ Resultado: `t3.micro` → `t3.small` e disco raiz de 8 GiB → 10 GiB.

---

## Tarefa 5: Testar a proteção contra encerramento

> Esta etapa não constava no texto original; foi escrita a partir dos objetivos do laboratório. Confira com as instruções do seu curso.

1. Selecione o Web Server → `Instance state` → **Terminate instance**.
2. Ao tentar confirmar, a opção fica **desabilitada** ou retorna erro, pois a proteção contra encerramento está ativa.
3. Para encerrar, é preciso desativá-la antes: `Actions` → `Instance settings` → **Change termination protection** → desmarcar **Enable** → **Save**.

## Tarefa 6: Encerrar a instância

1. Com a proteção desativada, selecione o Web Server.
2. `Instance state` → **Terminate instance** → confirme em **Terminate**.
3. O estado passa a `Shutting-down` e depois `Terminated`.

> 💰 Encerrar recursos ao fim de cada lab evita cobranças inesperadas em contas pessoais.

---

## 🧠 Conceitos-chave

| Conceito | Descrição |
|---|---|
| **EC2** | Servidores virtuais redimensionáveis na nuvem |
| **AMI** | Modelo usado para criar a instância (SO + configurações) |
| **Tipo de instância** | Combinação de CPU, memória, armazenamento e rede |
| **EBS** | Disco virtual conectado à instância |
| **Security Group** | Firewall virtual no nível da instância |
| **User data** | Script executado na inicialização da instância |
| **Termination protection** | Evita o encerramento acidental da instância |
| **CloudWatch** | Serviço de métricas e monitoramento |

## 📝 O que aprendi

- Como provisionar um servidor web automaticamente com *user data*;
- Que o tráfego é bloqueado por padrão até uma regra de entrada ser criada;
- A diferença entre **interromper** (stop) e **encerrar** (terminate) uma instância;
- Como redimensionar CPU/memória e disco sem recriar a instância;
- Onde acompanhar a saúde da instância (status checks e CloudWatch).

---

📚 *Laboratório do curso de AWS. Documentado para fins de estudo e portfólio.*

 
