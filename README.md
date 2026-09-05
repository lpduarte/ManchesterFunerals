# Manchester Funerals — holding page

Página única servida em <https://manchesterfunerals.co.uk>. Sem build, sem
dependências: HTML e CSS num ficheiro, assets estáticos ao lado.

```
index.html              markup + CSS (inline, é uma página só)
favicon.svg             símbolo branco sobre círculo #424642
assets/img/moor-*.webp  fundo actual (foto), 3 larguras + variante portrait
assets/video/bg.mp4     fundo em vídeo — fora de uso, ver "Fundo"
assets/img/             logo, símbolo, poster do vídeo, ícones
```

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

O fundo é **temporário**. O vídeo original mostrava um vale que não é o
noroeste de Inglaterra — campos em faixas, casas dispersas, encostas
florestadas até ao cume: Europa Central, provavelmente os Beskids. Nada disso
se lê debaixo do véu, mas o ficheiro está errado para a marca.

Está no lugar dele uma fotografia de Saddleworth Moor, com o Dovestone
Reservoir ao centro, enquanto se gera um vídeo da região. O `bg.mp4` e o
`poster.webp` ficam no repo à espera dessa substituição.

### A fotografia

De Roger Cornfoot, 2017, SE0206 — a 3 km de Diggle, Oldham.
[Geograph 5485216](https://www.geograph.org.uk/photo/5485216), **CC BY-SA 2.0**.

A licença obriga a crédito visível na página (autor, título, ligação à obra,
ligação à licença) e a indicar as alterações. É o que faz o `.credit` no
rodapé. O ShareAlike faz a versão recortada herdar a mesma licença — a página
declara-o. **Se o fundo mudar, o `.credit` sai com ele.**

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

A qualidade 72 foi escolhida como o CRF do vídeo: comparando **com o véu
aplicado**, que é como a página se vê. Aí q72 dá 44,9 dB de PSNR — acima dos
40 dB a que a diferença deixa de ser visível — e subir para q78 custa +172 KB
por 1 dB. O peso por visitante é um ficheiro só: 380 KB a 912 KB conforme o
ecrã, ou 491 KB no telefone.

### Fundo em vídeo

53,8 MB (3840×2160, 34 Mbps) → 1,1 MB (1920×1080, 24 fps, 0,8 Mbps).

O corte original saltava no loop — ao fim de 12,5s as nuvens já se tinham
deslocado. O `xfade` funde o fim com o princípio: o corpo é `V[1.2:12.5]` e os
seus últimos 1,2s desvanecem para `V[0:1.2]`, portanto o vídeo começa e acaba
no mesmo ponto do material.

```sh
ffmpeg -i video-background.mp4 -filter_complex "
[0:v]eq=saturation=0.60,hqdn3d=2:1.5:4:4,fps=24,scale=-2:1080:flags=lanczos,format=yuv420p,split=2[x][y];
[x]trim=1.2:12.5,setpts=PTS-STARTPTS[rest];
[y]trim=0:1.2,setpts=PTS-STARTPTS[head];
[rest][head]xfade=transition=fade:duration=1.2:offset=10.1[out]" -map "[out]" \
  -c:v libx264 -profile:v high -level 4.0 -preset veryslow -crf 36 \
  -pix_fmt yuv420p -g 48 -movflags +faststart -an assets/video/bg.mp4
```

O `hqdn3d` é o que faz a poupança: sem ele, o mesmo CRF dá 7,5 MB. A erva ao
vento é conteúdo de entropia alta, e o denoise não se nota debaixo do véu.

O CRF 36 foi escolhido comparando crops 1:1 contra uma referência CRF 16 de
26 MB, **com o overlay aplicado** — é assim que a página se vê, e a esse nível
de escurecimento as duas são indistinguíveis.

O poster é o primeiro frame do resultado, para não haver salto ao arrancar:

```sh
ffmpeg -i assets/video/bg.mp4 -frames:v 1 poster.png
cwebp -q 76 -m 6 poster.png -o assets/img/poster.webp
```

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

Chrome via Playwright, 1728×1080 e 390×844 (iPhone 13).

Nota: `python3 -m http.server` não responde a *range requests*, por isso o
`<video>` não faz seek. Reproduz e faz loop na mesma; qualquer servidor a sério
não tem o problema.
