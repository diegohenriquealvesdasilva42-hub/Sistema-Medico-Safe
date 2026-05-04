# 🏥 SAFE v11 - Sistema de Avaliação em Anestesiologia e Formulários Eletrônicos

O **SAFE v11** é uma solução avançada de software desenvolvida para digitalizar, otimizar e unificar o processo de Avaliação Pré-Anestésica (APA). O sistema permite que médicos anestesiologistas centralizem a coleta de dados e realizem a leitura automatizada de laudos complexos, transformando processos manuais em um fluxo digital inteligente e seguro.

> **📢 Nota de Repositório:** Este repositório atua estritamente como uma **Vitrine Técnica (Showcase)**. Por motivos de propriedade intelectual e sigilo comercial da **DHTECH PROJETOS**, o código-fonte original é mantido em um repositório privado. Este espaço é dedicado à exposição da arquitetura, documentação e competências técnicas aplicadas no projeto.

---

## 🎯 O Problema e a Solução

A coleta de históricos médicos pré-operatórios costuma ser lenta e suscetível a erros de transcrição manual. O SAFE v11 resolve esses gargalos através de:

* **Coleta Digital Antecipada (Módulo Pré-SAFE):** O paciente preenche seus dados remotamente, alimentando o sistema antes mesmo do encontro presencial.
* **Extração Inteligente de Dados (OCR):** Utiliza reconhecimento ótico de caracteres para ler laudos em PDF ou imagem, preenchendo automaticamente valores laboratoriais (Plaquetas, Glicose, Creatinina, etc.) e identificando comorbidades via processamento de texto.
* **Apoio à Decisão Médica:** Centraliza calculadoras de risco (ARISCAT, etc.) e regras estritas da medicina clínica para consolidar a evolução anestésica.

## 🛠️ Arquitetura e Stack Técnica

O projeto utiliza uma filosofia **Serverless Front-End Oriented**, onde o processamento pesado de documentos ocorre diretamente no navegador do usuário, garantindo privacidade e baixo custo de infraestrutura.

* **Linguagens Core:** HTML5 Semântico, CSS3 (Flexbox/Grid) e JavaScript Vanilla Moderno (ES6/Async-Await).
* **BaaS (Backend as a Service):** Firebase (Auth, Firestore NoSQL e Hosting).
* **Processamento de Documentos:** `pdf.js` (Mozilla) para extração nativa e `Tesseract.js` (WebAssembly Worker) para OCR em documentos escaneados.
* **Segurança e Criptografia:** Utilização do algoritmo **AES (Crypto-JS)** para proteger variáveis sensíveis no `sessionStorage` e conformidade com a **LGPD** através da desativação de logs sensíveis em ambiente de produção.
* **Comunicação:** Integração com **EmailJS** para disparos transacionais sem exposição de credenciais SMTP no client-side.

## 📸 Demonstração da Interface

*Imagens disponiveis na pasta Imagens*


## 📂 Documentação Disponível

Para uma análise profunda da engenharia por trás do sistema, consulte:
* [`DocumentacaoTecnica.md`](DocumentacaoTecnica.md): Detalhamento de módulos, diagramas lógicos (`Mermaid`) e fluxos de dados.

---

## 🚀 Sobre o Desenvolvedor

Projeto idealizado e desenvolvido por **Diego Henrique**, graduando em Engenharia de Computação pela PUC Minas e fundador da **DHTECH PROJETOS**. Focado no desenvolvimento de sistemas robustos, automação IoT e infraestrutura de software.

**Contatos:**
* **LinkedIn:** [www.linkedin.com/in/diegohenrique-dtech]
* **Empresa:** DHTECH PROJETOS
