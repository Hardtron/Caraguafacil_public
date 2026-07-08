# Caraguá Fácil 🌴 https://www.caraguafacil.com.br/

Caraguá Fácil é um portal web interativo, moderno e de alta performance para a cidade de Caraguatatuba/SP, focado na busca e divulgação de serviços locais premium e anúncios imobiliários. 

O projeto foi projetado para oferecer uma experiência de usuário excepcional ("wow"), combinando design premium com uma infraestrutura robusta, segura e moderna.

---

## 🛠️ Stack Tecnológica

O portal é construído e implantado utilizando o seguinte ecossistema de nuvem e ferramentas:

1. **Frontend**: React + Vite + TypeScript + TailwindCSS + Shadcn/ui (design do sistema e componentes visuais).
2. **Banco de Dados (Realtime & NoSQL)**: [Cloud Firestore](https://firebase.google.com/docs/firestore) (banco de dados NoSQL com sincronização em tempo real para chats, presença e atualizações).
3. **Autenticação**: [Firebase Authentication](https://firebase.google.com/docs/auth) (fluxo Passwordless Multi-Método: E-mail via Magic Link, Celular via OTP/SMS com autenticação MFA em duas etapas, e Login Social integrado com Google OAuth).

---

## 🔐 Fluxo de Autenticação Inteligente & Proteção Cross-Channel
 
 O portal implementa um fluxo avançado de autenticação desenhado para otimizar a experiência do usuário (UX), garantir a integridade dos dados e economizar recursos da nuvem:
 
 *   **Entrada Passwordless (Sem Senha)**: Login simples e seguro via Magic Link (e-mail), código de 6 dígitos SMS (celular) ou Google.
 *   **Pre-Check Antiduplicidade (Economia de Recursos)**: Antes de disparar SMS (OTP) ou enviar Magic Links, o sistema verifica no Firestore se o número ou e-mail já pertence a outra conta vinculada a outro método de login. Se houver duplicidade cruzada (ex: tentar logar com celular quando a conta original foi criada com Google), o envio de e-mail/SMS é **bloqueado previamente**, informando ao usuário qual método ele deve utilizar.
 *   **Gestão Segura de Alterações Cadastrais (E-mail e Telefone)**: Se o usuário editar suas informações de e-mail ou telefone dentro do painel, o sistema exige verificação imediata para concluir a ação. A alteração de e-mail envia um link de verificação (`verifyBeforeUpdateEmail`), que sincroniza ativamente com o Firestore após a confirmação. A alteração de telefone exige re-validação via SMS OTP (reCAPTCHA + código) antes de salvar.
 *   **MFA (Autenticação Multi-Fator)**: Usuários que entram por métodos tradicionais (e-mail ou Google) passam pelo fluxo de verificação de número de celular por SMS de forma segura, ativando a proteção em duas etapas (MFA) no Firebase.
 *   **Sincronização Comercial Automática**: Na conclusão de cadastro, todos os dados validados (incluindo biografia e localização via CEP/Bairro) são sincronizados em tempo real no perfil do usuário (`profiles`) e no seu registro comercial (`providers`), garantindo que o prestador de serviços seja ativado de imediato sem pendências.
 
 ---
 
 4. **Armazenamento de Mídia**: [Firebase Storage](https://firebase.google.com/docs/storage) (armazenamento e processamento de avatares, fotos de imóveis e serviços).
 5. **Hospedagem (CDN)**: [Firebase Hosting](https://firebase.google.com/docs/hosting) (distribuição global estática ultrarrápida via CDN do Google).
 6. **Backend Serverless**: [Google Cloud Run](https://cloud.google.com/run) (microsserviços escaláveis em Node.js dedicados para tarefas de processamento pesado).
 7. **CI/CD**: [GitHub Actions](https://github.com/features/actions) (automação completa de testes, build e deploy contínuo integrado diretamente com a branch `main`).

---

## 🏗️ Arquitetura de Funções (Cloud Run)

As tarefas de segundo plano e regras de negócio complexas rodam em microsserviços dedicados no Cloud Run (região `southamerica-east1` em São Paulo):

*   **`cf-send-email`**: Envio de e-mails transacionais e validação de prestadores de serviços via Resend API.
*   **`cf-gemini-chat`**: Assistente inteligente de IA integrado ao Google Gemini, que ajuda os usuários a buscarem serviços e imóveis no portal.
*   **`cf-moderate-content`**: Moderação automática de imagens e textos de anúncios com IA contra spam e conteúdo abusivo.
*   **`cf-scrape-news`**: Web scraper automatizado que captura notícias de portais e fontes oficiais da prefeitura de Caraguatatuba/SP.
*   **`cf-validate-upload`**: Validação de segurança em tempo real de mídias enviadas para o Storage.
*   **`cf-delete-user`**: Processamento e exclusão em cascata de dados do usuário e seus anúncios no banco.

---

## 🚀 Desenvolvimento Local

### Pré-requisitos
*   **Node.js** (versão 18+ recomendada)
*   **npm** ou **yarn**
*   **Firebase CLI** (opcional, para emuladores)

### Configuração
1.  **Instalar dependências**:
    ```sh
    npm install
    ```

2.  **Configurar variáveis de ambiente**:
    Crie um arquivo `.env.local` na raiz do projeto com base no arquivo `.env.example`, preenchendo as chaves do seu projeto do Firebase:
    ```env
    VITE_FIREBASE_API_KEY=seu-api-key
    VITE_FIREBASE_AUTH_DOMAIN=seu-projeto.firebaseapp.com
    VITE_FIREBASE_PROJECT_ID=seu-projeto-id
    VITE_FIREBASE_STORAGE_BUCKET=seu-projeto.firebasestorage.app
    VITE_FIREBASE_MESSAGING_SENDER_ID=seu-sender-id
    VITE_FIREBASE_APP_ID=seu-app-id
    ```

3.  **Iniciar o servidor de desenvolvimento**:
    ```sh
    npm run dev
    ```
    A aplicação estará disponível em `http://localhost:8080`.

---

## 📦 Deploy e Automação (CI/CD)

O pipeline de deploy está configurado no GitHub. Toda vez que alterações são enviadas ou um Pull Request é mesclado na branch `main`:
1.  O GitHub Actions intercepta e inicia o workflow.
2.  Instala as dependências de forma limpa (`npm ci`).
3.  Gera o build de produção do frontend (`npm run build`).
4.  Faz a implantação automática no Firebase Hosting.

Para fazer o deploy manual das regras de segurança ou das Cloud Functions do backend, utilize os comandos:
```sh
# Deploy das regras do banco e storage
npx firebase deploy --only firestore,storage

# Deploy de todas as Cloud Functions (requer gcloud CLI instalado e autenticado)
cd cloud-functions
npm run deploy:all
```

---

## 🛠️ Ferramentas Administrativas & Utilitários de Banco

Na pasta `cloud-functions/` estão inclusos scripts utilitários em Node.js para gerenciamento administrativo rápido e diagnósticos do banco de dados Firestore em produção:
- `setAdmin.cjs`: Promove imediatamente qualquer usuário (via e-mail) a Administrador do sistema no Firestore.
- `query_profile.cjs`: Consulta e exibe informações completas de um perfil do usuário no Firestore buscando por e-mail.
- `delete_ghost_profile.cjs`: Remove perfis fantasma órfãos (documentos remanescentes no Firestore de UIDs deletados do Firebase Auth) de forma a garantir a integridade dos e-mails e liberar novos cadastros.

