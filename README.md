Documentação do Projeto Chatbot de IA Multicanal

1. Visão Geral do Projeto

Este projeto consiste no desenvolvimento e implantação de um chatbot de Inteligência Artificial robusto e flexível, capaz de interagir com usuários em diversas plataformas de comunicação. O objetivo principal é demonstrar a capacidade de integrar modelos avançados de IA (como o Google Gemini) a fluxos de trabalho automatizados, provendo respostas inteligentes e personalizadas.

A solução visa ser um ponto de partida para a criação de assistentes virtuais, suporte ao cliente, automação de tarefas, ou qualquer aplicação que se beneficie de uma comunicação inteligente e escalável.

2. Arquitetura da Solução

A arquitetura do chatbot é baseada em contêineres Docker, orquestrados via Docker Compose, garantindo portabilidade, escalabilidade e isolamento dos serviços. O "cérebro" da IA é fornecido pelo Google Gemini, enquanto a comunicação com o WhatsApp é gerenciada por uma API gateway.

- Docker e Docker Compose:  A espinha dorsal da implantação. O Docker permite empacotar cada serviço (N8N, WAHA, Bancos de Dados) em seu próprio contêiner isolado. O Docker Compose orquestra esses contêineres, definindo como eles são construídos, executados e como se comunicam entre si.
- N8N: Atua como o motor de automação e orquestração do fluxo de trabalho (workflow). Ele recebe as mensagens do WhatsApp (via WAHA), processa-as, interage com a IA e coordena o envio das respostas. É uma plataforma low-code/no-code que simplifica a criação de lógicas complexas.
- WAHA (WhatsApp Helper API): É um gateway de API para o WhatsApp. Ele se conecta a uma sessão do WhatsApp (escaneando um QR Code) e expõe uma API local que o N8N utiliza para receber mensagens (via webhooks) e enviar respostas para o WhatsApp.
- PostgreSQL (Postgres): Servindo como o banco de dados principal do N8N. Ele armazena persistentemente todos os dados do N8N, incluindo os workflows criados, as credenciais configuradas, e o histórico de execuções. É um banco de dados relacional robusto para dados estruturados.
- Redis: Utilizado como um banco de dados em memória de alta performance. No contexto deste projeto, o Redis é fundamental para o recurso de "memória de chat" da IA no N8N, permitindo que o bot se lembre do contexto e das interações anteriores com cada usuário de forma rápida e eficiente. Também pode ser usado para cache e filas.
- Google Gemini API: É o serviço de inteligência artificial da Google que fornece as capacidades de linguagem natural e geração de texto para o chatbot. O N8N se comunica com esta API para obter as respostas inteligentes para as perguntas dos usuários.

Como os Componentes se Conectam:
Os contêineres do Docker Compose operam dentro de uma rede virtual isolada. O N8N se conecta ao WAHA (geralmente via `http://host.docker.internal:3000`), ao PostgreSQL (para seus dados) e ao Redis (para a memória da IA) usando endereços de rede internos fornecidos pelo Docker. A comunicação externa, como as mensagens do WhatsApp chegando ao WAHA, é gerenciada por um Webhook configurado no WAHA.

 3. Pré-requisitos

Para configurar e rodar este projeto em seu ambiente local, você precisará dos seguintes softwares e contas:

