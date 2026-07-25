# Controle de Validade — Guia de configuração

App web (PWA) para leitura de código de barras, cadastro de validade e
exportação para o Google Drive. Roda 100% pelo celular, sem PC.

## Estrutura das tabelas (dentro do app, IndexedDB)

**`leituras`** — registros lidos no celular
- `id` (automático)
- `ean`
- `descricao`
- `validade` (obrigatório)
- `lote`
- `data_leitura`
- `foto` (imagem em base64)

Regra: no máximo **5 registros por EAN**. Ao salvar um 6º, o mais antigo
é apagado automaticamente. Se ler o mesmo EAN + mesmo lote + mesma
validade de novo, o app avisa e não duplica.

**`ean_base`** — base geral de consulta (upload de CSV/XLSX)
- `cod_ean`
- `descricao`

## Passo 1 — Criar o Client ID do Google (necessário para o login no Drive)

1. Acesse https://console.cloud.google.com/ (pelo celular mesmo, funciona no navegador)
2. Crie um projeto novo (ou use um existente)
3. Vá em **APIs e Serviços → Biblioteca**, procure **Google Drive API** e clique em **Ativar**
4. Vá em **APIs e Serviços → Tela de consentimento OAuth**
   - Tipo: Externo
   - Preencha nome do app, e-mail de suporte, e-mail do desenvolvedor
   - Em "Escopos", não precisa adicionar nada manualmente
   - Em "Usuários de teste", adicione seu próprio e-mail Google (enquanto o app não for publicado)
5. Vá em **APIs e Serviços → Credenciais → Criar credenciais → ID do cliente OAuth**
   - Tipo de aplicativo: **Aplicativo da Web**
   - Em "Origens JavaScript autorizadas", adicione a URL onde o app vai rodar
     (ex: `https://seuprojeto.replit.app` ou `https://seuusuario.github.io`)
   - Salve e copie o **Client ID** gerado (termina com `.apps.googleusercontent.com`)
6. Abra o arquivo `index.html`, procure a linha:
   ```js
   const GOOGLE_CLIENT_ID = "SEU_CLIENT_ID_AQUI.apps.googleusercontent.com";
   ```
   e substitua pelo seu Client ID real.

> Sem esse passo, tudo funciona normalmente (leitura, cadastro, consulta,
> importação de base) — só o login/exportação para o Drive fica bloqueado.

## Passo 2 — Publicar o app (pelo celular, sem PC)

**Opção recomendada: Replit**
1. Crie conta em https://replit.com pelo navegador do celular
2. Crie um novo "Repl" em branco (tipo HTML/estático)
3. Suba os arquivos `index.html`, `manifest.json` e `sw.js` (use o botão de
   upload de arquivos do Replit)
4. Clique em "Run" — o Replit gera uma URL pública (ex: `https://seuprojeto.usuario.repl.co`)
5. Volte no passo 1 e cadastre essa URL como origem autorizada no Google Cloud

**Alternativa: GitHub Pages**
1. Crie um repositório no GitHub (dá pra fazer pelo app do GitHub ou pelo navegador)
2. Suba os 3 arquivos
3. Ative GitHub Pages em Settings → Pages
4. Use a URL gerada da mesma forma no passo 1

## Passo 3 — Instalar no celular

Abra a URL publicada no Chrome do Android (ou Safari no iPhone) e escolha
**"Adicionar à tela inicial"**. O app passa a abrir como um aplicativo comum.

> Nota: para o prompt de instalação funcionar 100%, adicione dois ícones
> (`icon-192.png` e `icon-512.png`, quadrados) na mesma pasta — pode ser
> qualquer imagem simples do seu produto/logo. Sem eles o app ainda funciona,
> só o ícone da tela inicial fica genérico.

## Limitações importantes

- **Leitura automática de código de barras** funciona bem no Chrome
  Android (usa a API `BarcodeDetector`). No iPhone/Safari essa API não
  existe — nesse caso, use sempre a opção "Digitar EAN manualmente".
- As fotos ficam guardadas **dentro do app** (IndexedDB do navegador), não
  em uma pasta do sistema operacional — isso é uma limitação de qualquer
  app que roda no navegador. Elas só viram arquivos de verdade, com nome
  `EAN_DATA.jpg`, no momento da exportação para o Drive.
- Se limpar os dados do navegador/cache do celular, os registros locais
  que ainda não foram exportados se perdem — exporte com regularidade.
