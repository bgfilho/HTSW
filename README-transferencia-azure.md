
# 🚀 Transferência de Imagens para Azure Blob com Proxy usando Rclone

Este projeto descreve como mover 10 TB de imagens (2021–2024) armazenadas em uma máquina Windows virtualizada e isolada, para um blob de armazenamento **frio (Cool Tier)** no Azure, utilizando `rclone`. A máquina acessa a internet por meio de **proxy**.

---

## 📦 Estrutura da Solução

- 💾 **Fonte dos dados:** Máquina virtual Windows com as imagens
- ☁️ **Destino:** Azure Blob Storage (Cool Tier)
- 🔁 **Ferramenta de transferência:** `rclone`
- 🌐 **Acesso externo:** Via proxy corporativo
- 🗃️ **Autenticação:** SAS Token (ou chave da conta)

---

## 🧱 Etapas de Implementação

### 1. 🔒 Defina a Redundância do Blob

| Tipo     | Descrição |
|----------|-----------|
| **LRS**  | Local Redundant Storage – 3 cópias na mesma zona de disponibilidade |
| **ZRS**  | Zone Redundant Storage – 3 cópias em zonas distintas na mesma região |
| **GRS**  | Geo-Redundant Storage – Cópias em regiões diferentes |
| **RA-GRS** | GRS com leitura secundária habilitada |

---

### 2. 👥 Configure Permissões com RBAC

**RBAC** (Role-Based Access Control) permite controlar acessos no Azure com base em funções atribuídas, como:

- `Storage Blob Data Reader`
- `Storage Blob Data Contributor`

Alternativamente, use **SAS Token** para acesso direto sem RBAC.

---

### 3. 🔧 Instale o Rclone

- [Baixar aqui](https://rclone.org/downloads/)
- Extraia em `C:\rclone\` ou outro diretório
- (Opcional) Adicione ao `PATH`

---

### 4. 🌐 Configure o Proxy

No PowerShell ou Prompt de Comando:

```powershell
set HTTP_PROXY=http://usuario:senha@proxy.empresa.com:8080
set HTTPS_PROXY=http://usuario:senha@proxy.empresa.com:8080
```

---

### 5. ⚙️ Configure o Rclone com SAS Token

Use o comando abaixo para copiar arquivos:

```powershell
rclone copy "C:\Imagens\2021-2024" :azureblob:meucontainer --azureblob-sas-url="https://<conta>.blob.core.windows.net/<container>?<sas-token>" --progress
```

---

## 📜 Script Automático PowerShell: `upload-imagens.ps1`

```powershell
# Configurações
$proxy = "http://usuario:senha@proxy.seudominio.com:8080"
$origem = "C:\Imagens\2021-2024"
$containerUrl = "https://<conta>.blob.core.windows.net/<container>?<sas-token>"
$logPath = "C:\Logs\rclone-upload.log"
$rclonePath = "C:\rclone\rclone.exe"

# Cria diretório de log, se não existir
if (!(Test-Path -Path (Split-Path $logPath))) {
    New-Item -Path (Split-Path $logPath) -ItemType Directory
}

# Define o proxy
$env:HTTP_PROXY = $proxy
$env:HTTPS_PROXY = $proxy

# Executa rclone
$comando = "`"$rclonePath`" copy `"$origem`" :azureblob:meucontainer --azureblob-sas-url=`"$containerUrl`" --progress --log-file=`"$logPath`" --log-level INFO"
Invoke-Expression $comando

# Verificação de sucesso
if ($LASTEXITCODE -eq 0) {
    Write-Host "✅ Transferência concluída com sucesso!" -ForegroundColor Green
} else {
    Write-Host "❌ Falha na transferência. Verifique o log em: $logPath" -ForegroundColor Red
}
```

---

## ⏰ Agendamento da Tarefa no Windows

1. Abra o **Agendador de Tarefas**
2. Crie nova tarefa com privilégios administrativos
3. Em **Ação**, escolha:
   - Programa/script: `powershell.exe`
   - Argumentos:
     ```bash
     -ExecutionPolicy Bypass -File "C:\Scripts\upload-imagens.ps1"
     ```

---

## ✅ Teste

Execute manualmente o script:

```powershell
powershell -ExecutionPolicy Bypass -File "C:\Scripts\upload-imagens.ps1"
```

---

## 📌 Observações

- Recomendado testar com arquivos pequenos primeiro
- Mantenha o SAS Token seguro
- Pode ser adaptado para usar `sync` ao invés de `copy`
- Log detalhado será salvo em `C:\Logs\rclone-upload.log`

---

## 📎 Referências

- [Documentação do Rclone – Azure Blob](https://rclone.org/azureblob/)
- [Tipos de Redundância no Azure](https://learn.microsoft.com/pt-br/azure/storage/common/storage-redundancy)
- [Rclone e Proxy](https://rclone.org/faq/#how-do-i-use-rclone-with-a-http-proxy)
