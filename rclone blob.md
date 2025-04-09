☁️ Usando rclone para enviar arquivos do Windows para Azure Blob Storage
Sim! ✅ É totalmente possível usar o rclone para transferir arquivos do Windows (mesmo atrás de proxy) para o Azure Blob Storage, com suporte a:

🔐 SAS Token

🔒 Azure AD

🌐 SFTP (caso esteja ativado no Blob)

Ele é flexível, leve e scriptável 💪

🧩 Vantagens de usar o rclone
✅ Compatível com Azure Blob Storage

🌍 Funciona com proxy

🔒 Transferência criptografada

🔁 Suporte a sync, mirror, copy, etc

⚙️ Automatizável com scripts PowerShell, batch ou agendador de tarefas

⚙️ Como configurar o rclone com Azure Blob Storage
🔧 1. Instale o rclone
Baixe: https://rclone.org/downloads/

Extraia o executável

(Opcional) Adicione ao PATH

🔧 2. Configure o Remote
Execute no terminal:

bash
Copiar
Editar
rclone config
E siga os passos:

n → Novo remote

Nome: azureblob (ou outro)

Tipo de armazenamento: selecione 13 para Azure Blob

Preencha:

account: nome da conta de armazenamento

key: chave de acesso (ou deixe em branco para usar SAS Token ou Managed Identity)

endpoint: (opcional)

✅ Exemplo com SAS Token (sem chave de conta)
bash
Copiar
Editar
rclone copy "C:\imagens" "azureblob:<nome-do-container>" --azureblob-sas-url="https://<conta>.blob.core.windows.net/<container>?<sas-token>" --progress
🌐 Usando rclone com Proxy
Configure as variáveis de ambiente:

bash
Copiar
Editar
set HTTP_PROXY=http://usuario:senha@proxy.seudominio.com:8080
set HTTPS_PROXY=http://usuario:senha@proxy.seudominio.com:8080
Ou use um script com essas variáveis exportadas antes da execução.

💡 Comandos úteis do rclone
Comando	Descrição
rclone copy	Copia arquivos locais para o Blob
rclone sync	Sincroniza (apaga do destino o que não existe na origem!)
rclone ls	Lista arquivos remotos
rclone config	Configura conexões
rclone serve sftp	(Avançado) Exponha um SFTP local com rclone
🧪 Exemplo completo de upload com progresso
bash
Copiar
Editar
rclone copy "C:\imagens\2021-2024" azureblob:meucontainer --azureblob-sas-url="https://minhaconta.blob.core.windows.net/meucontainer?<sas_token>" --progress
🛠️ Script PowerShell completo
Crie um arquivo chamado upload-imagens.ps1 com o seguinte conteúdo:

powershell
Copiar
Editar
# --- CONFIGURAÇÕES ---
$proxy = "http://usuario:senha@proxy.seudominio.com:8080"
$origem = "C:\Imagens\2021-2024"
$containerUrl = "https://<conta>.blob.core.windows.net/<container>?<sas-token>"
$logPath = "C:\Logs\rclone-upload.log"
$rclonePath = "C:\rclone\rclone.exe"

# --- CRIA DIRETÓRIO DE LOG SE NÃO EXISTIR ---
if (!(Test-Path -Path (Split-Path $logPath))) {
    New-Item -Path (Split-Path $logPath) -ItemType Directory
}

# --- EXPORTA VARIÁVEIS DE AMBIENTE PARA USO DO PROXY ---
$env:HTTP_PROXY = $proxy
$env:HTTPS_PROXY = $proxy

# --- EXECUTA RCLONE COM LOG ---
$comando = "`"$rclonePath`" copy `"$origem`" :azureblob:meucontainer --azureblob-sas-url=`"$containerUrl`" --progress --log-file=`"$logPath`" --log-level INFO"

Write-Host "Iniciando transferência..." -ForegroundColor Cyan
Invoke-Expression $comando

if ($LASTEXITCODE -eq 0) {
    Write-Host "✅ Transferência concluída com sucesso!" -ForegroundColor Green
} else {
    Write-Host "❌ Falha na transferência. Verifique o log em: $logPath" -ForegroundColor Red
}
📌 Agendando no Windows Task Scheduler
Abra o Agendador de Tarefas

Crie uma nova tarefa com privilégios administrativos

Na aba Ações, configure:

plaintext
Copiar
Editar
Programa/script: powershell.exe
Argumentos: -ExecutionPolicy Bypass -File "C:\Scripts\upload-imagens.ps1"
Defina o agendamento desejado (diário, semanal etc.)

🧪 Teste manual
Antes de agendar, execute manualmente no terminal:

powershell
Copiar
Editar
powershell -ExecutionPolicy Bypass -File "C:\Scripts\upload-imagens.ps1"
🎁 Quer um script com seus dados reais?
Me envie:

✅ URL com SAS Token

✅ Caminho das imagens

✅ Nome da conta / container

✅ Se usará proxy (com ou sem autenticação)

Posso montar com seus dados, ou deixar com placeholders prontos pra você preencher 😉

