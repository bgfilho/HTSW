# Guia Completo: Montando OneDrive com Rclone e Criando um Serviço no Windows

Este documento contém todos os comandos e configurações discutidos para instalar, configurar e montar o OneDrive com Rclone, além de rodá-lo como serviço no Windows.

---

## 1️⃣ Instalação do Rclone e WinFsp

### 🔹 Instalar o Rclone
1. Baixe e instale o **Rclone**:
   👉 https://rclone.org/downloads/

2. Extraia o arquivo e copie `rclone.exe` para `C:\Windows\System32` (opcional, para facilitar o uso via terminal).

### 🔹 Instalar o WinFsp (necessário para montagem)
1. Baixe o **WinFsp**:
   👉 https://github.com/winfsp/winfsp/releases
2. Instale normalmente e reinicie o computador.

Para verificar se o WinFsp está funcionando, execute:
```cmd
fsutil fsinfo drives
fltmc
```

Se o WinFsp não aparecer na lista, reinstale e adicione ao `PATH`:
1. Pressione `Win + R`, digite `sysdm.cpl` e pressione **Enter**.
2. Vá para **Avançado** → clique em **Variáveis de Ambiente**.
3. Em **Variáveis do Sistema**, edite **Path** e adicione:
   ```
   C:\Program Files (x86)\WinFsp in
   ```

---

## 2️⃣ Configuração do OneDrive no Rclone

Para configurar o OneDrive:
```cmd
rclone config
```
1. Escolha **"New remote"** e dê um nome (ex: `ONEDRIVE-EXEMPLO`).
2. Escolha **OneDrive** como tipo de armazenamento.
3. Autentique com sua conta Microsoft.
4. Teste a conexão com:
   ```cmd
   rclone lsd ONEDRIVE-EXEMPLO:
   ```

Se listar pastas, está funcionando corretamente.

---

## 3️⃣ Montar o OneDrive como Unidade de Disco

Para montar o OneDrive como unidade `Z:`, execute:
```cmd
rclone mount ONEDRIVE-EXEMPLO: Z: --vfs-cache-mode writes --allow-other --network-mode
```
- **`--vfs-cache-mode writes`** → Melhora desempenho e permite salvar arquivos grandes.
- **`--allow-other`** → Permite acesso por outros usuários.
- **`--network-mode`** → Evita problemas de permissão.

Para rodar em segundo plano (daemon):
```cmd
rclone mount ONEDRIVE-EXEMPLO: Z: --vfs-cache-mode writes --allow-other --network-mode --daemon
```

Se precisar desmontar:
```cmd
rclone rc mount/unmount mountPoint="Z:"
```

---

## 4️⃣ Criar um Serviço do Windows para Montar Automaticamente

### 📌 Criar um Script de Montagem

1. Abra o Bloco de Notas e copie este código:
```cmd
@echo off
rclone mount ONEDRIVE-EXEMPLO: Z: --vfs-cache-mode writes --allow-other --network-mode --daemon
```

2. Salve como `C:
clone\mount_onedrive.bat`.

### 📌 Criar um Serviço com NSSM

1. Baixe o **NSSM** 👉 https://nssm.cc/download
2. Extraia o arquivo e copie `nssm.exe` para `C:\Windows\System32`.
3. Abra o **Prompt de Comando (cmd)** como **Administrador**.
4. Crie o serviço:
   ```cmd
   nssm install RcloneOneDrive
   ```
5. No menu do NSSM:
   - **Path**: Selecione `C:
clone\mount_onedrive.bat`.
   - **Startup Type**: Escolha `Automatic`.
   - **Log on**: Marque `Local System Account`.
6. Clique em **Install service**.

Para iniciar:
```cmd
net start RcloneOneDrive
```
Para parar:
```cmd
net stop RcloneOneDrive
```
Para remover:
```cmd
nssm remove RcloneOneDrive confirm
```

---

## 5️⃣ Executar o Rclone com Interface Gráfica (Web GUI) como Serviço

Se quiser rodar o **Rclone Web GUI** como daemon junto com a montagem do OneDrive:

1. Abra o Bloco de Notas e copie:
```cmd
@echo off
rclone rcd --rc-web-gui --rc-user=admin --rc-pass=senha --rc-addr=localhost:5572 --rc-no-auth --daemon
timeout /t 5
rclone mount ONEDRIVE-EXEMPLO: Z: --vfs-cache-mode writes --allow-other --network-mode --daemon
```
2. Salve como `C:
clone\mount_onedrive_gui.bat`.
3. Crie um serviço:
   ```cmd
   nssm install RcloneWebGui
   ```
4. Configure como feito anteriormente.

A interface gráfica estará disponível em:  
👉 **http://localhost:5572/gui**  
Usuário: `admin` | Senha: `senha` (se configurado).

---

## 6️⃣ Erros e Soluções

### 🔴 **Erro: mountpoint path already exists**
```cmd
CRITICAL: Fatal error: failed to mount FUSE fs: mountpoint path already exists
```
🔹 **Solução**: Verifique se a pasta já está sendo usada:
```cmd
tasklist | findstr rclone
taskkill /IM rclone.exe /F
rclone rc mount/unmount mountPoint="Z:"
```

### 🔴 **Erro: cgofuse: cannot find winfsp**
```cmd
CRITICAL: Fatal error: failed to mount FUSE fs: mount stopped before calling Init: mount failed: cgofuse: cannot find winfsp
```
🔹 **Solução**:
1. Instale o **WinFsp** 👉 https://github.com/winfsp/winfsp/releases.
2. Reinicie o computador.
3. Adicione `C:\Program Files (x86)\WinFspin` ao `PATH`.

### 🔴 **Erro: Request failed with status code 500**
🔹 **Solução**: Tente reiniciar o Rclone Web GUI ou verifique permissões no firewall.

---
