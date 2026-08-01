# 🛒 QuantoPaguei

Controle histórico de preços de supermercado a partir do QR Code de notas fiscais (NFC‑e de Minas Gerais). Registre suas compras e, na próxima ida ao mercado, consulte na hora se o preço da gôndola está bom: menor preço já pago (com mercado e data), média histórica e último preço.

## Como usar

1. Publique este repositório no **GitHub Pages** (Settings → Pages → branch principal, pasta raiz). O app é 100% estático — é só o `index.html`.
2. Abra o site no **celular** e, no mercado, use a aba **Ler nota**:
   - Leia o QR Code da nota com a câmera (ou cole a URL/chave de 44 dígitos);
   - O app abre a nota no site da SEFAZ‑MG; selecione tudo, copie e cole de volta no app;
   - Confira os itens extraídos (tudo editável) e salve.
3. Na aba **Consulta**, digite um produto para ver menor preço, média e histórico. Informe o preço da gôndola para saber se está acima ou abaixo da sua média.

> Por que copiar e colar? Um site estático não consegue baixar a página da SEFAZ diretamente (bloqueio de CORS). O fluxo de colar o texto contorna isso sem precisar de servidor.

## Recursos

- Leitura de QR Code pela câmera (html5-qrcode via CDN) com detecção de nota duplicada;
- Parser tolerante do texto do portal da NFC‑e (vários formatos de cópia, vírgula decimal brasileira);
- Cadastro manual de item avulso (sem nota);
- Unificação de nomes: agrupe "PEITO FGO KG" e "FILE PEITO FRANGO RESF" sob um mesmo produto;
- Dados no **IndexedDB** do navegador (funciona offline após o primeiro carregamento);
- Backup completo: exportar/importar arquivo JSON, e opção de apagar tudo.

## Privacidade

Nenhum dado sai do seu aparelho: tudo fica no navegador. Exporte o backup JSON de tempos em tempos para não perder o histórico.
