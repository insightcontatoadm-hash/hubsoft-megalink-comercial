# Painel Comercial Megalink — HubSoft

Dashboard HTML autocontido que reproduz as informações do relatório de referência com a fonte substituída pelo banco HubSoft Megalink.

## Conteúdo preservado

- Evolução de vendas (MoM);
- Indicadores gerais do período: vendas/MRR, baixas e saldo;
- Performance e mix por ISP, Corporativo, Carrier e Governo;
- Negócios contratados, com grupo, tipo, cliente, consultor, plano, valor, data e observação;
- Negócios em andamento do CRM, com etapa, consultor, plano e valor previsto;
- Matriz consultor × segmento;
- Relatório detalhado de baixas;
- Balanço anual;
- Ticket médio por segmento;
- Evolução mensal de churn;
- Top 10 maiores contratos;
- Metodologia, rastreabilidade, grupos excluídos e limites conhecidos.

## Atualização

A página é um **snapshot**: o HTML não abre conexão com o banco. Para atualizar com uma nova leitura read-only:

```text
cd C:\Users\dieys\hubsoft\megalink_comercial
py refresh.py
```

Para visualizar localmente e validar via HTTP:

```text
py -m http.server 8765 --bind 127.0.0.1
```

Abra `http://127.0.0.1:8765/index.html`.

## Contrato analítico

- `VENDA`: `cliente_servico` sem `id_cliente_servico_antigo`, com `data_venda`.
- `UPGRADE`: serviço novo com serviço anterior e `novo_valor > antigo_valor`; a tabela mostra o valor mensal do novo plano, como no relatório de referência.
- `DOWNGRADE`: serviço novo com serviço anterior e `novo_valor < antigo_valor`; a perda mensal é `antigo_valor - novo_valor`.
- `MIGRAÇÃO_NÃO_RESOLVIDA`: serviço anterior ou valor novo ausente; fica fora das métricas para não fabricar eventos.
- `CANCELAMENTO`: `data_cancelamento` preenchida, exceto a perna antiga de uma migração; essa exclusão evita dupla contagem.
- `Baixas totais = cancelamentos + downgrades`.
- `Saldo comercial = vendas/upgrade - baixas`.
- `Ticket médio = soma do valor mensal de vendas/upgrades ÷ quantidade de eventos`.
- `Pipeline`: prospectos ativos, não convertidos, com etapa em `crm_lista.nome`. Registros explicitamente marcados como teste e CRMs de teste/cobrança são excluídos e contabilizados na aba de qualidade.
- `Evolução MoM`: série mensal do ano selecionado; filtros dimensionais permanecem ativos. O recorte mensal/trimestral/semestral não elimina os demais meses da série anual; intervalo personalizado recorta os meses informados.
- Datas: `data_venda` para vendas/upgrades; `data_cancelamento` para cancelamento; data da migração para downgrade; `created_at` para pipeline.

## Segmentos

O de-para é explícito e está visível no painel:

- ISP: grupos iniciados por `ISP`;
- Corporativo: `Corporativo`, `PME` e `MANUTENÇÃO/BACKBONE EMPRESAS`;
- Carrier: `Carrier`, `Rede Neutra`, `Link de Backup` e `SWAP`;
- Governo: grupos iniciados por `Governo`.

Demais grupos — inclusive Varejo, sem grupo, cortesia, teste e status administrativos — não entram nas métricas B2B/B2G principais. Eles aparecem agregados na aba **Metodologia e Qualidade** para evitar exclusão silenciosa.

O CRM não mantém grupo de cliente antes da conversão em `cliente_servico`. Por isso, oportunidades não convertidas aparecem como `Não informado` em segmento/grupo; o plano não é usado como proxy de segmento.

## Segurança e escopo

- Consultas somente leitura pelo utilitário local `hubsoft/db.py`;
- Nenhum token, senha, string de conexão ou parâmetro de acesso é gravado no projeto ou no HTML;
- O HTML contém nomes/valores necessários ao drill-down operacional, mas não inclui credenciais nem dados de conexão;
- A aplicação não usa planilha, Google Apps Script, API externa ou dados simulados.
