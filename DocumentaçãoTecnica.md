# Manual e Documentação Técnica - SAFE v11
(Sistema de Avaliação em Anestesiologia e Formulários Eletrônicos)

## 1. Visão Geral do Sistema

### Objetivo do Programa
O SAFE v11 foi desenvolvido para digitalizar, otimizar e unificar o processo de Avaliação Pré-Anestésica (APA). Ele centraliza a interface onde médicos anestesiologistas podem revisar dados enviados por seus pacientes previamente ou fazer a leitura automatizada de laudos em PDF/imagem (via OCR).

### Problema que ele resolve
Tradicionalmente, a coleta de histórico médico pré-operatório é demorada e suscetível a erros. Pacientes preenchem formulários em papel que precisam ser digitados pelo médico. O SAFE resolve isso ao:
1. Permitir a pré-coleta de dados do paciente via formulário web próprio.
2. Ler laudos e formulários automaticamente utilizando tecnologias de OCR (Reconhecimento Ótico de Caracteres) e PDF parsing, economizando tempo considerável na digitação de exames laboratoriais e textos repetitivos.
3. Centralizar calculadoras médicas, avaliação de riscos e formulários do processo anestésico de maneira segura e digital.

### Público-Alvo / Usuários
- **Pacientes**: Respondem a um questionário simplificado remotamente via e-mail (Módulo Pré-SAFE).
- **Médicos Anestesiologistas**: Utilizam o Dashboard completo para consolidar os dados, processar os laudos automaticamente, calcular escores de risco (ARISCAT, etc.) e registrar a evolução anestésica no sistema de forma completa.

---

## 2. Manual de Uso

### Como acessar o sistema
1. Acesse a URL que aponta para `safe_login.html`.
2. Insira as credenciais (E-mail e Senha). 
3. *Novos médicos* devem utilizar a aba de "Cadastro" usando um **Código de Convite** fornecido pela administração e aguardar aprovação dos mantenedores do sistema via painel administrador.

### Como utilizar cada funcionalidade principal
- **Dashboard**: Após o login, a tela inicial (`safe_dashboard.html`) exibe os atalhos para Novo Formulário, Carregar Pré-SAFE, Enviar formulário ao paciente, além da aba de configuração e calculadoras médicas anexas.
- **Carregar Pré-SAFE**: Clicando nesta opção, você é levado para a tela de importação de dados dos pacientes (`safe_carregar_presafe.html`). Existem duas abas:
  - **Token**: Permite inserir um código ou link web proveniente da resposta prévia do paciente do banco de dados na nuvem.
  - **Upload Local (PDF)**: Faça o upload ou arraste o PDF do paciente.
- **Formulário Completo**: Na etapa final de fluxo longo (`safe-wrapper-final.html`), faça a revisão técnica. Utilize as flags de expandir e os sistemas automáticos (como a "Mágica do ECG/Lab" ou Extração de Comorbidades) que leem os textos do paciente preenchendo caixas de checagem.

### Passo a passo de um fluxo laboratorial e de avaliação (Exemplo)
1. Arraste o arquivo `.pdf` referente ao laudo do paciente no Menu Pré-SAFE.
2. O sistema executará internamente o reconhecimento de blocos textuais.
3. Aprove a importação inicial (dados pessoais, cirurgia e comorbidades).
4. Dentro do formulário principal, clique no botão para "Extração de Laboratório". Arraste um laudo de exames de sangue ou cole texto na caixa. O sistema usará as expressões para encontrar Plaquetas, Glicose, Hematócritos, Creatinina, preenchendo tudo de imediato.
5. Finalizado, clique para exportar, enviar ou salvar sua avaliação pré-anestésica.

