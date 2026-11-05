# Revisão técnica — Soup Engine para Canon EOS 5D Classic

**Escopo:** análise estática dos arquivos presentes no fork `neoonzy/soup-engine`, em 5 de novembro de 2026. **Não houve compilação, execução de testes nem validação em câmera física nesta revisão.** As classificações abaixo indicam prioridades de investigação, e não falhas comprovadas em uso real.

## 1. Estado atual

O código da 5D Classic apresenta uma implementação embarcada substancial:

| Área | Arquivos principais | O que existe |
| --- | --- | --- |
| Motor comum | `src/engine/` | Decodificação CR2/JPEG, codificação JPEG, curvas, transformação de cores, LUT, dithering e grão |
| Câmera | `src/ml/payload/film.c` e `start.S` | Seletor na tela, tarefas embarcadas, varredura de arquivos e processamento em fila |
| Integração | `src/tools/patch-film.py` | Verificações antes de aplicar um patch em `AUTOEXEC.BIN` conhecido |
| Desktop | `src/host/` | Utilitário `filmdev` e rotinas de autoteste |
| Simulação | `src/tools/camsim/`, `src/tools/simtest.sh` | Simulação das APIs Canon, leituras parciais, interrupções, verificação de saída e comportamento |
| Perfis | `src/stocks/` e `card/ML/FILM/` | Arquivos `.fs` para simulações de filmes |

### Acertos importantes

- **O RAW original é preservado pelo fluxo de desenvolvimento:** o resultado é gravado em outro arquivo JPEG.
- **Separação entre processamento e integração:** componentes de `engine/` são reutilizados no desktop e na câmera.
- **Restrições de hardware consideradas:** compilação para ARM946E-S, sem ponto flutuante em hardware; caminhos de processamento por pixel favorecem aritmética de ponto fixo.
- **Testes de equivalência:** a suíte de simulação tenta comparar a saída com uma referência calculada no desktop, inclusive byte a byte.
- **Defesas de ambiente:** há limites de arena, detecção de stack canary, limpeza dos arquivos temporários de spool e mecanismos para leituras curtas da mídia.
- **Dependências de cabeçalhos:** os Makefiles da versão embarcada e da versão host usam `-MMD -MP`, reduzindo o risco de compilar estruturas inconsistentes após alterações em `.h`.

## 2. Melhorias prioritárias

### P0 — Validar a identidade exata do executável-base antes do patch

**Arquivo:** `src/tools/patch-film.py` (aproximadamente linhas 245–280).

O script valida tamanho fixo (`221184` bytes), assinatura `gaonisoy` e algumas instruções/literais esperados. São salvaguardas úteis, mas **não equivalem à identificação completa do binário**. Dois binários com os mesmos trechos verificados ainda podem diferir em outras regiões utilizadas pelo patch.

**Recomendação:** exigir um SHA-256 conhecido do `autoexec.bin` de entrada (ou uma lista explícita de hashes aprovados), registrar o hash do artefato gerado e abortar se a origem não for reconhecida. Só adicionar novos hashes depois de validar a compatibilidade com a compilação correspondente. Nunca contornar uma incompatibilidade apenas ajustando a lista.

**Critério de aceite:** um arquivo com mesmo tamanho, assinatura e instruções nos pontos verificados, mas conteúdo diferente em outra região, deve ser rejeitado.

### P0 — Tornar a suíte de simulação reproduzível e impedir passes vazios

**Arquivos:** `src/Makefile`, `src/tools/simtest.sh`, `src/tools/camsim/camsim.c`.

O alvo `simtest` utiliza `$(firstword $(SAMPLES))`, mas **não existem amostras `.CR2` versionadas** na árvore analisada. O simulador também procura o executável-base em `vendor/magiclantern-5Dc/autoexec.bin`, ausente do repositório. Isso não é necessariamente um bug: são dependências externas, porém falta uma verificação de pré-requisitos que explique claramente o que está ausente antes da compilação.

O alvo `test-full` contém laços baseados em `$(SAMPLES)`; sem amostras, partes desses laços não são executadas. Portanto, um `test-full` concluído não assegura que o processamento com CR2 foi realmente exercitado.

