# IA privada pronta

[![IA privada pronta](guia/assets/banner.jpg)](https://inematds.github.io/ia-privada-pronta/guia/)

**🇧🇷 [Português](README.md) · 🇺🇸 [English](README.en.md) · 🇪🇸 [Español](README.es.md)**

Guia completo: https://inematds.github.io/ia-privada-pronta/guia/

**Validação real em Linux:** execução sem rede externa, conversa com contexto e encerramento dos processos próprios confirmados. Os 14 testes de software passaram. O modelo leve errou o exercício de conta em documento; o resultado está registrado, sem trocar o gabarito. Veja [os resultados completos](verificacao/STATUS.md).

## O que é

Este kit gratuito põe uma IA para conversar com você no próprio computador. Ele serve para quem quer processamento local, sem uma empresa de IA receber cada pedido. Traz preparação do modelo, conversa, leitura de arquivo e um modo Linux com a rede bloqueada para o processo. Você precisa de Python, Ollama e memória suficiente para o modelo. Os testes reais e as limitações estão nesta página.

Este kit põe uma IA para conversar com você no próprio computador, sem provedor de inferência externo. O foco é manter o processamento local: serviço próprio em endereço local, nuvem desativada e, no Linux, um modo que remove o acesso de rede do processo. O modelo padrão é Llama 3.2 3B; ele mantém as restrições do próprio modelo.

**Você termina conversando com um modelo real.** O kit traz o lançador, o perfil do modelo, a preparação dos pesos, conversa com contexto, perguntas por arquivo, um modo Linux sem rede e uma verificação executável. Não precisa de chave, assinatura de modelo ou bibliotecas Python extras.

## Requisitos

- Git para clonar, ou [baixe o ZIP](https://github.com/inematds/ia-privada-pronta/archive/refs/heads/main.zip) e extraia a pasta.
- Python 3.10 ou superior: [instalador oficial](https://www.python.org/downloads/).
- [Ollama instalado](https://ollama.com/download), no Linux, macOS ou Windows.
- Para o modelo padrão: aproximadamente **2 GB de download**, **8 GB de RAM recomendados**, espaço livre para os pesos e recursos para o sistema. Contexto padrão 2048; o consumo real depende do runtime. GPU é opcional: CPU funciona, com desempenho diferente.
- No Linux, para o modo sem rede: `bubblewrap` e `iproute2`. O modo exige namespaces de usuário permitidos pelo sistema.

Verifique antes de começar:

```bash
python3 --version
ollama --version
```

No Windows, use `py` no lugar de `python3` se necessário. Depois de instalar o Python ou Ollama, abra um terminal novo. O kit não instala pacotes com privilégios nem muda o Ollama usado por outros projetos.

## Instalar e conversar

```bash
git clone https://github.com/inematds/ia-privada-pronta.git
cd ia-privada-pronta
python3 ia.py diagnostico
python3 ia.py preparar
python3 ia.py verificar
python3 ia.py conversar
```

`preparar` usa internet para baixar ou conferir os pesos do modelo. Depois, `verificar` e `conversar` usam o modelo local já baixado, sem baixar automaticamente outro modelo. Os pesos são armazenados pelo próprio Ollama, não dentro do repositório.

Na conversa, digite `/limpar` para reiniciar o contexto e `/sair` para encerrar. O kit conserva apenas algumas mensagens recentes em memória e não salva histórico. Ao sair, descarrega o modelo e encerra somente sua instância e seus processos filhos. O terminal ou o sistema operacional ainda podem registrar o que aparece na tela.

## Linux: conversa com a rede bloqueada para o processo

Depois de preparar os pesos, instale os dois recursos do sistema, se faltarem:

```bash
sudo apt install bubblewrap iproute2
bash sem-rede.sh verificar
bash sem-rede.sh conversar
```

O lançador cria uma rede separada com apenas a interface local `lo`. O modelo e o cliente continuam se comunicando dentro dela; não há interface para sair à internet. O computador continua conectado normalmente para os outros programas. Esse modo isola a rede, não os arquivos do usuário; não é uma máquina virtual.

Em macOS/Windows, a conversa local funciona pelo `ia.py`; o lançador sem rede é específico de Linux. Para uma conferência manual nesses sistemas, prepare primeiro, desconecte Wi-Fi/cabo e execute `verificar`. Não declaramos esse teste realizado em sistemas que não usamos.

## Pergunta única e arquivo

```bash
python3 ia.py perguntar "Escreva três ideias para organizar uma reunião curta."
python3 ia.py perguntar --arquivo exemplos/primeiro-pedido.txt
```

Para texto sensível, prefira `--arquivo` ou entrada padrão: argumentos digitados na linha de comando podem ficar no histórico do shell. As respostas são texto; o kit não executa comandos sugeridos pela IA.

## Como saber que funcionou

```bash
python3 ia.py verificar
```

A verificação envia uma pergunta real ao modelo (`17 + 25`) e espera `42`. Salva o resultado em `.estado/verificacao.json`, com o digest dos pesos, tokens gerados e interfaces da rede. Código de saída 0 significa que esse caso passou; 1 indica resposta diferente; 2 indica erro de preparação/execução. Uma resposta correta não comprova competência geral nem ausência de recusas.

Leia também [verificacao/STATUS.md](verificacao/STATUS.md): ele distingue testes de software de inferência realmente executada. O gabarito de um segundo exercício está em `exemplos/documento-ficticio.txt`.

## O que cada escolha significa

**Privacidade:** quem recebe os dados e onde a tarefa roda. Neste kit, o pedido vai somente à instância própria do Ollama em `127.0.0.1`. Nuvem e ferramentas externas ficam desativadas. O modo Linux sem rede acrescenta uma restrição do sistema operacional.

**Sem censura:** modelo publicado pelo mantenedor com menos recusas. Este termo não promete aceitar todo pedido, acertar toda resposta ou substituir revisão humana. O kit não adiciona filtros de temas e não altera automaticamente o modelo.

## Modelo, versão e memória

O perfil usa **`llama3.2:latest`**, conferido em 15/11/2026. `perfil.json` registra o digest esperado dos pesos para impedir uma troca silenciosa do modelo. Se o mantenedor mudar a tag e o digest deixar de bater, a execução para: confira a nova versão antes de atualizar `digest_esperado`. Não remova a conferência só para passar.

O perfil limita o contexto a 2048 e a resposta a 512 tokens. A conversa tem janela curta; documentos longos precisam ser divididos. O tempo da primeira resposta inclui carregar os pesos. Se houver falta de memória, use uma máquina com capacidade adequada ou escolha conscientemente outro modelo local, ajustando nome e digest. Não há fallback para nuvem.

## Comportamentos importantes

- Instância própria, porta exclusiva e nuvem desativada. Porta ocupada gera erro; nunca conectamos silenciosamente a outro serviço.
- Proxies herdados não recebem os pedidos e redirecionamentos HTTP são recusados.
- Nome de modelo remoto, pesos ausentes, digest diferente e resposta vazia interrompem a execução.
- Nenhum conector, busca na web, agente que executa ferramentas ou publicação automática.
- Privacidade deste fluxo não protege contra malware, administrador do computador, captura de tela, backups ou runtime comprometido.

## Testes de software

```bash
python3 -m unittest discover -s tests -v
```

Esses testes usam um runtime simulado e não contam como teste de modelo. O teste com IA real é `verificar`.

## Documentação das ferramentas

[Ollama: execução local e nuvem](https://docs.ollama.com/faq) · [API local de conversa](https://docs.ollama.com/api/chat) · [Modelo padrão](https://ollama.com/library/llama3.2).

Código do kit: MIT. Runtime e pesos seguem suas próprias licenças; os pesos não são redistribuídos aqui.

## Se você baixou o ZIP

Extraia a pasta. No Windows, abra a pasta que contém ia.py, clique na barra de endereço, digite powershell e pressione Enter. Execute os comandos Python do guia, usando py se necessário. Os atalhos preparar.cmd e conversar.cmd também funcionam com duplo clique. No Linux/macOS, abra Terminal e use cd para entrar na pasta extraída; o caminho depende de onde você salvou o ZIP. Não execute git clone se escolheu ZIP.

Runtime é o programa que executa o modelo; digest é a identificação exata dos pesos; namespace de rede é uma rede separada para o processo no Linux. Se houver mudança de digest, o erro mostra os valores esperado e encontrado: confirme a versão na página oficial da tag antes de decidir atualizar digest_esperado no perfil.json. Sem confirmação, mantenha a proteção.
