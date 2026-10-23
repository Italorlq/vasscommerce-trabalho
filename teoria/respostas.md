# Questões Teóricas sobre Protocolos

## 1. HTTP, TCP e IP

**IP (Internet Protocol):** é o protocolo responsável por endereçar e rotear os pacotes entre as máquinas da rede. Cada dispositivo tem um endereço IP, e o IP tenta levar o pacote até o destino, mas não garante entrega, ordem nem ausência de duplicação.

**TCP (Transmission Control Protocol):** funciona sobre o IP e cria uma conexão confiável entre dois programas. Ele garante que os dados cheguem completos, na ordem certa e sem duplicação, retransmitindo o que for perdido.

**HTTP (HyperText Transfer Protocol):** é o protocolo da camada de aplicação usado na web. Ele roda sobre o TCP e define o formato das mensagens trocadas entre cliente e servidor, como pedir e receber páginas, imagens e dados em JSON.

**Transação HTTP:** é o ciclo completo de comunicação em que o cliente envia uma requisição e o servidor devolve uma resposta. Cada requisição gera exatamente uma resposta.

**Mensagens HTTP e sua constituição:** são textos divididos em quatro partes:
1. Linha inicial: na requisição tem método, caminho e versão (`GET /produto HTTP/1.1`); na resposta tem versão, código e descrição do status (`HTTP/1.1 200 OK`).
2. Cabeçalhos: pares `nome: valor`, um por linha.
3. Uma linha em branco, que separa os cabeçalhos do corpo.
4. Corpo: o conteúdo em si, que é opcional.

## 2. Número mínimo e máximo de cabeçalhos

O HTTP não define um número máximo de cabeçalhos. O que existe são limites práticos de tamanho impostos pelos servidores. Quanto ao mínimo, no HTTP/1.0 uma mensagem pode ter zero cabeçalhos. No HTTP/1.1, a requisição precisa de pelo menos o cabeçalho `Host`, enquanto a resposta não tem cabeçalho obrigatório pela especificação, embora na prática o servidor sempre envie alguns, como `Date` e `Content-Type`.

## 3. Limitações do conteúdo do corpo

- O corpo é opcional: requisições GET e DELETE normalmente não têm corpo.
- Alguns status não admitem corpo, como o `204 No Content`, e a resposta a um `HEAD` também não tem.
- O tamanho precisa ser informado por `Content-Length` (ou por transferência em partes, `chunked`), e o tipo precisa ser informado por `Content-Type`.
- O corpo trafega como texto ou bytes, então dados binários precisam de um tipo MIME adequado ou de codificação.
- Os servidores costumam limitar o tamanho máximo do corpo que aceitam.

## 4. Principais características do TCP

- Orientado à conexão: antes de trocar dados, abre a conexão com o *three-way handshake* (SYN, SYN-ACK, ACK).
- Confiável: usa confirmações (ACK) e retransmite o que se perdeu.
- Entrega ordenada: usa números de sequência para reordenar os segmentos.
- Controle de fluxo: evita que o receptor seja sobrecarregado.
- Controle de congestionamento: reduz a velocidade quando a rede está saturada.
- Full-duplex: os dados podem trafegar nos dois sentidos ao mesmo tempo.
- Encerramento organizado da conexão, com troca de mensagens FIN.

## 5. Pilha de protocolos da web

```
+-----------------------------------------+
|  Aplicação:  HTTP                       |
+-----------------------------------------+
|  Transporte: TCP                        |
+-----------------------------------------+
|  Internet/Rede: IP                      |
+-----------------------------------------+
|  Enlace/Física: Ethernet, Wi-Fi, etc.   |
+-----------------------------------------+
```

Cada camada usa os serviços da camada de baixo. O HTTP monta a mensagem e entrega ao TCP, que a divide em segmentos e garante a entrega confiável. O TCP entrega os segmentos ao IP, que os encapsula em pacotes e os roteia pela rede. Por fim, a camada de enlace e física transmite os bits pelo meio (cabo, Wi-Fi). No destino, o processo se inverte, subindo camada por camada.

## 6. Fluxo de uma transação HTTP GET entre um Arduino e um serviço web

7. O servidor envia a resposta pelo TCP, e o Arduino confirma os segmentos recebidos com ACK.
8. O Arduino lê a resposta (linha de status, cabeçalhos e corpo) e usa os dados.
9. **Encerramento do TCP:** quem fecha primeiro envia FIN, o outro lado responde com ACK e depois envia o próprio FIN, que também é confirmado. A conexão é encerrada.

## 7. URL e sua composição

URL (Uniform Resource Locator) é o endereço que localiza um recurso na rede. Exemplo:

`http://vasscommerce.com.br:80/categoria/1/produto?nome=Note#topo`

- **Esquema (protocolo):** `http`, define como acessar o recurso.
- **Host (domínio):** `vasscommerce.com.br`, identifica o servidor.
- **Porta:** `80`, é opcional, e o padrão do HTTP é 80 (HTTPS usa 443).
- **Caminho (path):** `/categoria/1/produto`, indica o recurso dentro do servidor.
- **Query string:** `?nome=Note`, parâmetros enviados ao servidor, no formato `chave=valor` e separados por `&`.
- **Fragmento:** `#topo`, indica uma parte do recurso e é tratado só pelo cliente.

## 8. Diferença entre URL e URI

URI (Uniform Resource Identifier) é o conceito mais amplo: qualquer identificador de um recurso. URL é um tipo de URI que, além de identificar, diz onde o recurso está e como acessá-lo. Outro tipo de URI é a URN, que só dá o nome do recurso, como `urn:isbn:9788575228104`. Toda URL é uma URI, mas nem toda URI é uma URL.

