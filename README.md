# Rich media — banner HTML5 e e-mail marketing

Duas peças de produção para campanha de mídia paga, escritas para rodar em ambiente real de
veiculação: **sem build, sem framework, sem CDN**.

Tudo usa a identidade do TaskBio (projeto real, no ar em [taskbio.com.br](https://taskbio.com.br)).

| Peça | Arquivo | Formato |
|---|---|---|
| Banner retângulo médio | [`banner-300x250/index.html`](banner-300x250/index.html) | HTML único, 300×250 |
| Banner leaderboard | [`banner-728x90/index.html`](banner-728x90/index.html) | HTML único, 728×90 |
| E-mail de campanha | [`email/email-taskbio.html`](email/email-taskbio.html) | Tabela + CSS inline, 600 px |

![Banner 300x250](banner-300x250/preview.png)
![Banner 728x90](banner-728x90/preview.png)

---

## Decisões técnicas (e o motivo)

### 1. Animação só em `transform` e `opacity`

Banner é veiculado em milhares de dispositivos, muitos fracos, e a maioria das plataformas limita
peso e tempo de animação. `transform` e `opacity` são compostas pela GPU e não disparam *layout*
nem *paint*. Por isso nenhuma animação mexe em `width`, `top`, `margin` ou `box-shadow`.

```css
@keyframes rise{
  from{opacity:0;transform:translateY(10px)}
  to{opacity:1;transform:none}
}
```

### 2. HTML único, sem dependência externa

Cada banner é **um arquivo** que abre sozinho. Sem fonte externa, sem imagem, sem CSS separado.
Fonte é *system stack* (`-apple-system, Segoe UI, Roboto, Arial`), que existe em qualquer máquina e
não adiciona requisição. Isso mantém a peça abaixo de 5 KB — muito longe do limite usual de 150 KB.

A marca é feita em CSS: o "TB" é um `div` com `border-radius:50%`, não um PNG. Zero requisição,
zero *blur* em tela retina.

### 3. `clickTag` para o veículo trocar o destino

Todo comprador de mídia precisa apontar o clique para a URL da campanha sem editar o criativo. O
padrão é a variável global `clickTag`:

```html
<a href="https://taskbio.com.br"
   onclick="window.open(window.clickTag || 'https://taskbio.com.br','_blank');return false;">
```

O `return false` evita navegação dupla, e o fallback garante que funcione em preview local.

### 4. Tamanho fixo travado

`html, body` têm `width`/`height` fixos e `overflow:hidden`. Sem isso, o banner ganha barra de
rolagem quando o host renderiza num iframe menor — o erro mais comum em peça de rich media.

### 5. E-mail é outro planeta

Nada de flexbox, grid ou `gap`. O que funciona em cliente de e-mail:

- **tabela** para estrutura (`role="presentation"` para não confundir leitor de tela)
- **CSS inline** em cada elemento — o Gmail descarta `<style>` na maioria dos casos
- **largura fixa 600 px**, porque é o que cabe no Outlook e no painel de leitura
- **botão em tabela com `bgcolor`**, que renderiza no Outlook (que ignora `border-radius`)
- `media query` só para o que precisa mudar no celular

### 6. O bug que eu corrigi (e que quase ninguém vê)

A primeira versão numerava os itens com `<td width="40" height="40">`. Em tabela, **célula estica
junto com a linha**: o círculo virou uma elipse de 100 px de altura.

A correção é não confiar em dimensão de `td`:

```html
<td width="40" valign="top" style="width:40px;vertical-align:top;">
  <div style="width:40px;height:40px;line-height:40px;border-radius:20px;...">1</div>
</td>
```

`div` com largura e altura fixas e `line-height` igual à altura centraliza o número verticalmente em
qualquer cliente.

### 7. Acessibilidade e movimento

`@media (prefers-reduced-motion: reduce)` desliga todas as animações. Quem tem sensibilidade a
movimento não precisa ver o banner pulsando.

---

## Como testar

Basta abrir os arquivos no navegador. Ou servir a pasta:

```bash
npx serve .
```

**Para o e-mail**, navegador não é suficiente — ele é mais permissivo que cliente real. O teste
honesto é:

1. Enviar para uma caixa **Gmail** e uma **Outlook.com**
2. Conferir no celular (Gmail app e Outlook app)
3. Se possível, testar no Outlook desktop, que usa o motor de renderização do Word

---

## O que eu faria diferente em produção

- **Imagem de fundo** em vez de gradiente CSS, para fotos de produto (com `alt` e versão 2x)
- **Fallback estático** para clientes que bloqueiam animação
- **Múltiplos tamanhos** do mesmo criativo (`300x600`, `970x250`, `320x50`) — o mesmo layout não
  escala
- **Versionamento de nome** (`taskbio_300x250_v2`) para controle de cache no veículo
