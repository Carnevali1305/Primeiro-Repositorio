# Fluxo n8n – Assistente de Status de Processo (CNJ)

Este fluxo foi ajustado para garantir uma conversa cordial com o cliente, validar os dados antes da consulta e fazer buscas dirigidas ou amplas nos endpoints do DataJud (CNJ). Ele também envia feedback quando nenhum processo é encontrado.

## Pontos principais
- **Saudação imediata** ao receber "oi", "olá" ou `/start`, explicando o que será pedido.
- **Coleta guiada**: nome completo → data de nascimento → advogado(a) → número do processo (validação CNJ de 20 dígitos).
- **Mensagens de correção** quando um dado chega em formato incorreto.
- **Busca dirigida** quando o código do tribunal é identificado no número CNJ; caso não encontre, o fluxo cai para **busca ampla** em todos os tribunais.
- **Resposta cordial e simplificada** usando OpenAI, com encerramento convidando o cliente a voltar.
- **Aviso de ausência de resultados** quando nada é encontrado em nenhuma busca.
- **Limpeza do estado no Redis** após a resposta final.

## Como usar
1. Importe o arquivo `n8n_flow_cnj_atendimento.json` no n8n.
2. Configure as credenciais:
   - Telegram (Trigger e Envio de mensagens).
   - Redis.
   - OpenAI.
3. A API Key do DataJud já está incluída no cabeçalho `Authorization` dos nós HTTP. Ajuste se necessário.
4. Publique o webhook do Telegram Trigger após configurar o bot.

## Endpoints consultados
O fluxo tenta primeiro um endpoint direcionado (quando o código do tribunal é reconhecido no número CNJ). Se falhar, ele varre todos os endpoints públicos listados na documentação do DataJud, usando o mesmo payload:

```
POST https://api-publica.datajud.cnj.jus.br/api_publica_<tribunal>/_search
Body: { "numeroProcesso": "<numero CNJ>" }
Headers: Authorization: APIKey ...
         Content-Type: application/json
```

## Conversa resumida
1. Saudação + pedido de nome completo.
2. Solicitação de data de nascimento (DD/MM/AAAA).
3. Nome do advogado(a).
4. Número do processo (20 dígitos, com ou sem formatação).
5. Mensagem de “aguarde” durante a busca.
6. Resposta cordial com resumo dos últimos movimentos ou aviso de não encontrado.
