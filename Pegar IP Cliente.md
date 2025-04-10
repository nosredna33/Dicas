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