## 9. Diferença entre Web e Internet

A Internet é a infraestrutura física e lógica: a rede global de computadores interligados que usam o protocolo IP. A Web (World Wide Web) é um serviço que roda sobre a Internet, formado por recursos acessados por URLs e transferidos por HTTP. A Internet também suporta outros serviços, como e-mail (SMTP), transferência de arquivos (FTP) e acesso remoto (SSH).

## 10. Arquitetura Cliente-Servidor

É um modelo em que a comunicação é dividida em dois papéis. O cliente é quem inicia a comunicação e faz requisições (um navegador, um app, um Arduino). O servidor fica aguardando, processa as requisições e devolve respostas. Um mesmo servidor atende vários clientes, concentrando regras de negócio e dados. No HTTP, o cliente sempre começa a conversa e o servidor só responde.

## 11. Códigos de status HTTP e suas cinco classes

Os códigos de status são números de três dígitos que o servidor envia na resposta para informar o resultado da requisição. A classe é definida pelo primeiro dígito:

- **1xx:** informativo
- **2xx:** sucesso
- **3xx:** redirecionamento
- **4xx:** erro do cliente
- **5xx:** erro do servidor

## 12. Explicação de cada classe, com exemplos

- **1xx (Informativo):** a requisição foi recebida e o processo continua. Exemplo: `100 Continue`, `101 Switching Protocols`.
- **2xx (Sucesso):** a requisição foi recebida, entendida e aceita. Exemplos: `200 OK`, `201 Created`, `204 No Content`.
- **3xx (Redirecionamento):** o cliente precisa tomar outra ação, normalmente acessar outra URL. Exemplos: `301 Moved Permanently`, `302 Found`, `304 Not Modified`.
- **4xx (Erro do cliente):** a requisição está errada ou não pode ser atendida. Exemplos: `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`.
- **5xx (Erro do servidor):** a requisição parece válida, mas o servidor falhou ao processá-la. Exemplos: `500 Internal Server Error`, `502 Bad Gateway`, `503 Service Unavailable`.

## 13. MIME Types

MIME Types são identificadores que indicam o tipo e o formato de um conteúdo, no formato `tipo/subtipo`. Servem para o receptor saber como interpretar o corpo da mensagem. Exemplos: `text/html`, `text/plain`, `application/json`, `image/png`, `image/jpeg`, `application/pdf`.

## 14. Métodos HTTP mais comuns

Métodos HTTP indicam qual ação o cliente quer executar sobre um recurso.

- **GET:** busca e retorna dados de um recurso, sem alterá-lo.
- **POST:** envia dados ao servidor para criar um novo recurso.
- **PUT:** substitui/atualiza um recurso existente com os dados enviados.
- **PATCH:** atualiza parcialmente um recurso.
- **DELETE:** remove um recurso.
- **HEAD:** igual ao GET, mas retorna só os cabeçalhos, sem corpo.

## 15. Métodos HTTP e CRUD

| Operação CRUD | Método HTTP |
|---|---|
| Create (criar) | POST |
| Read (ler) | GET |
| Update (atualizar) | PUT (completo) e PATCH (parcial) |
| Delete (excluir) | DELETE |

## 16. Idempotência

Um método é idempotente quando executá-lo uma vez ou várias vezes seguidas produz o mesmo efeito no servidor. Por exemplo, excluir o mesmo produto duas vezes deixa o sistema no mesmo estado. São idempotentes: **GET, PUT, DELETE, HEAD, OPTIONS e TRACE**. Não são idempotentes: **POST** (cada chamada pode criar um novo recurso) e **PATCH** (em geral).

## 17. CONNECT, OPTIONS e TRACE

- **CONNECT:** pede ao servidor (normalmente um proxy) que estabeleça um túnel TCP até o destino. É muito usado para tráfego HTTPS passando por proxy.
- **OPTIONS:** pergunta ao servidor quais métodos e opções ele aceita para um recurso. É usado, por exemplo, nas requisições *preflight* do CORS.
- **TRACE:** o servidor devolve a própria requisição recebida, como um eco. Serve para diagnóstico, mas costuma ser desabilitado por motivos de segurança.

## 18. MIME Types e tipos de arquivo

- `application/vnd.ms-excel`: planilha do Microsoft Excel (formato `.xls`).
- `application/json`: dados em formato JSON.
- `video/mp4`: vídeo no formato MP4.
- `application/pdf`: documento PDF.
- `text/javascript` e `application/javascript`: ambos são arquivos JavaScript. O `application/javascript` era o tipo mais usado antigamente, mas foi marcado como obsoleto, e hoje o padrão oficial recomendado é o `text/javascript`. Na prática, os navegadores aceitam os dois.
- `text/css`: folha de estilo CSS.

## 19. Cabeçalhos e onde aparecem

| Cabeçalho | Significado | Aparece em |
|---|---|---|
| Content-Language | Idioma do conteúdo do corpo (ex.: `pt-BR`) | Requisição e resposta |
| Content-Type | Tipo MIME do corpo (ex.: `application/json`) | Requisição e resposta |
| Content-Encoding | Compressão aplicada ao corpo (ex.: `gzip`) | Requisição e resposta (mais comum na resposta) |
| Content-Length | Tamanho do corpo em bytes | Requisição e resposta |
| Accept | Tipos MIME que o cliente aceita receber | Requisição |
| User-Agent | Identifica o cliente (navegador, app, versão) | Requisição |
| Server | Identifica o software do servidor | Resposta |
| Last-Modified | Data da última modificação do recurso | Resposta |
| Date | Data e hora em que a mensagem foi gerada | Requisição e resposta (sempre na resposta) |