### Possíveis Erros e Soluções
- **"Código muito curto" ao buscar Pré-SAFE**: O usuário não colou o token completo. Verifique o link e tente novamente.
- **OCR falha ou texto não capturado**: Pode ocorrer em fotos de baixa resolução enviadas. Escaneie diretamente do PDF nativo se possível, ou opte pela opção colar "Texto Manual".
- **Formulário volta imprevistamente para Login**: O `sessionStorage` expirou ou sua guia do navegador foi isolada limpando a sessão. Entre no sistema novamente.
- **Bloqueio ao não carregar Tesseract/OCR**: Verifique restrições de rede locais cortando acesso aos pacotes de `CDN` (WebAssembly) usados pela biblioteca de OCR.

---

## 3. Fluxo do Sistema

### Fluxo de Navegação entre Telas
`Login` ➔ `Dashboard` ⇌ (`Atalhos Rápidos` e `Calculadoras`)
`Dashboard` ➔ `Iniciar Pré-SAFE / Upload` ➔ `Revisão Intermediária de Campos` ➔ `Formulário Completo (Edição e Análise)` ➔ `Ação Final (Baixar/Salvar)`

### Fluxo Técnico e de Dados (Processo de Execução)
1. O usuário é autenticado rigorosamente usando token Firebase. Os dados autorizados ficam alocados em variáveis locais encriptadas até que a aba seja fechada.
2. O médico inicia a captura (por nuvem via Token ou processamento local PDF).
3. A inteligência do `carregar-presafe.js` converte imagens de PDF e formata as posições mapeando cada informação (Nome, CPF, Negações de alergia, Hábitos) num objeto JSON puro de troca de estado.
4. O Objeto cruza as `sessionStorage` e abre o `safe-form.js`.
5. Ao construir a página, dezenas de `listeners` observam o Objeto local. Checkboxes de exames de mama, medicações e cirurgias assumem valores instantaneamente em tela. O médico altera os estados livremente e revisa.

---

## 4. Fluxo do Código

### Estrutura de Pastas e Arquivos
- **`/Pages/`**: Esqueleto visual do sistema contendo Views como Login, Dashboard, e Wrapper principal de formulário.
- **`/Scripts/`**: Responsável pela vida e inteligência do front-end. O cérebro do software.
- **`/Styles/`**: Folhas de estilo isoladas (`base.css`, `components.css`, `dashboard.css`), favorecendo manutenção de UI sem choques com frameworks externos.

### Função dos Principais Módulos Arquiteturais
- **`core.js`**: Raiz global do SAFE. Instancia o Firebase, define os parâmetros de infraestrutura (EmailJS) e implementa o componente `SAFE_STORAGE` que criptografa (CryptoJS/AES) qualquer entrada persistente entre as páginas.
- **`auth.js`**: Motor autônomo de login, cadastro, controle de rastro de token em `IndexedDB` e limites de autorização entre rotas estáticas.
- **`dashboard.js`**: Controlador de interface do menu principal. Modela os menus dos usuários, escuta estado online e valida regras preliminares.
- **`carregar-presafe.js`**: Processador assíncrono avançado para análise de dados. Responsável por invocar o Web Worker nativo do `pdf.js` e do algoritmo de OCR. Limpa ruídos do texto de imagens e aplica Expressões Regulares (`RegEx`) densas para identificar informações fragmentadas e devolvê-las limpas para visualização médica.
- **`safe-form.js`**: O coração das lógicas anestésicas e o arquivo de maior robustez estrutural no sistema. Realiza os cálculos contínuos das interações na interface principal e processa as dinâmicas laboratoriais com regras estritas descritas pela medicina clínica.

### Relação entre Frontend, Backend e Banco de Dados
A interface e a orquestração do SAFE contrapõem-se a modelos convencionais pela sua **Arquitetura Serverless Totalmente Front-End Oriented**:
As máquinas host em "Backend" do código inexistem no modelo relacional, em seu lugar o client-side injeta bibliotecas que interagem diretamente de forma assíncrona ao Firebase SDK. O peso computacional de leitura documental está no navegador da máquina (mantendo total privacidade médica e conformidade à LGPD localmente) com comunicação em nuvem (BaaS) restrita aos repositórios autorais do banco, que valida quem pede e como pede (`Firestore`).

