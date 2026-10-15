# Nuvio Anime Sources

Plugin modular para o contrato de provedores documentado em [yoruix/nuvio-providers](https://github.com/yoruix/nuvio-providers). Inclui adaptadores isolados para AnimesDrive e AnimesOnline e devolve somente streams diretos `m3u8`/`mp4` detectados. Não hospeda nem contorna mecanismos de acesso dos sites.

## Estado real da validação

- Build do provider pelo `build.js` oficial do repositório de referência: compilado com sucesso.
- O site AnimesOnline respondeu HTTP 200 na página inicial e exibiu busca WordPress `?s=...&post_type=anime`. Foi observado um anime com URLs de episódios. Não foi demonstrado que a página do episódio contenha um stream direto FHD compatível com o Nuvio; o HTML indica um player incorporado. O adaptador deliberadamente não apresenta um iframe como stream direto.
- AnimesDrive respondeu HTTP 403 na requisição de inspeção. Portanto, busca, identificação da opção FHD e reprodução nessa fonte **não foram validadas**.
- A assinatura oficial recebe somente `tmdbId`, `mediaType`, `season`, `episode`; não recebe título nem títulos alternativos. Para converter o ID em nome sem inventar uma API do Nuvio, este projeto inclui a função `/api/titles` do Vercel que consulta TMDB server-side. Sua chave fica como variável de ambiente.
- Não foi possível executar o Plugin Tester no aplicativo Nuvio neste ambiente. O teste final em Hermes/React Native permanece necessário.
- Assim, o repositório é uma base compilável/instalável, mas não deve ser descrito como reprodução funcional confirmada. Se nenhuma URL direta válida estiver presente, retorna `[]` sem fingir sucesso.

## Estrutura

- `src/nuvio-anime-sources/`: plugin de entrada e módulos compartilhados.
- `src/nuvio-anime-sources/animesdrive/`: adaptador independente, exigindo marcador FHD próximo do link direto.
- `src/nuvio-anime-sources/animesonline/`: pesquisa por anime, escolha do episódio e aceitação de links diretos de mídia apenas.
- `providers/nuvio-anime-sources.js`: bundle que o Nuvio carrega.
- `manifest.json`: manifesto oficial em formato de lista, com filename para o bundle.
- `api/titles.js`: resolução de TMDB ID → títulos oficiais via função Vercel.
- `tests/`: testes de contrato, comportamento degradado e sondas reais opcionais.

## Publicação no Github e Vercel

1. Extraia o ZIP e envie todos os arquivos para um repositório GitHub próprio, preservando a estrutura e o `manifest.json` na raiz.
2. No Vercel, importe esse repositório como projeto. Use o diretório raiz padrão; não é necessário build command para servir arquivos estáticos. A função `/api/titles` é implantada automaticamente.
3. Crie uma chave de API TMDB na sua conta TMDB e adicione no Vercel, em **Settings → Environment Variables**, a variável `TMDB_API_KEY`. Não publique essa chave em arquivo ou no GitHub. Faça redeploy após definir a variável.
4. Copie a URL de implantação, por exemplo `https://seu-projeto.vercel.app`.
5. Em `src/nuvio-anime-sources/shared/config.js`, preencha `titleServiceUrl` com `https://seu-projeto.vercel.app/api/titles`. Reconstrua o bundle: `npm ci && npm run build:provider -- nuvio-anime-sources` (ou `node build.js nuvio-anime-sources`). Envie o bundle atualizado ao Github e aguarde novo deploy.
6. A URL do manifesto no Vercel será `https://seu-projeto.vercel.app/manifest.json`. No Nuvio, abra **Settings → Plugins**, adicione essa URL, atualize a lista e habilite o plugin.
7. Para desenvolvimento local, `npm ci && npm start`; no Nuvio use **Settings → Developer → Plugin Tester** com a URL local de `manifest.json`. O guia oficial ressalta que alguns plugins exigem a build de desenvolvimento do Nuvio para testes.

**Nota:** Vercel não corrige o HTTP 403 do AnimesDrive nem converte players incorporados em mídia direta. Os provedores e seus formatos podem mudar. O suporte a FHD só é rotulado quando o marcador FHD estiver junto da URL que o adaptador reconhece como stream direto.

## Desenvolvimento e testes

```sh
npm ci
npm run build:provider -- nuvio-anime-sources
npm test
npm run test:sources
```

`npm test` verifica o manifesto e o contrato do plugin, incluindo o retorno seguro `[]` quando o resolvedor não está configurado. `npm run test:sources` faz sondas HTTP separadas das duas fontes; falhas de rede e bloqueios são reportados como limitações, não como streams encontrados. O teste automatizado com Node não substitui o Plugin Tester/Hermes.

## Contrato Nuvio

`module.exports = { getStreams }`, com parâmetros documentados `getStreams(tmdbId, mediaType, season, episode)`. Cada item válido tem `name`, `title`, `url`, `quality` e `headers` opcionais; URL deve ser mídia direta para o player compatível. Não foram adicionados exports/funções de busca ao contrato do app.

## Extensão

Para adicionar uma fonte, crie outro módulo em `src/nuvio-anime-sources/<fonte>/`, exporte `findStreams(titles, season, episode)`, importe no ponto de entrada e atualize `manifest.json` se necessário. Preserve os limites e retorne `[]` em timeout, indisponibilidade ou ausência de stream direto.

## Referências

- Repositório e guia obrigatórios: https://github.com/yoruix/nuvio-providers (README e `DOCUMENTATION.md`).
- AnimesDrive: https://animesdrive.cloud/
- AnimesOnline: https://animesonlines.net/
- TMDB API: https://developer.themoviedb.org/reference/tv-series-details

Licença herdada do repositório de referência: GPL-3.0. O usuário é responsável por verificar permissões, termos e leis aplicáveis às fontes usadas.
