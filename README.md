<div align="center">

# 🎞️ Soup Engine <sub><sup>beta</sup></sub>

### Simulações de filmes diretamente em câmeras Canon DSLR clássicas.

<img src="Canon-EOS-5D/doc/camera.gif" alt="Seleção de simulações de filmes na tela da Canon EOS 5D Classic" width="620">

`beta` · `GPL-2.0` · `5 modelos de câmera` · `Magic Lantern`

</div>

## O que é?

O **Soup Engine** é um projeto de código aberto que permite aplicar estilos inspirados em filmes fotográficos dentro de câmeras Canon DSLR compatíveis. Depois de escolher um perfil na tela da câmera, as novas fotografias podem gerar um **JPEG adicional, já processado**, ao lado do arquivo RAW.

```text
IMG_1234.CR2        Arquivo RAW original, sem alterações
IMG_1234.FLM.JPG    JPEG adicional com a simulação de filme
```

O RAW não é alterado pelo processamento do Soup Engine. O nome dos arquivos e as opções disponíveis variam conforme a câmera; consulte as instruções da versão correspondente.

![Comparação de treze simulações de filmes](Canon-EOS-5D/doc/stocks.jpg)

> [!WARNING]
> **Projeto experimental (beta).** O Soup Engine não é um firmware oficial da Canon nem um projeto do Magic Lantern. Instalar ou executar código não oficial em câmeras antigas envolve riscos. Faça backup do cartão, confira o modelo e a versão do firmware e siga as instruções específicas antes de instalar. O software é fornecido **sem garantias**.

## Câmeras e versões

Cada adaptação possui código, arquivos para cartão e procedimentos próprios. **Não misture arquivos de câmeras ou versões de firmware diferentes.**

| Câmera | Firmware | Adaptação |
| --- | --- | --- |
| [Canon EOS 5D Classic](Canon-EOS-5D/) | 1.1.1 | [@jiijii0](https://github.com/jiijii0) |
| [Canon EOS 5D Classic — Half Frame](Canon-EOS-5D-Half-Frame/) | 1.1.1 | [@mmarco25](https://github.com/mmarco25) |
| [Canon EOS 7D (original)](Canon-EOS-7D/) | 2.0.3 | [@thebakers-club](https://github.com/thebakers-club) |
| [Canon EOS 50D](Canon-EOS-50D/) | 1.0.9 | [@rebelt2i](https://github.com/rebelt2i) |
| [Canon EOS 700D / Rebel T5i / Kiss X7i](Canon-EOS-700D-Rebel-T5i/) | 1.1.5 | [@GitariDev](https://github.com/GitariDev) |

São **cinco modelos de câmera** e **seis variantes**, contando a versão Half Frame da 5D.

### Comece pela Canon 5D Classic

Se você usa a primeira Canon EOS 5D (de 2005), consulte o [guia em português da EOS 5D Classic](Canon-EOS-5D/README.md), com requisitos, instalação, uso, remoção e avisos de segurança.

Para entender o estado do código dessa adaptação, veja também a [revisão técnica da EOS 5D](Canon-EOS-5D/REVISAO-TECNICA.pt-BR.md).

## Como funciona?

1. **Integração com o Magic Lantern.** O suporte à execução na câmera depende de uma versão específica do Magic Lantern. Na 5D Classic, o Soup Engine utiliza um `AUTOEXEC.BIN` adaptado e um payload separado (`SE.BIN`).
2. **Motor de processamento em C.** O código reúne leitura de CR2, decodificação/codificação JPEG, transformação de cores, curvas e aplicação de estilos. Parte dele é compartilhada por ferramentas de desktop e pela câmera.
3. **Perfis editáveis.** Os filmes são descritos por arquivos de texto `.fs`, geralmente armazenados em `ML/FILM/` no cartão.
4. **Processamento local.** Na 5D Classic, o modo padrão usa a prévia JPEG já incluída no arquivo CR2; o modo RAW é mais lento e tem limitações próprias.

As adaptações mantêm cópias independentes do motor, ajustadas às restrições de cada câmera. Por isso, uma mudança numa pasta não atualiza automaticamente as outras.

## Ferramentas complementares

**[Soup Stocks](https://jiijii0.github.io/soup-stocks/)** — editor web de perfis `.fs`, com pré-visualização baseada no motor compilado para WebAssembly. Segundo o projeto, a imagem usada na edição permanece no navegador.

**[SoupDesk](https://github.com/thebakers-club/SoupDesk)** — aplicação de simulações de filmes no desktop (macOS), desenvolvida por [@thebakers-club](https://github.com/thebakers-club).

## Estrutura do repositório

```text
Canon-EOS-5D/                 Implementação da 5D Classic
Canon-EOS-5D-Half-Frame/      Variante Half Frame da 5D
Canon-EOS-7D/                 Adaptação para a 7D original
Canon-EOS-50D/                Adaptação para a 50D
Canon-EOS-700D-Rebel-T5i/     Adaptação para a 700D
.github/workflows/            Empacotamento das pastas por câmera
```

Cada diretório apresenta suas próprias instruções. A árvore da 5D Classic inclui `card/` (arquivos preparados para o cartão) e `src/` (código-fonte, ferramentas e simulador).

## Contribuições e novas adaptações

Novas adaptações são bem-vindas no [projeto original](https://github.com/jiijii0/soup-engine). Para desenvolver suporte a outra câmera, organize-o em um novo diretório `Canon-EOS-<modelo>/`, com código, arquivos necessários e instruções de instalação, uso e remoção. Prefira mudanças restritas à pasta da câmera.

O projeto original publica pacotes por câmera em suas [versões disponíveis no GitHub](https://github.com/jiijii0/soup-engine/releases). Os links de download referem-se ao **projeto original**; este fork não afirma manter pacotes próprios compilados ou verificados.

## Autoria e agradecimentos

O Soup Engine foi criado por [@jiijii0](https://github.com/jiijii0), com adaptações de [@mmarco25](https://github.com/mmarco25), [@thebakers-club](https://github.com/thebakers-club), [@rebelt2i](https://github.com/rebelt2i) e [@GitariDev](https://github.com/GitariDev).

O projeto depende do trabalho de engenharia reversa e desenvolvimento da comunidade do [Magic Lantern](https://magiclantern.fm). O Soup Engine **não é mantido pelo Magic Lantern**: problemas específicos do Soup Engine devem ser relatados no repositório do Soup Engine, informando a câmera e o firmware utilizados.

Os nomes dos filmes são referências estéticas, não reproduções exatas nem indicação de vínculo com seus fabricantes. As marcas pertencem aos seus respectivos titulares.

## Licença

Distribuído sob a **GNU GPL v2**. Consulte [LICENSE](LICENSE) e as licenças nas pastas das câmeras. O software é fornecido **sem qualquer garantia**, conforme os termos da licença.

---

*Documentação em português brasileiro mantida neste fork. Código, créditos e licenças originais preservados.*