---

## 5. Documentação Técnica

### Tecnologias e Camadas Utilizadas
- **Web Pura Base**: HTML5 Semântico, Vanilla JavaScript moderno (ES6/Async-Await), Vanilla CSS flexbox/grid architecture.
- **Segurança de Armazenamento Client-side**: Algoritmo `AES` Criptográfico.
- **Autenticação e Distribuição**: `BaaS Base Firebase Cloud`.

### Bibliotecas de Terceiros e CDN Dependentes
1. **Firebase Compat v10.12.0 (Google)**: Nuvem unificada p/ Auth, DB Firestore e SDK Storage.
2. **Crypto-JS (4.1.1)**: Algoritmo base para criptografia das variáveis temporárias do médico/paciente.
3. **pdf.js (Mozilla - 3.11.174)**: Interpretador nativo de buffer ArrayBuffer para extração de caracteres exatos (nativo PDF).
4. **Tesseract.js (v4/v5 - WebAssembly Worker)**: Sub-processo carregado via thread paralela pra aplicar Reconhecimento Ótico de Caracteres em matrizes onde não há texto extraível direto de PDF (pdfs escaneados/imagens).
5. **EmailJS (@emailjs/browser)**: Micro-serviço SMTP serverless permitindo dispares de template transacionais sem expor autenticadores do cliente no browser.

### Requisitos e Configuração do Ambiente
**Desenvolvimento Prático**: Nenhuma configuração especial de transpilador (Webpack, Vite) é mandatório para debugar as views. Bastará inicializar qualquer servidor estático simplificado que lide com sub-pastas `/Pages/` e `/Scripts/` (Ex: `Live Server vscode`, `Node http-server`, `python -m http.server`).

*Nota de Segurança de Configuração Produtiva:* Hoje o projeto possui a sua infraestrutura declarada estaticamente no `SAFE_CONFIG` no topo do `core.js`. Num cenário produtivo, essas restrições da API (Apikey, projeto) não vazam informações restritas unicamente pois estão blindadas pelas `Security Rules` do Firestore, o que é o procedimento canônico validado pela Google para frameworks client-less sem middlewares.

### Ambiente de Produção (Firebase Hosting) e LGPD
O projeto já está configurado e formatado para instâncias baseadas em **Linux** (Case Sensitive), seguindo os padrões do **Firebase Hosting**.
- **Case Sensitivity:** Os diretórios nativos do projeto como `/Styles/` e `/Scripts/` e seus chamados CSS/JS foram rigorosamente tipados em Maiúscula.
- **Roteamento Primário:** Há um `index.html` na pasta root (raiz) do projeto rodando um dispatcher de tela direta para `/Pages/safe_login.html`, mascarando a estrutura de diretórios para o usuário final.
- **Privacidade e LGPD:** Todos os retornos explícitos em `console.log` usados puramente para depuração laboratorial nos submódulos web (como `carregar-presafe.js` e `safe-form.js`) foram desativados via código, garantindo que nenhum vazamento de strings de exames dos pacientes fique visível via Developer Tools do navegador em redes de terceiros.

---

## 6. Arquitetura do Sistema

### Organização Lógica de Componentes Sensíveis
O sistema quebra componentes funcionais em páginas e em processos (State Pattern não intrusivo):
A captura e transição é toda efetuada preenchendo variáveis na camada `Storage`. 
A ponte comunicativa de `A` (Origem Presafe PDF/Input) com `B` (Visualização Meticulosa Anestésica) acontece encapsulando informações numa bolha com chaves simétricas providas por `SAFE_CONFIG.security.storageSecret` no `core.js`. Toda tela ao iniciar se responsabiliza por ler se a sessão a qual o `Session` diz quem pertence, pode ser descriptografada para acesso, evitando exposição direta do dado de paciente persistido via ferramentas de inspeção do navegador (`DevTools Panel`).