**Recomendação:** adicionar um alvo `make doctor`; interromper `simtest` e `test-full` quando faltar a amostra que o teste exige; separar testes totalmente autônomos dos que precisam de dados externos; documentar como fornecer amostras de maneira legal e privada. Acrescentar GitHub Actions para os testes que não dependem de imagens ou binários não distribuíveis.

**Critério de aceite:** cada teste informa seus pré-requisitos, quantas amostras executou e falha explicitamente quando não puder exercitar suas verificações principais.

### P1 — Reforçar aritmética segura nos leitores de CR2/TIFF

**Arquivos:** `src/engine/cr2.c` (linhas 36–70) e `src/engine/se.c` (linhas 54–83).

Há operações como `tiff_typesize(e->type) * e->count`, `e->voff + i * sz` e checagens do tipo `r->pos + 4 > r->len`. **Essas expressões merecem validação de overflow antes da operação**, especialmente se houver entrada malformada ou valores de deslocamento próximos aos limites do tipo. Não afirmamos aqui que qualquer entrada específica provoque falha explorável.

**Recomendação:** usar checagens na forma `n > len - pos` após validar `pos <= len`, verificar multiplicações usando `count > SIZE_MAX / item_size` e calcular offsets em `size_t` com proteção contra overflow. Estender os testes com cabeçalhos TIFF truncados, contagens extremas e offsets fora do arquivo. Executar esses testes sob ASan/UBSan no host.

**Critério de aceite:** arquivos malformados são rejeitados com erro controlado, sem leitura fora dos limites.

### P1 — Validar os parâmetros do alocador de arena

**Arquivo:** `src/engine/se.c`, função `se_arena_alloc` (linhas 28–37).

A expressão `(a->used + (align - 1)) & ~(align - 1)` pressupõe que `align` seja potência de dois e que a soma não transborde. O código atual normaliza somente `align < 1`.

**Recomendação:** rejeitar valores de alinhamento inválidos e proteger a soma antes de arredondar; usar casos-limite em autotestes. A gravidade prática depende dos chamadores, que também devem ser auditados.

**Critério de aceite:** nenhum valor de `align` ou `n` força o alocador a retornar um endereço fora da arena.

### P1 — Evitar falsos positivos de arquivos RAW completos

**Arquivo:** `src/ml/payload/film.c`, função `on_cr2` (aproximadamente linhas 3030–3055).

O watcher considera arquivos com menos de 6 MiB como possivelmente ainda sendo gravados; os demais podem seguir para processamento. **Um tamanho mínimo não prova que a câmera terminou de escrever um CR2.** Uma fotografia ainda em gravação pode ultrapassar 6 MiB.

**Recomendação:** considerar tamanho/mtime estáveis entre varreduras e, se a API da câmera permitir, verificar conclusão da gravação por evento ou estado de captura; manter tratamento para suspensão e desligamento. Garantir que um arquivo incompleto não seja marcado permanentemente como concluído.

**Critério de aceite:** simulações de escrita lenta e fotografia durante o processamento não produzem JPEG incompleto nem impedem a tentativa posterior.

### P1 — Revisar a estratégia de recuperação dos arquivos com falha

**Arquivo:** `src/ml/payload/film.c`, função `on_cr2` (aproximadamente linhas 3080–3107).

O watermark avança inclusive após erro, para não reprocessar repetidamente um arquivo inválido. A decisão evita loops, mas pode deixar uma fotografia recuperável sem nova tentativa automática na mesma sessão.

**Recomendação:** registrar falhas por arquivo com motivo e número de tentativas; adotar um atraso progressivo ou permitir um comando explícito para reprocessar falhas; diferenciar erro permanente de leitura transitória. Conservar as proteções contra looping infinito.

**Critério de aceite:** uma falha temporária pode ser recuperada sem perder o controle da fila.

### P1 — Validar caminhos e limites de nomes de perfis

**Arquivo:** `src/ml/payload/film.c`, função `se_scan_stocks` (linhas 876–930).

Os nomes são obtidos de `ML/FILM/INDEX.TXT` e concatenados em `A:/ML/FILM/%s`. O arquivo de índice é controlado pelo cartão, portanto convém **restringir as entradas a nomes simples de arquivos `.fs`** em vez de permitir componentes de caminho, caracteres inválidos ou truncamento silencioso.

