# Manchester Funerals — holding page

Página única, em <https://lpduarte.github.io/ManchesterFunerals/> enquanto
espera aprovação do cliente. Sem build, sem dependências: HTML e CSS num
ficheiro, assets estáticos ao lado.

O destino é `manchesterfunerals.co.uk`, mas **esse domínio está ocupado** por
uma página "Coming Soon" do Squarespace. Esta há-de substituí-la.

```
index.html              markup + CSS (inline, é uma página só)
favicon.svg             símbolo branco sobre círculo #424642
assets/img/moor-*.webp  fundo, 3 larguras + variante portrait
assets/img/og.jpg       imagem de partilha, 1200x630
assets/img/             logo, símbolo, ícones
```

## Quando o domínio migrar

Três coisas mudam ao mesmo tempo, e esquecer qualquer uma delas parte alguma
coisa em silêncio:

1. **`CNAME`** no repo com `manchesterfunerals.co.uk`, e o DNS a apontar para
   o GitHub Pages — o que implica tirar do ar a página do Squarespace.
2. **`og:url` e `og:image`** no `index.html`, que hoje apontam para o
   `lpduarte.github.io`. Se ficarem, o preview de quem partilha o link vai
   buscar a imagem ao endereço antigo.
3. **O domínio no kit do Adobe Fonts** (ver "Marca"). Sem isso a Museo não
   carrega, e falha sem dar erro.

### A imagem de partilha

`assets/img/og.jpg` é um screenshot da própria página a 1200×630, com o
crédito escondido — não é uma composição à parte, por isso não pode divergir
do que o site mostra. Refaz-se abrindo a página nessa medida e escondendo o
`.credit`.

É **JPEG de propósito**: vários scrapers, o do WhatsApp incluído, não lêem
WebP em `og:image`.

## Marca

| | |
|---|---|
| Cor principal | `#424642` |
| Tipografia | Museo Sans Rounded (Adobe Fonts, kit `glf6otm`) |
| Logo | fornecido em SVG a `#424642`, convertido para `#FFFFFF` |

### Adobe Fonts — domínios

O kit só serve as fonts nos domínios registados nas suas definições. **Cada
domínio novo tem de ser adicionado ao kit**, senão a Museo não carrega e a
página cai no arredondado do sistema:

- `localhost` (desenvolvimento)
- `lpduarte.github.io` (pré-visualização no GitHub Pages)
- `manchesterfunerals.co.uk` (produção)

O kit vem com `font-display: auto`, o que no Chrome significa um instante de
texto invisível. Muda-se nas definições do kit, não aqui no código.

## Fundo

Uma fotografia de Saddleworth Moor, com o Dovestone Reservoir ao centro.

Até 2026-09-05 o fundo foi um vídeo, removido porque mostrava um vale que não
é o noroeste de Inglaterra — campos em faixas, casas dispersas, encostas
florestadas até ao cume: Europa Central, provavelmente os Beskids. Nada disso
se lia debaixo do véu, mas o ficheiro estava errado para a marca. Chegou a
haver a intenção de o substituir por um vídeo da região; foi abandonada. Se
alguma vez se retomar, o `bg.mp4` e o pipeline que o fazia estão no commit
`593d341`.

### A fotografia

