# Kali-Linux-Acesso-Remoto-via-XRDP-XFCE
# 🖥️ Kali Linux — Acesso Remoto via XRDP + XFCE

Tutorial para configurar e recuperar o acesso remoto gráfico ao Kali Linux através do **RDP do Windows**, sem utilizar navegador.

## 📌 Informações da configuração

* **Sistema:** Kali Linux
* **Desktop:** XFCE
* **Protocolo:** RDP
* **Serviço:** XRDP
* **Porta:** `3389`
* **IP do Kali:** `seu ip do kali`
* **Usuário:** `seu usuario`
* **Cliente Windows:** Conexão de Área de Trabalho Remota (`mstsc`)

---

# 1. Instalar XRDP e XFCE

```bash
sudo apt update
sudo apt install --reinstall kali-desktop-xfce xfce4 xrdp dbus-x11 -y
```

Verificar o XRDP:

```bash
sudo systemctl status xrdp --no-pager
```

Deve aparecer:

```text
Active: active (running)
```

---

# 2. Configurar o XRDP para iniciar o XFCE

Editar:

```bash
sudo nano /etc/xrdp/startwm.sh
```

Apagar o conteúdo e deixar exatamente:

```bash
#!/bin/sh

export XDG_CURRENT_DESKTOP=XFCE
export XDG_SESSION_DESKTOP=xfce
export XDG_SESSION_TYPE=x11

if [ -r /etc/profile ]; then
    . /etc/profile
fi

if [ -r "$HOME/.profile" ]; then
    . "$HOME/.profile"
fi

exec dbus-run-session startxfce4
```

Salvar:

**Ctrl + O → Enter → Ctrl + X**

Dar permissão:

```bash
sudo chmod +x /etc/xrdp/startwm.sh
```

---

# 3. Corrigir permissões da chave do XRDP

Executar:

```bash
sudo chown root:xrdp /etc/xrdp/key.pem
sudo chmod 640 /etc/xrdp/key.pem
sudo usermod -aG xrdp xrdp
```

---

# 4. Reiniciar o XRDP

```bash
sudo systemctl daemon-reload
sudo systemctl restart xrdp
```

Verificar:

```bash
sudo systemctl status xrdp --no-pager
```

O serviço deve estar:

```text
Active: active (running)
```

---

# 5. Acessar pelo Windows

No Windows:

**Win + R**

Digite:

```text
mstsc
```

No campo computador:

```text
192.168.100.141
```

Na tela do XRDP:

```text
Session: Xorg
Username: seu user
Password: senha do Kali
```

Depois de entrar, o desktop XFCE do Kali será carregado.

---

# 🚨 6. CORREÇÃO RÁPIDA — XRDP CONECTA E FECHA

### Sintoma

O Windows conecta ao Kali, aparece a tela de login, mas depois mostra:

> O acesso remoto foi desligado do servidor.

Ou a conexão simplesmente fecha depois do login.

Isso normalmente significa que o XRDP conseguiu conectar, mas a sessão gráfica XFCE não iniciou corretamente.

---

## 🔧 Procedimento rápido

### 1. Parar o XRDP e encerrar sessões antigas

```bash
sudo systemctl stop xrdp
sudo pkill -u seu user
```

### 2. Limpar arquivos da sessão gráfica

```bash
rm -f ~/.Xauthority ~/.ICEauthority ~/.xsession ~/.xsession-errors
```

### 3. Conferir o `startwm.sh`

```bash
sudo nano /etc/xrdp/startwm.sh
```

Deve conter:

```bash
#!/bin/sh

export XDG_CURRENT_DESKTOP=XFCE
export XDG_SESSION_DESKTOP=xfce
export XDG_SESSION_TYPE=x11

if [ -r /etc/profile ]; then
    . /etc/profile
fi

if [ -r "$HOME/.profile" ]; then
    . "$HOME/.profile"
fi

exec dbus-run-session startxfce4
```

Depois:

```bash
sudo chmod +x /etc/xrdp/startwm.sh
```

### 4. Corrigir novamente a chave

```bash
sudo chown root:xrdp /etc/xrdp/key.pem
sudo chmod 640 /etc/xrdp/key.pem
sudo usermod -aG xrdp xrdp
```

