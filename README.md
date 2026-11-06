# 🚀 DeployToEC2 - Pipeline CI/CD com GitHub Actions

Pipeline de integração e entrega contínua (CI/CD) automatizado utilizando **GitHub Actions** para sincronização e deploy direto de arquivos em instâncias **AWS EC2** via SSH/rsync.

---

## 📋 Visão Geral

Este projeto serve como um modelo (template/PoC) de automação de deploy contínuo:
- A cada `push` na branch principal (`main`), o GitHub Actions é disparado automaticamente.
- O runner executa o checkout do código e utiliza a action [`easingthemes/ssh-deploy`](https://github.com/easingthemes/ssh-deploy).
- A sincronização dos arquivos com a máquina remota na AWS é realizada de forma segura e rápida através do utilitário `rsync` encapsulado por conexão SSH.

---

## 🛠️ Tecnologias e Recursos

- **Cloud Provider:** Amazon Web Services (AWS EC2 - Amazon Linux / Ubuntu)
- **CI/CD:** GitHub Actions
- **Protocolo de Transferência:** SSH (Porta 22) + rsync
- **Autenticação Segura:** GitHub Encrypted Secrets (Chave privada RSA/ED25519)

---

## 📂 Estrutura do Repositório

```text
DeployToEC2/
├── .github/
│   └── workflows/
│       └── deploy.yml    # Definição do pipeline de CI/CD do GitHub Actions
├── teste.txt             # Arquivo de teste para validação de sincronização
└── README.md             # Documentação do projeto
```

---

## ⚙️ Configuração e Pré-requisitos

### 1. Na AWS (Instância EC2)
- Uma instância EC2 em execução (Amazon Linux, Ubuntu, etc.).
- **Security Group:** A porta `22` (SSH) deve estar acessível para os IPs dos runners do GitHub Actions (ou liberada conforme política de segurança da sua VPC).
- Um par de chaves SSH (`.pem`) configurado e autorizado na instância (`~/.ssh/authorized_keys`).

---

### 2. No GitHub (Repositório)
Navegue até **Settings > Secrets and variables > Actions** e adicione as seguintes secrets:

| Nome da Secret | Descrição | Exemplo |
|---|---|---|
| `EC2_SSH_KEY` | Conteúdo completo da chave privada SSH (`.pem`) gerada na AWS | `-----BEGIN RSA PRIVATE KEY----- ...` |
| `EC2_HOST` *(opcional / recomendado)* | IP Público ou DNS público da instância EC2 | `ec2-xx-xxx-xxx-xx.compute-1.amazonaws.com` |
| `EC2_USER` *(opcional / recomendado)* | Usuário SSH do sistema operacional da EC2 | `ec2-user` (Amazon Linux) ou `ubuntu` |

---

## 🔄 Fluxo de Funcionamento do Pipeline

```mermaid
flowchart LR
    A[Push na branch main] --> B[GitHub Actions Runner]
    B --> C[Checkout do Código]
    C --> D[Conexão SSH com EC2]
    D --> E[Sincronização via rsync]
    E --> F[Arquivos atualizados na EC2]
```

### Detalhes do Workflow (`deploy.yml`)

O arquivo `.github/workflows/deploy.yml` define os seguintes parâmetros principais:

- **`SOURCE`:** Diretório de origem a ser enviado (`./` sincroniza todo o repositório).
- **`TARGET`:** Caminho de destino no servidor remoto (ex: `/home/ec2-user/`).
- **`ARGS`:** Flags do rsync (`-rltgoDzvO --delete`), garantindo preservação de atributos, compressão na transferência e remoção de arquivos obsoletos no servidor.
- **`EXCLUDE`:** Diretórios e arquivos ignorados no deploy (ex: `/node_modules/`, `**/.env`, pastas de build temporárias), garantindo segurança e desempenho.

---

## 🧪 Como Testar

1. Realize qualquer alteração em um arquivo (por exemplo, no `teste.txt`).
2. Faça o commit e envie para a branch principal:
   ```bash
   git add teste.txt
   git commit -m "test: valida deploy automatico para EC2"
   git push origin main
   ```
3. Acesse a aba **Actions** no GitHub para acompanhar a execução do pipeline em tempo real.
4. Conecte-se na sua instância EC2 via terminal para verificar se os arquivos foram atualizados:
   ```bash
   ssh -i sua-chave.pem ec2-user@seu-ip-ec2
   ls -la /home/ec2-user/
   ```

---

## 👤 Autor

Desenvolvido por **[Igor Paiva](https://github.com/igorpaiva26)**.
