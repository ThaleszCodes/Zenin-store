# Zenin Store

Loja digital single-seller com frontend React/Vinext, banco Postgres no Neon e cobrança Pix PaggOne/PaggPay.

## Acesso

- `/`: vitrine. O proprietário vê também rascunhos identificados como prévia.
- `/seller`: produtos, pedidos e configurações. Autorização pelo e-mail do proprietário no servidor.
- `/account`: pedidos e acesso aos produtos comprados, vinculados à identidade autenticada.
- Login preserva a autenticação gerenciada do Sites. Nenhuma senha de administrador padrão.

## Configuração

Secrets de runtime: `DATABASE_URL`, `ENCRYPTION_KEY` (hexadecimal de 32 bytes), `OWNER_EMAIL`. Não persistir em código. Configurar pelo serviço de hospedagem.

No painel do vendedor, salvar ambiente, API key e opcionalmente segredo do webhook da PaggOne. Credenciais ficam criptografadas com AES-256-GCM no banco. API key nunca retorna na resposta do painel. Testar conexão requer `balance.read`; cobrar requer `pix.create`, consultar requer `pix.read` conforme permissões da conta.

A versão inicial tem acesso privado e 50 rascunhos. Antes de abrir vendas: definir suporte e identificação comercial, revisar termos, configurar gateway de produção e publicar produtos com entrega e licença verificadas. Documentos de entrega não estão copiados neste repositório.

## Pagamentos

Contrato consultado: https://docs.paggone.com/ e OpenAPI https://docs.paggpay.com/openapi.json em 31/10/2026.

Produção: `https://api.paggone.com/v1`; sandbox: `https://sandbox-api.paggpay.com/v1`. `POST /pix`, valores inteiros em centavos e referência do pedido. Sem taxa adicional ao comprador. Sem cobrança real na ausência de API key.

O checkout recalcula valores a partir do banco e faz snapshot de preço, licença e entrega. Uma chave idempotente por carrinho impede reenvio acidental. Falha incerta de criação deixa pedido em processamento, sem recriar cobrança automaticamente.

Consulta autenticada `GET /pix/{id}` confere transação, referência e valor antes de liberar entrega. Navegador consulta a cada 8 segundos enquanto o pedido está aberto; também há verificação manual. O backend impõe intervalo mínimo de consulta e limite de geração de pedidos por conta.

Webhook: `/api/webhooks/paggone`. Requer URL pública, segredo igual na PaggOne e assinatura HMAC-SHA256 `timestamp.raw_body` com tolerância de 5 minutos. Eventos têm identificador único; confirmação financeira vem da consulta ao gateway, não de status enviado pelo cliente. Configurar a assinatura e o endpoint no painel PaggOne quando a loja tiver URL pública. Esta entrega privada não recebe callbacks de terceiros; polling continua funcional com uma chave válida. Pedidos de teste não entram na receita de produção.

## Catálogo

`db/catalog.json`: as 50 posições planejadas com origem e escopo. Todos iniciam como rascunho. `db/001-initial.sql` cria schema; `db/002-seed.sql` importa uma vez sem sobrescrever edições existentes. O fracionamento e os kits são propostas de montagem, não afirmações de autorização comercial.

Não há dados falsos de vendas, depoimentos ou pagamento confirmado. As capas automáticas são peças tipográficas da interface, não prévias dos arquivos fornecidos.

## Desenvolvimento

Use a instalação e os scripts fornecidos pelo Sites. `node node_modules/typescript/bin/tsc --noEmit` e build de produção. Não inclui `.env`, tokens ou credenciais do banco.

## Catálogo conferido em 31/10/2026

16 produtos PLR publicados, com pasta de entrega contendo arquivo e licença original. Licenças PDF comparadas byte a byte com o documento inspecionado; licenças PNG conferidas por OCR. Conteúdo amostrado para verificar formato e tema. Os outros 42 cadastros permanecem rascunhos. Nenhuma permissão do Drive foi alterada; o acesso das pastas por um comprador externo ainda precisa de validação. Pagamentos reais dependem da configuração PaggOne.

`db/catalog-ready.json` registra os 16 produtos reais. A prévia de desenvolvimento usa esse arquivo, sem executar alterações. `db/003-ready-products.sql` reproduz o cadastro por upsert, preservando os rascunhos. As capas em SVG são representações editoriais; o formato exato está na descrição.

## Substituição pela seleção original

A migração 004 arquiva os cadastros anteriores, preservando referências de pedidos, e registra os 50 itens originais com IDs próprios. O arquivo ativo para a prévia é `db/catalog-active.json`. O campo `display_in_catalog` permite exibir uma ficha em preparação, sem habilitar checkout. `published` continua exigindo licença e entrega válidas. A lista administrativa oculta arquivados. O cadastro original de 38 é uma proposta de economia no casamento; o material examinado anteriormente era sobre relacionamento e não foi reaproveitado como entrega.