### 5. Reiniciar

```bash
sudo systemctl daemon-reload
sudo systemctl restart xrdp
```

Verificar:

```bash
sudo systemctl status xrdp --no-pager
```

Se aparecer:

```text
Active: active (running)
```

tentar novamente pelo Windows:

```text
mstsc
```

IP:

```text
192.168.100.141
```

Sessão:

```text
Xorg
```

---

# ⚡ 7. COMANDO SOS

Se o `startwm.sh` já estiver configurado corretamente, primeiro tente somente:

```bash
sudo systemctl stop xrdp && sudo pkill -u proevox
rm -f ~/.Xauthority ~/.ICEauthority ~/.xsession ~/.xsession-errors
sudo systemctl start xrdp
```

Depois tente conectar novamente pelo Windows.

---

# 🔍 8. Verificar se a porta 3389 está funcionando

No Windows PowerShell:

```powershell
Test-NetConnection "seu ip do klai" -Port 3389
```

O resultado esperado:

```text
TcpTestSucceeded : True
```

Se aparecer:

```text
TcpTestSucceeded : False
```

o problema provavelmente é de rede, firewall ou do serviço XRDP.

No Kali:

```bash
sudo systemctl status xrdp --no-pager
```

E:

```bash
sudo ss -lntp | grep 3389
```

O esperado é algo semelhante a:

```text
LISTEN 0 10 0.0.0.0:3389
```

---

# 📝 9. Consultar os logs quando o problema persistir

### Log principal do XRDP

```bash
sudo tail -50 /var/log/xrdp-sesman.log
```

### Erros da sessão do usuário

```bash
tail -50 ~/.xsession-errors
```

### Verificar arquivos de log do Xorg

```bash
ls -la ~/.xorgxrdp*.log 2>/dev/null
```

Se existirem:

```bash
tail -100 ~/.xorgxrdp.*.log
```

---

# 🧠 10. Entendendo o funcionamento

A conexão funciona desta maneira:

```text
┌─────────────────────┐
│     Windows PC      │
│       mstsc.exe     │
└──────────┬──────────┘
           │
           │ RDP :3389
           ▼
┌─────────────────────┐
│    Kali Linux       │
│   192.168.100.141   │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│        XRDP         │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│        Xorg         │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│   dbus-run-session  │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│        XFCE         │
│  Desktop do Kali    │
└─────────────────────┘
```

---

# ✅ 11. Checklist rápido

Quando o acesso remoto parar de funcionar, verificar nesta ordem:

* [ ] Kali está ligado
* [ ] IP continua `192.168.100.141`
* [ ] XRDP está ativo
* [ ] Porta `3389` está aberta
* [ ] `startwm.sh` está correto
* [ ] XFCE está instalado
* [ ] Arquivos `.Xauthority` e `.ICEauthority` foram limpos
* [ ] Não existem sessões antigas do usuário
* [ ] Sessão do XRDP está selecionada como `Xorg`

---

# 📋 Comandos principais para guardar

### Status

```bash
sudo systemctl status xrdp --no-pager
```

### Reiniciar

```bash
sudo systemctl restart xrdp
```

### Parar sessões antigas

```bash
sudo pkill -u "user do kali"
```

### Limpar sessão

```bash
rm -f ~/.Xauthority ~/.ICEauthority ~/.xsession ~/.xsession-errors
```

### Ver porta 3389

```bash
sudo ss -lntp | grep 3389
```

### Ver log

```bash
sudo tail -50 /var/log/xrdp-sesman.log
```

### Recuperação rápida

```bash
sudo systemctl stop xrdp && sudo pkill -u proevox
rm -f ~/.Xauthority ~/.ICEauthority ~/.xsession ~/.xsession-errors
sudo systemctl start xrdp
```

---

## 🎯 Resultado esperado

Após a configuração, o acesso deve funcionar assim:

```text
Windows
   ↓
mstsc
   ↓
192.168.100.141:3389
   ↓
XRDP
   ↓
Xorg
   ↓
XFCE
   ↓
🖥️ Desktop gráfico do Kali Linux
```

**Não é necessário navegador para acessar o Kali.**
