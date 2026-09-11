# 📋 Coletor de Logs Centralizado com Sockets UDP

![Java](https://img.shields.io/badge/Java-17%2B-orange?style=for-the-badge&logo=openjdk)
![Maven](https://img.shields.io/badge/Maven-3.8%2B-C71A36?style=for-the-badge&logo=apachemaven)
![UDP](https://img.shields.io/badge/Protocolo-UDP-blue?style=for-the-badge)
![License](https://img.shields.io/badge/Licen%C3%A7a-MIT-green?style=for-the-badge)

Projeto desenvolvido para a disciplina de **Sistemas Paralelos e Distribuídos**. Aplicação cliente/servidor concorrente em Java utilizando **Sockets UDP**, **Threads (ExecutorService)**, **Exclusão Mútua (`synchronized`)** e **Interface Gráfica em Swing** com atualização assíncrona (`SwingWorker`).

---

## 🎯 Sobre o Projeto

O **Coletor de Logs Centralizado** permite que múltiplas aplicações clientes acumulem eventos de log localmente (nível, origem e mensagem) e os enviem em lote ao servidor central através de datagramas UDP em formato JSON.

### 💡 Principais Desafios e Conceitos Aplicados:
* **Concorrência e Multithreading no Servidor:** O servidor utiliza um pool de 10 threads (`ExecutorService`) para processar as listas de logs recebidas em paralelo sem bloquear a porta UDP.
* **Proteção de Região Crítica:** Para simular um processamento pesado, cada evento da lista leva 1 segundo para ser gravado no repositório compartilhado. O acesso a essa lista central é protegido via `synchronized` para evitar **Condição de Corrida (Race Condition)**.
* **Comunicação Não-Bloqueante na Interface (SwingWorker):** O cliente Swing despacha requisições de rede em segundo plano, mantendo a interface responsiva durante o envio dos pacotes.

---

## ⚙️ Arquitetura do Protocolo JSON

A comunicação entre Cliente e Servidor ocorre via pacotes UDP utilizando a biblioteca **Gson** para serialização/desserialização dos objetos em JSON:

### ✉️ `Requisicao.java`
* **`operacao`**: `"REGISTRAR"` ou `"LISTAR"`.
* **`nivel`**: Filtro para consultas (`"TODOS"`, `"INFO"`, `"WARN"`, `"ERROR"`).
* **`eventos`**: Lista de objetos `Evento` no modo de registro.

### 📩 `Resposta.java`
* **`status`**: `"OK"` ou `"ERRO"`.
* **`timestamp`**: Data/hora do servidor no momento do processamento.
* **`dados`**: Lista de textos contendo confirmações, registros retornados ou mensagens de erro.

---

## 📁 Estrutura dos Projetos

```text
├── ColetorLogsServidor/
│   ├── src/main/java/com/mycompany/coletorlogsservidor/
│   │   ├── Evento.java
│   │   ├── Requisicao.java
│   │   ├── Resposta.java
│   │   ├── RepositorioLogs.java    # Região Crítica (synchronized)
│   │   └── ServidorApp.java        # Socket UDP (porta 9999) + ThreadPool
│   └── pom.xml
│
└── ColetorLogsCliente/
    ├── src/main/java/com/mycompany/coletorlogscliente/
    │   ├── Evento.java
    │   ├── Requisicao.java
    │   ├── Resposta.java
    │   ├── ServicoCliente.java     # Camada de comunicação UDP
    │   └── TelaCliente.java        # Interface Swing + SwingWorker
    └── pom.xml
```

---

## 🚀 Como Executar

### Pré-requisitos
* **Java JDK 17** ou superior instalado.
* **Apache Maven** instalado.

### 1. Clonar o Repositório
```bash
git clone https://github.com/hugbrl09/coletor-logs-udp.git
cd coletor-logs-udp
```

### 2. Iniciar o Servidor
Navegue até a pasta do servidor e execute via Maven:
```bash
cd ColetorLogsServidor
mvn clean compile exec:java
```
O servidor estará ativo e escutando na porta **`9999/UDP`**.

### 3. Iniciar o Cliente (Interface Gráfica)
Em um novo terminal, navegue até a pasta do cliente e execute:
```bash
cd ColetorLogsCliente
mvn clean compile exec:java
```

---

## 🧪 Testando a Concorrência (Intercalação de Logs)

1. Abra **duas ou três instâncias** simultâneas do projeto `ColetorLogsCliente`.
2. No **Cliente 1**, adicione 3 eventos à fila local (ex: `A1`, `A2`, `A3`).
3. No **Cliente 2**, adicione 3 eventos à fila local (ex: `B1`, `B2`, `B3`).
4. Clique no botão **"Enviar Fila ao Servidor"** em ambos os clientes quase ao mesmo tempo.
5. No painel de consulta de qualquer cliente, selecione o filtro `"TODOS"` e clique em **"Consultar Servidor"**.
6. **Resultado Esperado:** Como cada thread do servidor processa a lista aguardando 1 segundo por evento, os logs de `A` e `B` aparecerão **intercalados no histórico central** (ex: `A1`, `B1`, `A2`, `B2...`), demonstrando o processamento paralelo seguro na Região Crítica!

---

## 🛠️ Tecnologias Utilizadas

* **Linguagem:** Java 17
* **Redes:** `java.net.DatagramSocket`, `DatagramPacket`
* **Concorrência:** `java.util.concurrent.ExecutorService`, `SwingWorker`, `synchronized`
* **Interface:** Java Swing (GUI)
* **Serialização:** Google Gson 2.10.1