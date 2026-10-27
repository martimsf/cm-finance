# CLAUDE.md — C&M Finance

App de finanças pessoais da família (perfis C&M Joint, Martim, Carlota). Publicada via GitHub Pages a partir do `main`.

## Regras críticas

1. **Plano primeiro.** Não editar, commitar ou fazer merge sem o "go" do Martim.
2. **Ler o código atual** (`main`) antes de propor alterações. Não assumir pelo histórico.
3. **Stack:** um único `index.html`, JS simples, Supabase REST via `sbFetch`. Sem frameworks nem build.
4. **Estilo:** igual ao código à volta (one-liners minificados, estilos inline, variáveis CSS existentes).
5. **`main` é produção** (repo público + Pages). Trabalhar em branches `claude/*`; merge só com go.
6. **Dados:** análise só com SQL de leitura. Escrever dados ou mudar schema exige go.
7. **Testar antes de entregar:** verificação de sintaxe + Playwright com dados de teste (esconder `#gate`, definir `DB`/`S`/`consolidatedNets`), screenshots a 1100px e 390px.
8. **Três perfis separados** (tabelas `cm_*`, `martim_*`, `carlota_*`). O que interessa são as entradas e saídas de cada conta. Não mexer na lógica de transferências entre perfis sem go.
9. **Categorização:** confiança ≥90% é automática. Medir no histórico real antes de afirmar ganhos.
10. **Respostas curtas por defeito**, em PT-PT.

## Reembolsos e claims (modelo atual — seguir o código)

- **Claims de saúde (Sanitas):** despesas *Doctor*/*Pharmacy* em qualquer perfil; espera-se 80% (farmácia 50%). Pagamentos = receita *Insurance Reimbursements* com "Sanitas". Ligação automática por valor exato (≤240 dias), Clínica Horas (pagamento parcial) ou soma de 2 claims do mesmo perfil (≤90 dias). Bolos de 3+ ficam em "unmatched" para ligar à mão. Estado em `health_claims`, ligações em `claim_links`, data de início em `cm_settings`.
- **Reembolsos pessoais** (amigos, C&M ↔ Martim/Carlota): sem ligação. Marcar com flag; o dinheiro devolvido por amigos entra como receita (*Income › Reimbursements*), salvo se lhe for dada a categoria da despesa (`isContra`, abate). Acertos C&M → Martim/Carlota: no pessoal, saída e entrada ficam ambas em *Finance › Joint Expenses* (a entrada abate, não infla a receita); no C&M a transferência leva a categoria do gasto real.
