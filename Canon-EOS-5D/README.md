<div align="center">

# 🎞️ Soup Engine — Canon EOS 5D Classic

### Simulações de filmes processadas dentro da Canon EOS 5D original (2005).

<img src="doc/camera.gif" alt="Seletor de filmes na Canon EOS 5D Classic" width="620">

`beta` · `Canon EOS 5D Classic` · `firmware 1.1.1` · `GPL-2.0`

[← Voltar às câmeras](../README.md#câmeras-e-versões) · [Revisão técnica desta implementação](REVISAO-TECNICA.pt-BR.md)

**[Baixar pacote da EOS 5D no projeto original](https://github.com/jiijii0/soup-engine/releases/download/folders/Canon-EOS-5D.zip)**

</div>

## O que faz?

O Soup Engine oferece um seletor de simulações de filmes na tela traseira da Canon EOS 5D Classic. Depois de selecionar um perfil, a câmera pode gerar um segundo JPEG já processado para as novas fotografias, preservando o arquivo RAW original:

```text
IMG_1234.CR2        Fotografia original em RAW
IMG_1234.FLM.JPG    JPEG produzido pelo Soup Engine
```

![Exemplo dos treze estilos originais](doc/stocks.jpg)

O projeto inclui **13 simulações de filmes**, ainda em desenvolvimento. Os perfis são arquivos de texto `.fs` editáveis.

> [!CAUTION]
> **Experimental e não oficial.** O projeto modifica o `AUTOEXEC.BIN` utilizado pelo Magic Lantern na 5D Classic. Usar arquivos incompatíveis ou remover o executável com o boot flag ativo pode impedir a câmera de inicializar normalmente. Faça backup dos seus arquivos, use bateria carregada e siga estritamente os requisitos abaixo. A manutenção do RAW não elimina todos os riscos de executar código não oficial.

## Compatibilidade e requisitos

- **Câmera:** Canon EOS 5D *Classic* — o primeiro modelo, lançado em 2005; não é a 5D Mark II, III ou IV.
- **Firmware da Canon:** **1.1.1**.
- **Armazenamento:** cartão **CompactFlash (CF)**.
- **Pré-requisito:** [Magic Lantern compatível com a 5D Classic](https://builds.magiclantern.fm/5DC.html) instalado e funcionando.
- **Pacote:** arquivos da pasta [`card/`](card/) da EOS 5D Classic, jamais de outra câmera.

O pacote disponibilizado pelo projeto original não foi validado fisicamente durante esta revisão.

## Instalação

1. **Instale e teste o Magic Lantern.** Siga as orientações oficiais da compilação específica para a 5D Classic. Verifique que a câmera inicializa e que o Magic Lantern funciona antes de prosseguir.
2. **Faça backup do cartão e do `autoexec.bin` original do Magic Lantern.** Guarde uma cópia segura fora do cartão; ela será necessária para restaurar a instalação.
3. **Baixe o pacote específico da [Canon EOS 5D Classic](https://github.com/jiijii0/soup-engine/releases/download/folders/Canon-EOS-5D.zip)** e confira sua procedência.
4. **Copie o conteúdo de `card/` para a raiz do CompactFlash.** Substitua `AUTOEXEC.BIN` somente após guardar a cópia original. Mantenha os demais arquivos no mesmo caminho em que aparecem no pacote.
5. **Insira o cartão e ligue a câmera.** Verifique o comportamento da câmera antes de fotografar trabalhos importantes.

| Caminho | Função |
| --- | --- |
| `AUTOEXEC.BIN` | Executável do Magic Lantern adaptado para iniciar a integração |
| `SE.BIN` | Payload do Soup Engine |
| `ML/FILM/` | Perfis de simulação de filmes |

> [!IMPORTANT]
> Na 5D Classic, a integração substitui o executável que oferece o menu original do Magic Lantern. Enquanto o Soup Engine estiver em uso, o menu habitual do Magic Lantern não fica disponível da mesma forma. Para restaurá-lo, use o `autoexec.bin` que você salvou.

## Como usar

Pressione o botão **lixeira (Trash/Delete)** para abrir o seletor de filmes.

| Controle | Ação |
| --- | --- |
| **Disco de seleção** | Percorre as simulações |
| **Cima / baixo** | Alterna a origem do processamento |
| **SET** | Confirma o perfil escolhido |
| **Lixeira** | Fecha o seletor sem alterar a seleção |

O último perfil escolhido é lembrado entre inicializações pelo arquivo de configuração.

### Origem JPEG ou RAW

**`SOURCE: JPEG` (recomendado):** processa a prévia JPEG presente no CR2, geralmente em cerca de **20 segundos por fotografia**, segundo a documentação do projeto. É a opção mais rápida e a que apresenta suporte mais completo aos efeitos.

**`SOURCE: RAW`:** processa dados do sensor. Pode levar **minutos**, e alguns efeitos de cor não funcionam da mesma forma. Em particular, a documentação original observa limitações do efeito monocromático ACROS nesse modo.

> [!TIP]
> Use **Picture Style: Standard, Neutral ou Faithful**. O estilo *Monochrome* da câmera faz com que a origem JPEG seja monocromática, interferindo nos resultados dos perfis coloridos.

O texto **`FILM SPEED 50`** no seletor é apenas informativo: ele **não muda o ISO real da câmera**.

A câmera exibe o andamento e a posição dos arquivos na fila de processamento, por exemplo `FRAME 2 OF 3`. Você pode continuar fotografando, mas o tempo total depende do cartão, da origem escolhida e da quantidade de imagens.

## Personalização dos filmes

Os perfis ficam em `ML/FILM/`, em arquivos `.fs`. Eles podem ser editados manualmente ou com o editor web **[Soup Stocks](https://jiijii0.github.io/soup-stocks/)**. Para modificar a lista de filmes visíveis, respeite também o `INDEX.TXT` e a capacidade máxima do seletor. Guarde cópias dos seus perfis antes de editá-los.

O Soup Stocks oferece pré-visualização no navegador usando uma versão WebAssembly do motor, segundo a documentação original.

## Remoção e recuperação

Para restaurar o Magic Lantern, **recopie o `autoexec.bin` original** que você salvou antes da instalação, substituindo o arquivo adaptado. Na documentação do projeto, também é possível desativar o Soup Engine removendo apenas **`SE.BIN`** e mantendo o executável de inicialização no cartão.

**Se a câmera apresentar comportamento inesperado:** desligue-a; se não responder, remova a bateria com cuidado, retire o CompactFlash e, no computador, remova `SE.BIN` para impedir seu carregamento. Depois, revise os arquivos e recupere a instalação a partir do backup.

> [!CAUTION]
> **Não apague `AUTOEXEC.BIN` com o boot flag ativo.** Nesse estado, a câmera depende desse arquivo para inicializar corretamente. Restaurar a instalação exige manter um `AUTOEXEC.BIN` compatível no cartão.

## Código-fonte e desenvolvimento

O código está em [`src/`](src/), separado em:

- **`engine/`:** CR2, JPEG, correções de cor, curvas, grão, perfis e processamento de imagem.
- **`ml/payload/`:** integração embarcada com a câmera, inicialização, interface, fila e monitoramento.
- **`host/`:** ferramenta desktop `filmdev`.
- **`tools/`:** simulador, verificações, scripts de empacotamento e patch.
- **`stocks/`:** perfis de filme em texto.

Para conhecer os alvos de compilação, dependências e testes, leia [`src/README.md`](src/README.md). Exemplos executados a partir de `Canon-EOS-5D/src/`:

```sh
make all          # compila as ferramentas desktop
make test         # testes básicos do motor
make test-full    # verificações ampliadas; amostras necessárias
make simtest      # simulação embarcada; requer arquivos de referência
make arm          # verifica compilação ARM freestanding
make payload      # monta os arquivos para o cartão (dependências adicionais)
```

**Atenção:** o repositório não inclui os arquivos CR2 de amostra nem o executável-base original necessário a algumas etapas. Consulte os erros e os pré-requisitos antes de interpretar a falha de um teste como defeito no processamento.

A [revisão técnica](REVISAO-TECNICA.pt-BR.md) documenta melhorias recomendadas no código, nos testes e na instalação. Nenhuma dessas sugestões foi aplicada automaticamente ao firmware.

## Créditos e licença

Projeto original de **[@jiijii0](https://github.com/jiijii0/soup-engine)**, construído sobre o trabalho do [Magic Lantern](https://magiclantern.fm). O Soup Engine não é um projeto oficial da Canon nem do Magic Lantern.

Distribuído sob **GPL v2**, conforme [LICENSE](LICENSE). **Sem garantias.** Nomes comerciais de filmes são referências estéticas e não representam vínculos com seus fabricantes.

---

*Tradução e organização em português brasileiro feitas no fork [neoonzy/soup-engine](https://github.com/neoonzy/soup-engine); documentação técnica original e autoria preservadas.*
