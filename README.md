# Desafio DevSecOps — Gerenciador de Tarefas

## Sobre o Projeto
Este repositório faz parte do desafio prático do módulo de DevSecOps da ADA Tech.
Você receberá este projeto com vulnerabilidades propositais e uma pipeline incompleta.
Seu objetivo é **implementar a pipeline de segurança** e **corrigir as vulnerabilidades**.

## Estado atual
A pipeline está **incompleta**. Os steps de segurança precisam ser implementados por você.

## Sua missão
1. Implementar os steps de segurança no `pipeline.yml`
2. Fazer a pipeline **quebrar** ao detectar os problemas
3. Corrigir as vulnerabilidades encontradas
4. Fazer a pipeline **passar** com tudo verde ✅
5. Documentar o funcionamento da pipeline neste README

## O que implementar
- [ ] Secrets Scanning com **Gitleaks**
- [ ] SAST com **Semgrep**
- [ ] SCA com **Grype**
- [ ] Assinatura do artefato com cosign
- [ ] Deploy com **GitHub Pages**

## Como a pipeline funciona

A pipeline roda automaticamente a cada push na branch `main`. Ela funciona como uma esteira
com "portões" (gates) de segurança: se algum portão encontrar um problema, a esteira para e
o código **não** chega à produção.

### Job 1 — Build, Segurança e Assinatura

1. **📥 Checkout do Código** — baixa o código do repositório, com todo o histórico de commits,
   para que as ferramentas possam analisá-lo.
2. **🔐 Inserir Secrets no JavaScript** — lugar reservado para injetar valores sensíveis (vindos
   dos GitHub Secrets) só na hora do build, em vez de deixá-los escritos no código.
   Por enquanto não insere nada.
3. **⚙️ Build** — confere se os arquivos do site estão presentes.
4. **🔑 Secrets Scanning (Gitleaks)** — procura senhas, tokens e chaves de API esquecidos no
   código. *Por que importa:* um segredo publicado no GitHub pode ser usado por qualquer pessoa.
   Aqui ele encontrou uma `API_KEY` e uma `DB_PASSWORD` no `src/script.js`, que foram removidas.
5. **🔍 SAST (Semgrep)** — analisa o código-fonte em busca de padrões inseguros, sem executá-lo.
   *Por que importa:* pega falhas antes de irem para o ar. Aqui ele apontou um XSS (`innerHTML`
   com o texto digitado pelo usuário) e um `eval` com entrada do usuário, que permitiriam executar
   código malicioso na página. Ambos foram trocados por alternativas seguras (`innerText` e um
   `console.log` direto).
6. **📦 SCA (Grype)** — verifica se as bibliotecas usadas pelo projeto têm vulnerabilidades
   conhecidas (CVEs). *Por que importa:* a maior parte do código de um projeto vem de terceiros.
   Aqui ele encontrou `lodash`, `express` e `axios` em versões antigas (incluindo uma falha
   crítica no lodash), que foram atualizadas. Também apontou a action `actions/download-artifact@v4`
   da própria pipeline, atualizada para `v4.1.3`.
7. **📦 Empacotar artefato do site** — junta os arquivos do site em um único pacote (`artifact.tar`).
8. **🔢 Hash do artefato** — calcula a "impressão digital" (SHA-256) do pacote, para detectar
   qualquer alteração depois.
9. **✍️ Instalar Cosign / Assinar artefato** — etapa de assinatura digital do pacote, que prova
   quem o gerou. *Atenção:* hoje a assinatura é apenas simulada (mock) e não tem validade real.
10. **⬆️ Upload do artefato assinado** — guarda o pacote para o próximo job.

### Job 2 — Verificação e Deploy

Só roda se o Job 1 passar inteiro.

1. **⬇️ Baixar o artefato** — pega o pacote gerado no Job 1.
2. **🔎 Verificar assinatura** — conferiria se o pacote não foi alterado no caminho. Ainda está
   desativado, porque a assinatura do Job 1 é simulada.
3. **📂 Extrair site / 🌐 Configurar Pages / 📤 Upload / 🚀 Deploy** — publica o site no
   GitHub Pages.

## URL de Produção
https://clarajfc.github.io/projeto-devsecop-desafio/