### Diagrama Lógico de Funcionamento

```mermaid
graph TD;
    %% Definições de atores e conexões principais 
    A([Usuário / Médico Anestesiologista]) --> B[safe_login.html];
    B -->|auth.js / signInWith...| C(Firebase Auth Service);
    C -->|Permissão & JWT Token| D[safe_dashboard.html];
    
    D --> E[Carregar Pré-SAFE do Paciente];
    D --> F[Navegar para Formulário Completo / Novo Forms];
    
    E -->|Caso 1: Upload Arquivo (.pdf)| G[pdf.js & Tesseract Web Worker];
    E -->|Caso 2: Inserir Link do Paciente| H[(Firestore NoSQL Collection)];
    
    G --> I[Filtro de Ruídos Visuais / Regex Normalizer];
    H --> I;
    
    I -->|Compressão e Encrypt JSON (AES)| J[(Browser SessionStorage)];
    J --> F;
    
    F -->|Cálculos Inteligentes (riscos anestésicos, laboratoriais automáticos)| K{Médico edita / ajusta / audita dados};
    
    K -->|Emissão | L[Exportação / Firebase Save Endpoint];
```

---

## 7. Melhorias Futuras (Roadmap)

### Sugestões de Otimização e Evolução Estrutural
1. **Migração do Client-side a Framework Componentizado**: O núcleo `safe-form.js` escalou como um documento extenso e poderoso monólito JavaScript. Encapsular a arquitetura num ecossistema reativo como **React, Svelte ou Vue.js**, tornaria a base muito mais fragmentada, isolando partes menores, facilitando testes e mantendo UI e Lógica fortemente entrelaçados em componentes e reduções significativas de tamanho.
2. **Integração Webhook Backend**: Apesar da ótima utilidade funcional transacional do `EmailJS`, estruturar um servidor minimalista, ex: Express e NodeJS na Firebase Cloud Functions, proporcionaria envio ilimitado (Via SDK SMTP), flexibilidade de HTML mais complexos, e isenção da dependência no browser para mandar alertas aos pacientes.

### Novas Funcionalidades para o Workflow Anestésico
- **Painel Analítico Avançado**: Adicionar um ambiente no Dashboard que compile os volumes da clínica do mês logado. (Tabelas e Gráficos de comorbidades mais frequentes ou escores de alto risco emitidos nas avaliações recentes do administrador da conta).
- **Telemedicina / Videoconferência P2P**: Um atalho na seção de preparo do formulário que gerasse um token transiente permitindo ao administrador/médico discutir detalhes sensíveis do pré-SAFE face a face instantaneamente através do browser (ex: WebRTC) durante a triagem inicial do paciente à distância.
- **Multi-Tenancy Organizacional para Hospitais**: Criações de papéis restritivos. Separar médicos num array de "Clinica ABC" para que o ambiente de gestão permita customizar a folha inicial de timbrados, marcas d'agua no laudo final e as predefinições anestésicas conforme contrato de seu próprio hospital corporativo.

### Segurança e Desempenho
- **Restrição Contínua das Regras (Security Rules)**: À medida que escala, fortificar fortemente no Backend (Google Cloud Console e Firebase) o escopo de leitura, para que somente as sub-entidades com perfis autenticados vinculados consigam buscar em sub-coleções de formulários com base na premissa médica real.
- **Processamento Computacional Offshore (OCR Serverless)**: Utilização mandatória do AWS Textract/Google Vision API em background se o arquivo fotográfico inserido possuir mais do que N páginas de escaneamento em extrema baixa qualidade (que atualmente é repassado ao `Worker` interno consumindo CPU do navegador). Alternando as infraestruturas, garantimos o fluxo zero de atrasos nos dispositivos menos potentes.