- Docker Desktop: Aplicativo essencial para rodar os contêineres Docker no seu sistema operacional (Windows, macOS ou Linux).
    - Download: [https://docker.com](https://docker.com)
    - No Linux, pode ser necessário `sudo apt install docker-compose` adicionalmente.
- Conta Google com Acesso à API do Google Gemini: Necessário para obter a chave de API que dará inteligência ao seu bot.
    - Geração da Chave de API: [https://aistudio.google.com/app/apikey](https://aistudio.google.com/app/apikey)
- Um número de WhatsApp para teste: Você precisará de um número de WhatsApp secundário (não o seu principal, se puder) para conectar ao WAHA. Este número será o "número do seu bot" para fins de teste.

 4. Configuração e Instalação (Passo a Passo Detalhado)

Aqui, você irá configurar o ambiente e iniciar os serviços do chatbot.

- 4.1. Download do Arquivo Docker Compose:
    - Faça download do arquivo `docker-compose.yml` específico para esta configuração N8N/WAHA (fornecido em seu curso ou repositório do projeto).
        - Link: `https://guilhermelaz.com.br/n8n-waha-...` (Nota: Este link deve ser verificado para acesso ao `docker-compose.yml` específico do guia N8N/WAHA.)

- 4.2. Preparação do Ambiente Local:
    - Crie uma nova pasta em um local de sua preferência no seu computador (ex: `C:\Projetos\ChatbotWAHAN8N`).
    - Mova o arquivo `docker-compose.yml` que você baixou para dentro desta nova pasta.

- 4.3. Iniciar os Serviços Docker:
    - Abra um terminal (PowerShell ou Prompt de Comando) e navegue até a pasta que você criou no passo anterior (onde está o `docker-compose.yml`).
        - Exemplo: `cd C:\Projetos\ChatbotWAHAN8N`
    - Execute o comando para baixar as imagens Docker e iniciar todos os serviços em segundo plano:
        ```bash
        docker-compose up -d
        ```
    - Aguarde a conclusão do comando. Isso pode levar alguns minutos na primeira vez, enquanto o Docker baixa as imagens.
    - Verifique se todos os contêineres estão rodando com: `docker ps`. Você deverá ver `n8nwahalocal-n8n-1`, `n8nwahalocal-waha-1`, `n8nwahalocal-redis-1`, `n8nwahalocal-postgres-1` (ou nomes similares) com status `Up`.

- 4.4. Conectar o WhatsApp ao WAHA:
    - Acesse o Dashboard do WAHA pelo seu navegador. A porta padrão é `3000`. No Docker Desktop, você pode clicar na porta mapeada para o contêiner `n8nwahalocal-waha-1`. Alternativamente, digite no navegador: `http://localhost:3000`
    - Dentro do Dashboard do WAHA, clique em **"Dashboard"** (geralmente na parte superior).
    - Localize a sessão "default" e clique em "Login".
    - Um QR Code será exibido. Use o seu WhatsApp no celular (o número que você reservou para testes) para escanear este QR Code (Vá em WhatsApp > Configurações > Aparelhos Conectados > Conectar um aparelho).
    - Após o escaneamento, a sessão no Dashboard do WAHA deve mudar para "CONNECTED" ou "WORKING".

- 4.5. Configurar o N8N:
    - Acesse o N8N pelo seu navegador. A porta padrão é `5678`. No Docker Desktop, você pode clicar na porta mapeada para o contêiner `n8nwahalocal-n8n-1`. Alternativamente, digite no navegador: `http://localhost:5678`
    - Na primeira vez, você será guiado a criar uma conta de usuário (admin) para o N8N. Siga as instruções.
    - Ativação do N8N (Opcional, mas libera mais recursos):
        - No painel do N8N, vá nos três pontinhos no canto inferior esquerdo, clique em "Settings" (Configurações).
        - Na tela que aparece, clique em "Enter Activation Key" (Inserir Chave de Ativação).
        - Informe o e-mail que você usou para sua conta N8N para receber a chave gratuita, ou use uma chave que já tenha.
    - Instalar Community Node do WAHA:
        - Ainda em "Settings" (Configurações), vá para a aba "Community Nodes".
         Clique em "Install a community node".
         No campo de pesquisa, digite: `n8n-nodes-waha`
        *Clique em "Install" e aguarde a conclusão.
        *Após a instalação, pode ser necessário reiniciar o N8N (desligando e ligando o contêiner `n8nwahalocal-n8n-1` via Docker Desktop ou `docker-compose restart n8n` no terminal).

 5. Como Usar o Bot (Fluxo de Trabalho no N8N)

Agora que todos os serviços estão funcionando, vamos montar o workflow que dá vida ao seu chatbot.

- 5.1. Criar um Novo Workflow:
 - No N8N, vá para "Workflows" e clique em "New Workflow" (ou no sinal `+`).

- 5.2. Node de Webhook (Trigger - Onde as mensagens chegam):
  - Adicione um nó "Webhook" ao seu workflow (pesquise por "Webhook" em "Triggers").
  - Configure:
     - HTTP Method:`POST`
    - Path: `webhook` (ou um nome de sua escolha, ex: `whatsapp-in`).
    - Copie a URL de Teste do Webhook: Essa URL aparecerá logo acima das configurações do nó Webhook (ex: `http://localhost:5678/webhook-test/seu-id-do-webhook`). Guarde-a.

- 5.3. Conectar o Webhook do WAHA ao N8N:
    - Acesse o Dashboard do WAHA novamente: `http://localhost:3000`
    - Vá nas configurações da sua sessão (clique na sessão "default").
    - Clique em "+ Webhook".
    - Cole a URL de Teste do Webhook do N8N (que você copiou no passo anterior) no campo de URL.
    - Em "events", deixe apenas o "message" selecionado.
    - Clique em "Update" para salvar.

- 5.4. Node "Edit Fields" (Set) - Organizar os Dados:
    - Adicione um nó "Edit Fields" (Set), conectado à saída do nó "Webhook".
    - Configure este nó com os seguintes campos (TODOS como "Expression", usando o ícone `</>`):
        - `session`: `{{ $item.json.session }}`
        `event`: `{{ $item.json.event }}`
        `chatId`: `{{ $item.json.payload.from }}`
       `pushName`: `{{ $item.json._data.PushName }}`
       `payload_id`: `{{ $item.json.payload.id }}`
        `message`: `{{ $item.json.payload.body }}`
        `fromMe`: `{{ $item.json.payload.fromMe }}`
    - Certifique-se de que "Include Other Input Fields" está desativado.
    - Teste: Envie uma mensagem do seu segundo WhatsApp para o número conectado no WAHA. No N8N, vá para a aba "Executions", clique na execução mais recente, clique no nó "Edit Fields" e veja a saída JSON para confirmar que os 7 campos estão corretos.

- 5.5. Node "Switch" - Filtrar Mensagens:**
 - Adicione um nó **"Switch"**, conectado à saída do nó "Edit Fields".
  - Configure:
   - Mode: `Basic`
    - Condition 1:
     - Value 1 (Expression):`{{ $json.event }}`
     - Operation: `Is Equal`
       - Value 2: `message` (texto literal)
    - Este nó garantirá que o fluxo só prossiga para mensagens de texto.

- 5.6. Node "AI Agent" - O Cérebro da IA:
    - Adicione um nó "AI Agent", conectado à saída `True` (verdadeira) do nó "Switch".
    - Configure:
        - Prompt (User Message) (Expression): `{{ $json.message }}`
        - System Message: "Você é uma assistente de IA com uma personalidade muito gentil, atenciosa e prestativa. Sua especialidade é auxiliar alunos de AWS em um grupo de estudos focado em responder o máximo de perguntas da AWS para a conquista da certificação, além de oferecer suporte em códigos e programação relevantes para o ambiente AWS. Meu objetivo é oferecer informações úteis, claras e precisas sobre os tópicos da AWS, guiar discussões e dar suporte prático com exemplos de código, tudo de forma colaborativa, respeitosa e imparcial, garantindo que todos alcancem seus objetivos de certificação."
        
- Chat Model:
            - Provider: `Google Gemini`
            - Model: `gemini-1.5-flash`
            - Google Gemini Api Key: Selecione ou crie sua credencial (com sua chave `AIzaSy...`).
        - Memory: Ativar (`toggle` verde).
            - Type: `Redis Chat Memory`
            - Redis Chat Memory Credential: Selecione ou crie sua credencial (Host: `host.docker.internal`, Port: `6379`, Password: `default`).
            - Session ID (Expression): `{{ $json.chatId }}`
    - Teste: Envie uma nova mensagem do WhatsApp e verifique se o nó "AI Agent" fica verde na aba "Executions".

- 5.7. Node "WAHA Send Text Message" - Responder ao WhatsApp:
    - Adicione um nó "WAHA" (WAHA Send Text Message), conectado à saída do nó "AI Agent".
    - Configure:
        - WAHA Account: Selecione ou crie sua credencial (WAHA API URL: `http://host.docker.internal:3000`).
        - Session (Expression): `{{ $json.session }}`
        - Chat ID (Expression): `{{ $json.chatId }}`
        - Payload ID (Expression, em "Add Option"): `{{ $json.payload_id }}`
        - Text (Expression): `Resposta do Cérebro de IA: {{ $json.response_text }}`

## 6. Solução de Problemas Comuns (Troubleshooting)

Aqui estão os problemas mais frequentes e suas soluções:

- Nó "Edit Fields" (Set) com `[null]` ou texto literal:
    - Problema: As expressões como `{{ $item.json.session }}` ou `{{ $json.event }}` não estão extraindo os valores reais, mostrando `null`, `undefined` ou o próprio nome do campo.
    - Solução: Abra as configurações do nó "Edit Fields (Set)". Para CADA CAMPO, clique no ícone `</>` ao lado do campo de valor para forçar o modo "Expression". Salve, desative/ative o workflow, e teste com uma nova mensagem.
- "Error in sub-node 'Redis Chat Memory' No session ID found":
    - Problema: O nó AI Agent não consegue obter um ID de sessão válido para a memória.
    - Solução: Geralmente causado pelo `chatId` não estar correto. Verifique o output do nó "Edit Fields (Set)" se o `chatId` está sendo extraído corretamente. No nó "AI Agent", na aba "Memory", certifique-se de que "Session ID" está com `{{ $json.chatId }}` e em modo "Expression".
- "Google Gemini Chat Model" vermelho / "Couldn’t connect with these settings":
    - Problema: A conexão com a API do Google Gemini está falhando.
    - Solução:
        1.  Verifique sua chave de API do Gemini no Google AI Studio. Copie-a com muito cuidado, sem espaços extras. No N8N (nó AI Agent -> Chat Model -> Credencial Gemini), clique no lápis para editar e cole a chave novamente, depois salve.
        2.  Certifique-se de que o "Model" selecionado é `gemini-1.5-flash`.
- "Problem activating workflow: Please resolve outstanding issues":
    - Problema: Há um erro em algum nó do workflow que impede a ativação.
    - Solução: Procure o nó com o círculo vermelho e o 'X'. Clique nele para ver a mensagem de erro específica e resolva-a.
- Nós com erros persistem mesmo após correção / N8N não carrega novas configs:
    - Problema: O N8N pode estar com um cache ou estado persistente.
    - Solução:
        1.  Desative o workflow no N8N e feche o navegador.
        2.  No terminal, vá para a pasta do seu `docker-compose.yml` do N8N/WAHA (`C:\Users\felip\Downloads\N8N WAHA Local`).
        3.  Execute `docker-compose down` e depois `docker-compose up -d` para reiniciar os contêineres do N8N/WAHA/Redis/Postgres.
        4.  Acesse o N8N novamente e ative o workflow.
- Nó WAHA vermelho / "Your request is invalid or could not be processed":
    - Problema: A requisição do N8N para o WAHA é inválida ou a sessão do WAHA não está ativa.
    Solução:
        1.  Verifique a sessão do WAHA: Acesse `http://localhost:3000`. Confirme se a sessão está "WORKING" ou "CONNECTED". Se não, re-escaneie o QR Code.
        2.  Verifique a "Operation": No nó WAHA, certifique-se de que a "Operation" está como "Send Text Message".
        3.  Verifique o formato do Chat ID: Embora o `chatId` do nó "Set" seja `558191361377@c.us`, alguns WAHA podem preferir apenas o número (`558191361377`). Se o problema persistir, tente modificar o `chatId` no nó "Set" para `{{ $item.json.payload.from.split('@')[0] }}`.

## 7. Próximos Passos e Ambiente de Produção

A solução atual roda localmente com o Docker Compose, utilizando um gateway de API WhatsApp e o N8N. Para um ambiente de produção real e escalável, são necessários passos adicionais:

- Sair do Sandbox do WAHA: O WAHA é ótimo para testes, mas para um uso comercial oficial, você precisaria da WhatsApp Business API (WABA) oficial da Meta. Isso envolve:
    - Registro e verificação de uma conta de negócios na Meta Business Manager.
    - Trabalhar com um Provedor de Solução de Negócios (BSP) como a Twilio (em um plano pago) para gerenciar o acesso à WABA.
    - Obtenção de um número de telefone dedicado e aprovado para a WABA.
    - Criação e aprovação de "Modelos de Mensagem" para iniciar conversas com usuários fora da janela de 24 horas.

- Hospedagem em Nuvem (Substituindo o Ambiente Local):
    - Você precisará de um provedor de nuvem (AWS, Google Cloud, Azure, DigitalOcean, Render.com, Heroku) para hospedar os contêineres do N8N, WAHA, PostgreSQL e Redis 24/7.
    - Serviços de contêiner gerenciados (como AWS Fargate/ECS, Google Cloud Run/GKE, Azure Container Apps) simplificam a implantação de aplicativos Docker.
    - Um domínio próprio e um certificado HTTPS são essenciais para Webhooks em produção.

- Gerenciamento de Mídias (Envio e Recebimento de Imagens/Vídeos):
    - Recebimento: O WAHA já envia URLs de mídias recebidas (nos campos `MediaUrlX`). Seu workflow no N8N pode baixar essas mídias e, opcionalmente, processá-las com a capacidade multimodal do Google Gemini (se usar modelos como `gemini-1.5-pro`).
    - Envio: Para enviar mídias, você precisaria de um serviço de armazenamento de objetos na nuvem (ex: AWS S3, Google Cloud Storage) para hospedar as imagens/vídeos. O nó WAHA de envio aceita URLs públicas para mídias (`media_url`).
