Sim, é possível obter o IP do cliente via JavaScript no navegador usando um serviço externo de consulta de IP (já que o JavaScript puro não tem acesso direto ao IP local do usuário). Depois, você pode enviar esse IP para o seu backend PHP via POST.

---

### **Método 1: Usando uma API de IP (recomendado)**
Você pode usar serviços gratuitos como:
- [https://api.ipify.org](https://api.ipify.org)  
- [https://ipapi.co/json](https://ipapi.co/json)  
- [https://ipinfo.io/json](https://ipinfo.io/json)  

#### **Exemplo em JavaScript (fetch)**
```javascript
// 1. Pega o IP usando uma API externa
fetch('https://api.ipify.org?format=json')
  .then(response => response.json())
  .then(data => {
    const clientIP = data.ip;
    console.log("IP do cliente:", clientIP);

    // 2. Envia o IP para o backend PHP via POST
    fetch('seu_script.php', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({ ip: clientIP }),
    })
    .then(response => response.json())
    .then(data => console.log("Resposta do servidor:", data))
    .catch(error => console.error("Erro ao enviar IP:", error));
  })
  .catch(error => console.error("Erro ao obter IP:", error));
```

#### **Exemplo em PHP (`seu_script.php`)**
```php
<?php
header('Content-Type: application/json');

$data = json_decode(file_get_contents('php://input'), true);
$clientIP = $data['ip'] ?? $_SERVER['REMOTE_ADDR']; // Fallback para REMOTE_ADDR se falhar

// Salva ou processa o IP (ex.: banco de dados, log, etc.)
file_put_contents('ips.log', $clientIP . PHP_EOL, FILE_APPEND);

echo json_encode(["status" => "success", "ip" => $clientIP]);
?>
```

---

### **Método 2: WebRTC (pode vazar IP local em redes privadas)**
O JavaScript pode tentar obter o IP local via WebRTC, mas **nem sempre retorna o IP público** (e pode ser bloqueado por navegadores como o Brave ou Safari).

```javascript
function getIPFromWebRTC() {
  return new Promise((resolve, reject) => {
    const pc = new RTCPeerConnection({ iceServers: [] });
    pc.createDataChannel(''); // Cria um canal de dados falso
    pc.createOffer()
      .then(offer => pc.setLocalDescription(offer))
      .catch(reject);
    pc.onicecandidate = (ice) => {
      if (!ice.candidate) return;
      const ip = ice.candidate.candidate.match(/([0-9]{1,3}(\.[0-9]{1,3}){3})/)?.[1];
      if (ip) resolve(ip);
    };
  });
}

// Uso:
getIPFromWebRTC()
  .then(ip => console.log("IP via WebRTC:", ip))
  .catch(error => console.error("WebRTC falhou:", error));
```
> ⚠️ **Limitação:** Só funciona em alguns navegadores e pode retornar apenas o IP local (não o público).

---

### **Método 3: Usando um iframe oculto (alternativa antiga)**
Se as APIs estiverem bloqueadas, você pode tentar carregar um iframe de um serviço que retorne o IP:
```html
<iframe id="ipFrame" src="https://api.ipify.org?format=json" style="display:none;"></iframe>
<script>
  document.getElementById('ipFrame').onload = function() {
    const iframeContent = this.contentWindow.document.body.innerText;
    const clientIP = JSON.parse(iframeContent).ip;
    console.log("IP do iframe:", clientIP);
  };
</script>
```

---

### **Qual método escolher?**
| Método       | Prós | Contras |
|--------------|------|---------|
| **API Externa** (ipify/ipapi) | Confiável, retorna IP público | Requer chamada externa |
| **WebRTC** | Pode pegar IP local em redes privadas | Inconsistente, bloqueado em alguns navegadores |
| **Iframe** | Funciona sem CORS em alguns casos | Menos elegante, dependente de serviço externo |

---

### **Importante:**
- Se o usuário estiver em uma **VPN/Proxy**, o IP retornado ainda pode ser mascarado.
- Sempre use `REMOTE_ADDR` no PHP como fallback para evitar falsificação.
- Para maior precisão, combine os métodos (ex.: API + WebRTC + PHP).

Se precisar de uma solução mais robusta, considere usar **Cloudflare** (que envia o IP real no cabeçalho `CF-Connecting-IP`). 🌐

Entendi! O IP **177.55.116.231** pertence à **RedeHost Internet Ltda.**, que é a empresa por trás da **Umbler** (hospedagem). Isso explica por que **todos os clientes aparecem com o mesmo IP** nos seus logs.  

---

### **Por que isso acontece?**  
A Umbler está usando um **proxy reverso, WAF (Web Application Firewall) ou balanceador de carga** que centraliza todas as requisições sob esse IP (`177.55.116.231`). Ou seja:  
- O PHP recebe apenas o IP da Umbler (`REMOTE_ADDR = 177.55.116.231`).  
- O IP real do cliente está **oculto** ou disponível apenas em cabeçalhos específicos.  

---

### **Como obter o IP real do cliente?**  
Você precisa verificar os cabeçalhos HTTP que a Umbler pode estar repassando. Os mais comuns são:  
1. **`X-Forwarded-For`** (padrão em proxies)  
2. **`X-Real-IP`** (alternativo)  
3. **`CF-Connecting-IP`** (se a Umbler usa Cloudflare)  

#### **Código PHP para pegar o IP real:**  
```php
function getClientRealIP() {
    $ip = $_SERVER['REMOTE_ADDR']; // IP da Umbler (177.55.116.231)

    // Verifica cabeçalhos comuns de proxy
    $headers = [
        'HTTP_X_FORWARDED_FOR',
        'HTTP_X_REAL_IP',
        'HTTP_CF_CONNECTING_IP', // Cloudflare
        'HTTP_TRUE_CLIENT_IP'    // Outros serviços
    ];

    foreach ($headers as $header) {
        if (!empty($_SERVER[$header])) {
            $ip = $_SERVER[$header];
            // Pega o primeiro IP se for uma lista (ex.: "IP1, IP2")
            $ips = explode(',', $ip);
            $ip = trim($ips[0]);
            break;
        }
    }

    return $ip;
}

$clientIP = getClientRealIP();
echo "IP real do cliente: " . $clientIP;
```

---

### **Se nenhum cabeçalho funcionar...**  
1. **Contate o suporte da Umbler**  
   - Pergunte qual cabeçalho eles usam para repassar o IP real.  
   - Exemplo:  
     > _"Meus logs só mostram o IP `177.55.116.231` (RedeHost). Como posso identificar o IP real do cliente?"_  

2. **Alternativas:**  
   - Use **JavaScript + API externa** (como `ipify.org`) para enviar o IP via AJAX.  
   - Migre para um serviço que não oculte IPs (ex.: AWS, DigitalOcean, Cloudflare com configuração correta).  

---

### **Por que a Umbler faz isso?**  
- **Segurança**: Protege contra ataques DDoS e injeções.  
- **Balanceamento**: Distribui tráfego entre servidores.  
- **Otimização**: Cache e compressão de conteúdo.  

---

### **Exemplo de saída no seu log:**  
| IP no Log (Umbler) | IP Real (Cabeçalho) | Origem |
|--------------------|---------------------|--------|
| `177.55.116.231`   | `189.45.12.34`      | Cliente A (Vivo) |  
| `177.55.116.231`   | `201.28.76.91`      | Cliente B (Claro) |  

---

### **Conclusão**  
- **Todos verão `177.55.116.231`** porque a Umbler atua como intermediária.  
- **O IP real está em `X-Forwarded-For` ou similar** (teste com o código acima).  
- **Se não houver cabeçalho**, a única solução é configurar a Umbler ou mudar de hospedagem.  

Se precisar de ajuda para testar os cabeçalhos, posso te ajudar a analisar uma requisição real! 
