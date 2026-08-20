[README.md](https://github.com/user-attachments/files/31271417/README.md)
# Controle de Inadimplência — Perinity

Painel de arquivo único (`index.html`) para a carteira de contas a receber vencida: aging, concentração, faixas de valor, rating de clientes, fila de cobrança e série histórica.

Senha atual: `Perinity@2026!` — **leia a seção de segurança antes de publicar.**

---

## Publicar no GitHub Pages

1. Crie o repositório **como privado**.
2. Suba apenas o `index.html`.
3. `Settings → Pages → Source: Deploy from a branch → main / (root)`.
4. A URL sai em `https://<usuario>.github.io/<repo>/` em ~1 minuto.

O arquivo não tem build. Puxa duas coisas de CDN (SheetJS para ler/gerar Excel, Google Fonts para tipografia) — funciona sem elas, só perde a leitura de planilha e a fonte.

## Uso no dia a dia

1. Abrir a URL, entrar com a senha.
2. Arrastar a exportação do Omie (*Base Contas a Receber Contas por Período*) na área de carga.
3. Conferir a **data de referência** — o aging inteiro depende dela. Padrão: hoje.
4. **Gravar snapshot.** É isso que constrói o rating ao longo do tempo.
5. Exportar o Excel quando precisar do número fora do painel.

Trocar a base = arrastar o novo arquivo. Nada mais muda, desde que o layout da exportação continue igual.

## Filtro de categoria e canais de cobrança

O filtro de **categoria** aceita múltipla seleção: marque quantas quiser, com atalhos *Todas / Só amigável / Só judicial / Nenhuma*. Serve para isolar um tipo de serviço ou o estágio de cobrança.

O painel também deriva um **canal de cobrança** de cada título — amigável/interna vs. judicial/extrajudicial — a partir da categoria "Cobrança judicial/Extrajudicial". O card *Canais de cobrança* (aba Carteira) compara os dois lado a lado e, quando há dois snapshots, mostra a **taxa de recuperação de cada canal**. Três coisas a ter em mente:

- **A categoria judicial substitui a de serviço.** No Omie, quando um título é escalado para o jurídico, ele perde a categoria original (SaaS, Consultoria…) e passa a "Cobrança judicial/Extrajudicial". O gráfico *Saldo por categoria* deixa de refletir o mix de receita — para o mix, filtre só o canal amigável.
- **Não compare os totais dos dois canais direto.** O judicial herda o crônico (atraso médio muito maior), então qualquer taxa dele parece pior. Eficiência real = recuperação dentro de faixas de atraso equivalentes.
- **Recuperação ≠ pagamento.** "Resolvido" é título que sumiu entre snapshots — pode ser pagamento, acordo, cancelamento ou baixa.

## Trocar a senha

A verificação compara o SHA-256 da senha digitada com a constante `PW_HASH` no topo do `<script>`. Para trocar:

```bash
echo -n 'NovaSenha' | shasum -a 256
```

Cole o hash em `const PW_HASH = '...'`.

## Segurança — o que esta senha faz e o que não faz

**Não faz:** proteger dados. A verificação roda no navegador. Qualquer pessoa com o link abre o código-fonte, remove a checagem e entra. Um hash SHA-256 de senha curta cai em segundos num ataque de dicionário offline. Isso é um aviso de uso restrito, não um controle de acesso — não existe autenticação real em página estática sem servidor.

**A decisão que importa não é a senha, é onde a base mora.** Por padrão o painel lê a planilha do seu computador e não guarda nada do lado do servidor. Enquanto for assim, o repositório não contém dado de cliente e o risco é baixo mesmo se vazar o link.

O botão *"Carregar dados.xlsx do repositório"* existe para quando alguém quiser versionar a planilha. **Não use em repositório público.** Ali estão CNPJ, valor devido e histórico de atraso de clientes reais — Caixa, ONS, SESI, Energisa. Repositório público + GitHub Pages = base pública, senha irrelevante. GitHub Pages em repositório privado só existe nos planos Enterprise; nos demais, publicar Pages torna o site acessível a qualquer um com a URL, mesmo com o repositório privado.

Ordem de preferência:

| Opção | Base exposta? | Esforço |
|---|---|---|
| Repositório privado + carga manual do arquivo | não | nenhum |
| Repositório público + carga manual | não (só o código) | nenhum |
| Qualquer repositório + `dados.xlsx` versionado | **sim** | — |
| Hospedagem interna com SSO | não | alto |

## Histórico

Fica no `localStorage` deste navegador. Some ao limpar cache, não acompanha outro computador e não é compartilhado com a equipe. **Exporte o JSON a cada snapshot** (aba Histórico → *Exportar histórico*) e guarde no Drive. Para reconstruir noutro computador: *Importar histórico*.

Um jeito de tornar isso robusto sem servidor: versionar o `historico.json` no repositório e importar quando trocar de máquina. O JSON tem nome de cliente e saldo — mesma regra de repositório privado se aplica.

## Rating — resumo

Score 0–100, cinco componentes com renormalização de pesos quando falta dado:

| Componente | Peso | Precisa de |
|---|---|---|
| Severidade do atraso | 35 | base atual |
| Acúmulo de títulos | 15 | base atual |
| Cronicidade | 20 | ≥ 2 snapshots |
| Tendência do saldo | 12 | ≥ 2 snapshots |
| Cura | 18 | ≥ 2 snapshots |

Faixas: A ≥ 80 · B 65–79 · C 50–64 · D 35–49 · E < 35.

Com um snapshot só, o score sai de severidade + acúmulo e vem marcado `prov`. A aba **Metodologia** traz o detalhe e as limitações. Duas que valem repetir:

- **Não é probabilidade de default.** É índice ordinal construído com julgamento, sem regressão contra perda observada. Serve para ordenar esforço de cobrança, não para decidir provisão ou rescisão sozinho.
- **"A" não é bom pagador.** A base só tem títulos vencidos. "A" é o atraso mais leve entre os atrasados.
