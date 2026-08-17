# LP Máquinas de Gelo — Bebedouros &amp; Cia

Landing page de captação para WhatsApp, destino do tráfego de busca paga da frente de máquinas de gelo.

Página estática, arquivo único, sem build, sem framework e sem dependência externa além da fonte do Google Fonts.

```
index.html     a página inteira (HTML, CSS e JS)
assets/        logo e fotos dos seis modelos, PNG com fundo transparente
```

---

## Como funciona

O visitante responde três perguntas (segmento, consumo, contato). A página calcula qual modelo atende o consumo informado e **abre o WhatsApp com a mensagem já escrita**, contendo nome, segmento, consumo em kg/dia e o modelo recomendado. O vendedor recebe o lead com o briefing pronto.

Quem informa quanto gasta com gelo comprado recebe também o cálculo de em quanto tempo a máquina se paga, feito sobre os próprios números.

---

## Configuração

Tudo fica no objeto `CFG`, no início do `<script>` no fim do `index.html`. Campo vazio simplesmente não carrega nada, sem quebrar a página.

| Campo | O que é |
| :--- | :--- |
| `whatsapp` | Número de destino, formato internacional só com dígitos (`5527XXXXXXXXX`) |
| `webhook` | URL que recebe o lead em POST JSON. Vazio desativa |
| `ga4` | `G-XXXXXXXXXX` |
| `googleAds` | `AW-XXXXXXXXXX` |
| `convLead` | `AW-XXXXXXXXXX/rótulo` — conversão **primária**, envio do seletor |
| `convZap` | `AW-XXXXXXXXXX/rótulo` — conversão **secundária**, clique no WhatsApp |
| `metaPixel` | ID numérico do pixel |
| `gtm` | `GTM-XXXXXXX`. **Se preenchido, desliga as tags diretas** e deixa tudo pelo dataLayer, evitando contagem dobrada |

> O número de WhatsApp precisa ser o mesmo que alimenta o CRM. Apontar para outro número faz o lead não ser registrado.

### Catálogo

Modelo, produção, reservatório, preço e foto ficam **em um único lugar**: o array `MODELOS`. Ele alimenta ao mesmo tempo a recomendação, o cálculo de retorno e os seis cartões da seção de modelos. Para mudar preço ou incluir modelo, edite só esse array.

---

## Rastreamento

O `dataLayer` é sempre alimentado, com GTM ou sem. Eventos:

| Evento | Quando | Campos |
| :--- | :--- | :--- |
| `lp_view` | carga | `gclid`, `utm_campaign` |
| `seletor_inicio` | primeira interação | — |
| `seletor_etapa` | a cada passo | `etapa` |
| `lead_form_submit` | envio do passo 3 | `segmento`, `consumo_kg_dia`, `modelo`, `valor_modelo`, `telefone_e164`, `gclid` |
| `whatsapp_click` | qualquer botão de WhatsApp | `origem_botao`, `tem_lead`, `modelo`, `gclid` |

Sem GTM, a página dispara direto: `generate_lead` e a conversão do Google Ads (com Enhanced Conversions, o telefone tem hash feito no navegador), mais `Lead` e `WhatsAppClick` no Meta.

**Nenhum evento de anúncio leva valor monetário, e isso é intencional.** Lead não é venda: mandar o preço da máquina no evento faria Google e Meta reportarem receita que não existe. As duas conversões do Google Ads estão configuradas como "não usar valor". O preço continua indo para o `dataLayer` e para o webhook do CRM, que é de onde a conversão offline lê o faturamento real. Não recolocar `value` nos eventos sem essa conversa.

O `gclid` é lido da URL, enviado no webhook e incluído na mensagem do WhatsApp, para permitir importação de conversão offline quando a venda fechar.

---

## Publicação

Qualquer host estático. Suba `index.html` e a pasta `assets/`.

1. HTTPS ativo. O Google Ads reprova destino sem certificado válido.
2. Ajuste o `<link rel="canonical">` no `<head>` para o domínio final.
3. Preencha o `CFG`.

### Cuidados ao editar

- **Não troque os PNG por JPG de fundo branco.** O herói e o cartão de resultado têm fundo escuro; a foto com fundo apareceria como um retângulo branco.
- **No passo 3, o `window.open` tem que ser a primeira instrução.** Ele só passa pelo bloqueador de pop-up do celular se acontecer dentro do gesto do clique. Webhook e conversão disparam logo em seguida.
- **Não remova a classe `js` do `<html>` nem a rede de segurança de 2,6 s** da animação de entrada. Sem elas, uma falha de script deixaria seções inteiras invisíveis.

---

## Pendências de conteúdo

Os três marcadores `[CONFIRMAR]` foram resolvidos com dados do cliente (17/08/2026). O FAQ agora afirma:

- **Instalação inclusa**, com taxa de deslocamento de R$ 39,90 na Grande Vitória (Vitória, Vila Velha, Serra e Cariacica). Fora dessas cidades, o frete é conferido no orçamento.
- **Garantia de 6 meses** na máquina de gelo, com assistência técnica própria.
- **Parcelamento no cartão**, com a condição variando por modelo. Sem boleto faturado no momento.

Duas informações seguem sem confirmação e por isso estão escritas de forma genérica, não estimadas: a **cobertura exata da garantia** (peças e mão de obra, atendimento no local ou em bancada) e o **número de parcelas da máquina de gelo**. Sobre um ticket de R$ 9 mil a R$ 35 mil, o parcelamento é argumento de fechamento e vale virar texto afirmativo quando o cliente confirmar.

> Atenção ao editar o herói: o title, a meta description e o selo do topo dizem "frete e instalação grátis". A taxa de R$ 39,90 é explicada só no FAQ, por decisão comercial. Mudar um lado sem o outro deixa a página se contradizendo.
