## 🤖 SDR Premium para Concessionárias, lojas de veículos (n8n Workflow)

### **Título do Projeto:** Automação de Qualificação e Atendimento de Leads (SDR) para Lojas de Veículos
**Nome do Workflow:** `n8n-agenteIA-sdr-lojasdeveiculos`

Este workflow de automação low-code opera como um **SDR (Sales Development Representative) virtual avançado**, rodando 24/7 no WhatsApp. Ele é projetado para interagir com leads da loja de veículos Tompes Motors, qualificar intenções de compra/troca, realizar simulações complexas e encaminhar o cliente de forma precisa para o consultor humano. O sistema utiliza IA e orquestração de dados.

---

### **Características e Funcionalidades Principais**

Este sistema é uma solução completa de atendimento e qualificação baseada em IA e orquestração de dados:

* **Atendimento Multimídia:** Processa e responde a mensagens de texto, áudio (via transcrição OpenAI) e imagens (com tratamento via Visão de IA).
* **Gestão de Estoque e Vendas:** Utiliza um catálogo de veículos injetado no System Prompt do Agente para responder perguntas específicas sobre modelos, anos e preços.
* **Envio de Mídia Inteligente:** O agente aciona a **tool `enviar_fotos`** sempre que o cliente solicita imagens de um veículo do estoque. A resposta é um link direto para o veículo no site da loja (conforme a lógica do prompt).
* **Base de Conhecimento Dinâmica (Vector Store):** Monitora uma pasta do **Google Drive** para extrair dados de documentos (PDF, Excel, Docs), deletar dados antigos e criar novos embeddings, **mantendo a base de conhecimento (estoque e informações da loja) sempre atualizada de forma automática.**
* **Qualificação Inteligente:** Classifica leads em **Normal** ou **Premium** com base na intenção (interesse em estoque, oferta de carro para troca, e solicitação de simulação).

#### **Simulação e Avaliação:**

* **Cálculo FIPE (Tool):** Solicita dados textuais do carro do cliente (Modelo, Ano, Combustível) e retorna uma **faixa de avaliação estimada** (FIPE ± 20%), alertando que o valor final é presencial.
* **Simulação de Financiamento (Tool):** Recebe o valor a financiar e simula parcelas fixas (24, 36, 48, 60x), mantendo a complexidade do cálculo oculta.

---

### **Arquitetura (Nodes e Tecnologias)**

O workflow é construído sobre uma arquitetura de Low-Code e IA, integrando as seguintes ferramentas:

| Ferramenta | Uso no Workflow |
| :--- | :--- |
| **n8n** | Orquestração, Lógica e Conectividade. |
| **OpenAI (Agente/GPT)** | Lógica conversacional, cálculo de FIPE/Simulação, transcrição de áudio e análise/tratamento de imagens. |
| **Google Drive** | **Gatilho e Download de Arquivos de Estoque:** Monitora alterações de documentos (PDF, Excel, Docs) para atualização do Vector Store. |
| **AvisaApi** | Envio de mensagens e controle de `typing/start` no WhatsApp. |
| **Redis** | Armazenamento temporário de estado (Buffer de mensagens e status de atendimento). |
| **Supabase (PostgreSQL)** | Banco de dados para contatos e histórico de follow-up. |
| **Vector Store (Supabase)** | Base de conhecimento para o Agente IA (estoque de veículos e informações da loja). |

---

### **Visão Geral do Fluxo Principal**

1.  **Gatilho:** Uma mensagem recebida via **Webhook**.
2.  **Monitoramento de Estoque (Nodes Inferiores):** Gatilhos do **Google Drive** (`File Created`/`File Updated`) iniciam a rotina de **Vector Store:**
    * O arquivo é baixado e seu ID é setado.
    * Os nós de Supabase deletam documentos antigos com o mesmo ID.
    * O Text Splitter processa o texto do documento e o Embeddings OpenAI o vetoriza, inserindo o novo conteúdo no Supabase Vector Store.
3.  **Processamento de Mensagem:** O fluxo principal extrai dados essenciais, verifica o status de atendimento e o buffer de mensagens.
4.  **Reconhecimento de Mídia:** O `Switch` encaminha mídias para a transcrição de áudio (`AUDIO`) ou tratamento de imagem (`IMAGEM`).
5.  **Agente IA (`ESPECIALISTA EM VEICULOS`):** O coração do sistema, que utiliza memória, o **Vector Store (Estoque atualizado)** e as **Tools** (FIPE, Simulação, Vendedor, Fotos) para gerar a resposta.
6.  **Formatação e Envio:** O `Parser Chain` e o `Split de Mensagem` formatam e dividem a resposta em pequenas mensagens (máximo de 200 caracteres) antes de enviar via AvisaApi.

---

### **Como Usar este Workflow (Importação)**

* **Baixar:** Faça o download do arquivo `lojasdeveiculos.json`.
* **Importar:** Na sua instância n8n, clique em **"Import from File"** e selecione o arquivo JSON.
* **Configurar Credenciais:** O usuário deverá configurar as credenciais (Redis, PostgreSQL, Supabase, **Google Drive OAuth2** e OpenAI) e o token da AvisaApi.
* **Customização do Agente:** **Substitua todas as variáveis na Sticky Note (System Prompt)** com as informações da sua loja (Nome, Endereço, Estoque, etc.).
* **Configuração do Drive:** Configure os **Gatilhos Google Drive** para monitorar a pasta onde estão os arquivos do seu estoque.
