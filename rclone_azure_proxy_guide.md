
# ☁️ Como Configurar o RClone com Azure Blob Storage Usando Proxy

Este guia mostra como configurar o **RClone** para enviar arquivos ao **Azure Blob Storage**, incluindo o uso de **proxy HTTP/HTTPS** em ambientes corporativos ou com restrição de rede.

---

## 📦 Requisitos

- RClone instalado (https://rclone.org/downloads/)
- Conta no Azure com acesso ao Blob Storage
- Acesso a um servidor proxy (caso necessário)
- Terminal (Linux, macOS ou Windows PowerShell/CMD)

---

## 🔧 Passo a Passo: Configurando o RClone com o Azure Blob

### 1. Inicie o processo de configuração

No terminal, execute:

```bash
rclone config
```

### 2. Crie um novo remote

Siga as instruções:

- Digite `n` para criar um novo remote
- Nomeie o remote (ex: `azureblob`)
- Escolha o tipo: digite `21` ou selecione `Azure Blob Storage`

---

### 🔐 Opção 1: Usando Account Name + Account Key

Essa é a forma mais comum e direta.

Você precisa obter:
- **Storage Account Name** (ex: `minhaconta`)
- **Storage Account Key** (disponível no portal Azure → Conta de Armazenamento → Chaves de Acesso)

#### Exemplo de respostas:

```
Storage Account Name> minhaconta
Storage Account Key> xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
Endpoint for the service (leave blank unless needed)> [pressione Enter]
```

---

### 🔐 Opção 2: Usando SAS Token

Mais seguro, ideal para acessos limitados e temporários.

Você precisa obter:
- **SAS URL** (com token), ex:  
  `https://minhaconta.blob.core.windows.net/meu-container?sas_token_aqui`

#### Exemplo de respostas:

```
Storage Account Name> minhaconta
Storage Account Key> [deixe em branco]
SAS URL for the container or blob> https://minhaconta.blob.core.windows.net/meu-container?sas_token_aqui
```

---

### ✅ Teste a conexão

Liste os containers ou envie um arquivo de teste:

```bash
rclone lsd azureblob:
rclone copy arquivo.txt azureblob:meu-container
```

---

## 🌐 Usando RClone com Proxy

Se sua rede exige proxy para acessar a internet, defina as variáveis de ambiente antes de usar o RClone:

### Linux/macOS:

```bash
export HTTPS_PROXY=http://usuario:senha@proxy.exemplo.com:3128
export HTTP_PROXY=http://usuario:senha@proxy.exemplo.com:3128
```

### Windows (CMD ou PowerShell):

```powershell
set HTTPS_PROXY=http://usuario:senha@proxy.exemplo.com:3128
set HTTP_PROXY=http://usuario:senha@proxy.exemplo.com:3128
```

> 🔐 **Dica de segurança:** Evite deixar senhas salvas em arquivos ou comandos permanentes. Prefira tokens ou autenticação controlada.

---

## 🗂 Exemplo de `rclone.conf`

Local: `~/.config/rclone/rclone.conf` (Linux/macOS) ou `%USERPROFILE%\.config\rclone\rclone.conf` (Windows)

### Usando chave de conta:

```ini
[azureblob]
type = azureblob
account = minhaconta
key = xxxxxxxxxxxxxxxxxxxxxxxxx
```

### Usando SAS Token:

```ini
[azureblob]
type = azureblob
sas_url = https://minhaconta.blob.core.windows.net/meu-container?sas_token_aqui
```

---

## 🚀 Upload automático com proxy ativo

Exemplo de envio direto via shell script:

```bash
#!/bin/bash

export HTTPS_PROXY=http://usuario:senha@proxy.exemplo.com:3128
export HTTP_PROXY=http://usuario:senha@proxy.exemplo.com:3128

rclone copy /caminho/do/arquivo.txt azureblob:meu-container --progress
```