**Recomendação:** validar comprimento, extensão, separadores, `..` e o número máximo de perfis; reportar de maneira visível índices e arquivos rejeitados. Isso melhora robustez mesmo quando não há um ator malicioso.

### P2 — Automatizar qualidade, compilação e documentação

Atualmente a única workflow versionada em `.github/workflows/` é `camera-folders.yml`, voltada ao empacotamento dos diretórios. **Não foi encontrada uma workflow versionada para compilar ou executar os testes da 5D em cada PR.**

**Recomendação:** adicionar CI de `make all`, `make test`, lint de Python e verificações de código C que funcionem sem dependências proprietárias; documentar quais etapas continuam manuais. Em builds de distribuição, evitar disponibilizar artefatos que não passaram por validação apropriada.

### P2 — Melhorar a experiência de usuário

- Exibir mensagens de erro de processamento e permitir a retomada manual de um arquivo.
- Esclarecer na interface quando um efeito do perfil não é suportado no modo RAW.
- Disponibilizar informações sobre espaço livre no cartão e tempo de processamento, se as APIs forem confiáveis.
- Exibir diferenças entre processar a prévia JPEG e os dados RAW sem sugerir equivalência de qualidade.
- Adicionar documentação de criação, validação e distribuição de novos perfis `.fs`.

## 3. Divergências documentais a esclarecer

- A documentação original da 5D afirma que `AUTOEXEC.BIN` tem **215 bytes modificados**, enquanto o comentário de `src/Makefile` menciona **262 bytes**. Essa divergência precisa ser esclarecida a partir do diff do executável, sem assumir que uma das duas quantidades esteja correta.
- O cabeçalho de `src/ml/payload/Makefile` menciona `tools/patch-soupeng.py`, mas o script presente é `tools/patch-film.py`.
- Comentários do `patch-film.py` citam documentos como `docs/ML_DISPLAY.md` e `BUILDING.md`, que **não aparecem na árvore versionada da 5D**. Vale atualizar referências ou trazer documentação apropriada sem copiar conteúdos indisponíveis.
- Os scripts e alguns comentários descrevem mecanismos e comportamentos que devem ser confrontados com o firmware realmente compilado antes de publicar instruções de engenharia reversa como garantias de segurança.

## 4. Ordem de implementação recomendada

1. **Sem mexer no firmware:** verificações de hash, documentação reprodutível das dependências, alvos que rejeitam testes vazios e CI host.
2. **Endurecimento do código:** proteção contra overflow no parser e alocador, testes de entradas malformadas, validação de nomes de perfis.
3. **Confiabilidade embarcada:** provas de conclusão de gravação do CR2 e política de retry de fotografias com falha, validadas primeiro no simulador e depois em hardware.
4. **Usabilidade:** informar limitações do modo RAW e aprimorar acompanhamento de fila, retomada e erros.
5. **Somente após revisão e testes em câmera:** disponibilizar novos binários para uso.

## 5. Validação sugerida

No diretório `Canon-EOS-5D/src/`:

```sh
make all
make test

# Exigem dependências adicionais: consulte src/README.md
make test-full
make simtest
make arm
make payload
```

**Status desta revisão:** inspeção estática concluída; resultados da compilação, testes reais, desempenho, consumo de memória em hardware e compatibilidade binária **não verificados**. Não confundir recomendações com correções já aplicadas.

## 6. Referências de código

- [Motor e alocador](src/engine/se.c)
- [Parser de CR2](src/engine/cr2.c)
- [Pipeline de imagem](src/engine/pipeline.c)
- [Integração com a câmera](src/ml/payload/film.c)
- [Patch de Magic Lantern](src/tools/patch-film.py)
- [Makefile principal](src/Makefile)
- [Simulação e testes](src/tools/simtest.sh)
- [Documentação original de desenvolvimento](src/README.md)

Esta revisão não altera os arquivos executáveis ou o código embarcado. As recomendações devem ser validadas no fork antes de qualquer contribuição ou envio ao projeto original.