De Roger Cornfoot, 2017, SE0206 — a 3 km de Diggle, Oldham.
[Geograph 5485216](https://www.geograph.org.uk/photo/5485216), **CC BY-SA 2.0**.

A secção 4(c) obriga a crédito visível com quatro coisas e mais nada: nome do
autor, título da obra, ligação à obra e uma nota que identifique o uso na
derivada ("cropped and desaturated"). Mais a ligação à licença, por 4(a). É o
que faz o `.credit` no rodapé.

O ShareAlike **não** se declara na página — é uma obrigação sobre como se
licencia a derivada, não sobre o que se escreve ao leitor. Não pôr lá texto
sobre isso não é incumprimento.

**Se o fundo mudar, o `.credit` sai com ele.**

O original tem 1024×768. Foi ampliado 4× no Photoshop (Camera Raw → Enhance →
Super Resolution), o que reconstruiu detalhe a sério: a energia espectral acima
de meio Nyquist passa de 0,01% (lanczos) para 14,1%. Por isso se servem 4096px
ao ecrã grande em vez de parar nos 2560 — cada degrau abaixo de 4096 corta mais
de metade do detalhe visível.

## Tratamento visual

O fundo leva sempre duas camadas, definidas em `:root` no `index.html`:

- véu `#424642` a 68% — dá o tom da marca e desatura
- vinheta radial (30% no centro → 68% nos bordos) — puxa o olho para o logo

A desaturação do vídeo (60%) está **gravada no ficheiro**, não em CSS. Um
`filter: saturate()` custaria GPU em cada frame; e menos crominância também
comprime melhor.

## Regenerar os assets

Partem dos originais, que não estão no repo.

### Fundo em fotografia

Do PNG de 4096×3072 saído do Photoshop. O corte 16:9 é feito **pelo topo**
(`y=0`): é o que deixa mais céu, e põe o horizonte a 38,7% da altura — o mais
perto que esta foto chega dos 45% do vídeo. O corte portrait é 3:4 centrado,
para o telefone não perder metade da imagem no `object-fit: cover`.

Tal como no vídeo, a desaturação de 60% vai **gravada no ficheiro**: menos
crominância comprime melhor, e mantém os dois fundos com o mesmo tratamento.

```sh
ffmpeg -i "Saddleworth Moor-upscale.png" \
  -vf "crop=4096:2304:0:0,eq=saturation=0.60,format=rgb24" base-L.png
ffmpeg -i "Saddleworth Moor-upscale.png" \
  -vf "crop=2304:3072:896:0,eq=saturation=0.60,format=rgb24" base-P.png

for w in 2560 3200 4096; do
  ffmpeg -i base-L.png -vf "scale=${w}:-2:flags=lanczos" L-${w}.png
  cwebp -q 72 -m 6 -sharp_yuv L-${w}.png -o assets/img/moor-${w}.webp
done

ffmpeg -i base-P.png -vf "scale=1536:2048:flags=lanczos" P.png
cwebp -q 72 -m 6 -sharp_yuv P.png -o assets/img/moor-portrait-1536.webp
```

A qualidade 72 foi escolhida comparando **com o véu
aplicado**, que é como a página se vê. Aí q72 dá 44,9 dB de PSNR — acima dos
40 dB a que a diferença deixa de ser visível — e subir para q78 custa +172 KB
por 1 dB. O peso por visitante é um ficheiro só: 380 KB a 912 KB conforme o
ecrã, ou 491 KB no telefone.

### Logo e ícones

O logo original é um SVG de uma só cor (`#424642`) — trocar por `#FFFFFF` chega.
O símbolo é o `<polygon>` do ficheiro, 370×370 exactos na origem e perfeitamente
radial (círculo mínimo de raio 185,6 em 185,185), por isso inscreve-se num
círculo sem desperdício.

O `favicon.svg` é circular; o `apple-touch-icon.png` é **quadrado e opaco** de
propósito — o iOS aplica a sua própria máscara e ignora transparência, e um PNG
circular ficaria com cantos pretos no ecrã inicial.

A 32px o símbolo perde os raios finos e lê-se como uma mancha. A 64px (retina,
o caso comum) está bem. Resolver a sério exigiria uma variante do símbolo com
menos raios e mais grossos — decisão de marca, não de código.

## Verificado em

Chrome via Playwright, 1728×1080 e 390×844 (iPhone 13) — as duas pontas que
interessam, porque são as duas variantes que o `srcset` serve.

O `lpduarte.github.io` está autorizado no kit do Adobe Fonts e o IP da rede
local não está: para testar tipografia no telemóvel, usa o endereço publicado,
não o `python3 -m http.server`. Pelo IP a Museo falha em silêncio.
