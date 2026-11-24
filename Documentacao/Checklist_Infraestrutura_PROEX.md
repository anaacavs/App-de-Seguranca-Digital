# Checklist de infraestrutura do PROEX

Rascunho de acompanhamento baseado no roteiro do trabalho de Infraestrutura de Servidores. Preencher apenas com evidências observadas; não registrar senhas, tokens, chaves privadas, dados pessoais ou conteúdo de `.env`.

**Atualizado em:** 24/11/2026  
**Aplicação:** aplicativo mobile PROEX, API Java/Spring Boot e MySQL.

## Identificação informada na Parte 1

- Instituição: Centro Universitário Assis Gurgacz.
- Curso/turma: Engenharia de Software, 4º semestre.
- Data que consta no documento da Parte 1: 03/11/2026.
- Integrantes: Ana Clara Cavasotto Polla; Gisela Talita Eberts; Guilherme Ribeiro Maciel; Bruno Martins Rauber; Matheus Leverentz De Lara.
- Observação: o PDF da Parte 1 contém capturas incorporadas como imagens. A extração de texto não as identificou; a inspeção visual confirmou os dados abaixo.

## Dados AWS/EC2 observados nas capturas da Parte 1

- Free Plan/custo e uso: captura de 03/11/2026 mostra US$ 100,00 em créditos restantes, 183 dias restantes (até 03/05/2027) e custo do mês de US$ 0,00.
- Instância: nome mostrado como `atv_final-Infra`, tipo `t3.micro`, estado `Running`.
- Zona de disponibilidade mostrada: `us-east-2c`; portanto, região `us-east-2`.
- Status check na captura: `Initializing`. Essa imagem não comprova que as verificações terminaram com sucesso; obter captura atual se for necessário provar saúde da instância.
- Sistema observado via SSH/`hostnamectl`: Ubuntu 26.04.1 LTS, kernel Linux 7.0.0-1014-aws, arquitetura x86-64; `whoami` retorna `ubuntu`.
- `df -h` em 03/11 mostra `/dev/root` com 6,7 GB, 2,1 GB usados, 4,6 GB disponíveis e 31% de uso. Isso é o filesystem visto pelo Ubuntu, não confirma o tamanho provisionado do EBS. O volume EBS não é legível nas capturas recebidas.
- A captura da instância atual em 24/11/2026 confirma `atv_final-Infra`, `t3.micro`, `Running`, `3/3 checks passed`, zona `us-east-2c` (região `us-east-2`) e IP público `18.116.161.154`.
- A captura de Elastic IP confirma `18.116.161.154` alocado e associado à instância atual. A chamada `GET /usuarios/me` nesse endereço, em 24/11/2026 às 16:47 GMT, retornou `401` do Nginx sem token, resultado esperado e evidência de que a rota responde.
- Armazenamento da instância atual: EBS raiz `/dev/sda1`, 8 GiB, `In-use` e `Attached`. A captura mostra criptografia como `No`. O `df -h` mostra o filesystem dentro do volume, não substitui essa informação do console.
- **Continuidade:** o Instance ID e o hostname privado da captura de 03/11 são diferentes dos da instância atual, embora o nome da instância seja igual. Tratar a EC2 de 03/11 como instância anterior; usar as capturas de 24/11 como evidência da hospedagem atual.
- Não incluir no relatório os IDs completos da conta, instância, volume, Elastic IP, associação, interface, VPC, Security Group nem os IPv4 pessoais usados para SSH. O IP público pode constar como endereço de demonstração.
- Antes da próxima demonstração, atualizar `EXPO_PUBLIC_API_URL` no ambiente local do Expo Go para `http://18.116.161.154`, reiniciar o servidor Metro/Expo e repetir o fluxo no app. Não versionar `.env`.
- **Conferência de continuidade pendente:** a captura SSH da Parte 1 mostra o hostname privado terminando em `38-110`, enquanto o servidor acessado posteriormente mostrou `43-75`. Confirmar no console, sem publicar IDs, se é a mesma EC2 após mudança/recriação de interface ou se houve substituição da instância. Se for outra instância, usar evidências atuais para os requisitos de hospedagem.

## Arquitetura observada até agora

```text
Cliente de teste / app
        |
        | HTTP público (porta 80; HTTPS ainda não confirmado)
        v
Security Group da AWS (regras ainda precisam ser verificadas)
        |
        v
EC2 Ubuntu 26.04.1 LTS
        |
        v
Nginx -> API Spring Boot (proex.service, Java, porta 8080)
        |
        v
MySQL (mysql.service)
```

O fluxo Nginx -> API é indicado pelos testes: `/usuarios/me` retornou `401` tanto pelo Nginx quanto diretamente pela porta 8080; sem token, `401` é esperado para essa rota protegida. O fluxo de cadastro, login, leitura do próprio perfil e logout também foi executado a partir do PowerShell no computador da integrante.

## Evidências por requisito

| Requisito | Estado | Evidência / pendência | Quem pode avançar |
|---|---|---|---|
| R01 — PROEX real e funcionalidade | Fluxo principal demonstrado; evidência sanitizada pendente | O PROEX foi executado no Expo Go com `EXPO_PUBLIC_API_URL` apontando para a API pública da EC2 via Nginx/HTTP 80. Cadastro e login funcionaram. O access log registrou `POST /usuarios` 201, `POST /login` 200, `GET /usuarios/me` 200 e `POST /logout` 204. A integrante informa que os testes mais recentes de login/perfil usaram `18.116.161.154`; a atualização do Expo Go nesse endereço precisa ser registrada nas evidências. A tela de verificação de e-mail aparece, mas envio/validação do código não estão implementados; o login ainda funciona sem verificar o e-mail. Descrever como limitação. | Você |
| R02 — Conta AWS / Free Plan / créditos | Capturado na Parte 1 | A captura mostra US$ 100 restantes, 183 dias (até 03/05/2027) e US$ 0,00 no mês, em 03/11/2026. Ocultar nome/conta no print final, se identificáveis. | Você insere captura |
| R03 — EC2 Linux | Capturas atuais recebidas | A instância atual aparece `Running`, `t3.micro`, zona `us-east-2c` (região `us-east-2`) e `3/3 checks passed`. Ubuntu 26.04.1 LTS está confirmado por SSH. Storage mostra EBS raiz de 8 GiB, anexado e em uso. Capturas da instância anterior de 03/11 devem ser identificadas como históricas; o ID da instância mudou. | Você insere capturas sanitizadas |
| R04 — Security Group | Captura atual recebida; confirmar associação à instância | A lista atual mostra três entradas IPv4: SSH/TCP 22 restrito a dois `/32` e HTTP/TCP 80 para `0.0.0.0/0`; não aparecem regras IPv6, 443, 8080 ou MySQL. Confirmar na aba Security da EC2 que este é o grupo associado à instância atual. `ss` mostra Java em `*:8080`, mas a regra não está aberta no grupo mostrado. Ocultar IDs do grupo e IPs administrativos. | Você registra |
| R05 — SSH e permissões | SSH básico feito na 1ª aula; `sudo` liberado, validação pendente | Conexão autenticada por SSH foi realizada e a chave privada permaneceu local. A integrante informa que agora pode executar `sudo`; confirmar com um comando administrativo de leitura e documentar a política de usuários. | Você |
| R06 — Código do PROEX na EC2 | Backend comprovado; cliente mobile demonstrado | `proex.service` executa `/opt/proex/backend-1.0.0.jar` na EC2. O cliente Expo Go executou no celular e consumiu a API pública da EC2 por meio do Nginx; o frontend mobile não precisa ser servido pela EC2 neste cenário, pois o backend real do PROEX está implantado nela. Guardar captura sanitizada do app e do fluxo. | Você |
| R07 — Nginx | Parcialmente comprovado | `sudo nginx -t` retornou que a sintaxe está ok e o teste da configuração foi bem-sucedido. Nginx escuta em `0.0.0.0:80`; serviço ativo e rota protegida respondeu através do Nginx. Falta guardar a configuração ativa sanitizada e confirmar se o backend pode ser limitado ao loopback. | Você |
| R08 — Inicialização automática | Configuração systemd comprovada; reboot pendente | A captura de `proex.service` mostra `User=proex`, `WorkingDirectory=/opt/proex`, `ExecStart=java -jar /opt/proex/backend-1.0.0.jar`, `Restart=on-failure`, `RestartSec=5`, `WantedBy=multi-user.target` e `EnvironmentFile=-/etc/proex/.env`. O `.env.example` enviado lista `SERVER_PORT`, `DB_URL`, `DB_USERNAME` e `DB_PASSWORD` sem valores preenchidos; é apenas um modelo, não prova a existência/permissões do `.env` real. Os três serviços também apareceram ativos e habilitados. Falta confirmar as permissões do arquivo real sem abrir seu conteúdo e fazer teste controlado após reinicialização da EC2. | Você, em horário seguro |
| R09 — Segurança / HTTPS | Pendente | A regra atual expõe HTTP/80 publicamente e não mostra HTTPS/443. O volume EBS aparece sem criptografia. Confirmar se há DNS apto a HTTPS; se não houver, documentar a limitação exigida pelo roteiro. Não usar credenciais reais em HTTP. | Você registra; HTTPS/DNS com o grupo |
| R10 — Logs e métricas | Parcial; duas coletas registradas | Nginx access log correlacionado com o fluxo do Expo Go: cadastro/login/perfil/logout. A captura mais recente, em 24/11/2026 às 17:18 UTC, mostra uptime de 1 dia e 6 horas, load average 0.00/0.02/0.00, RAM 908 MiB total e 311 MiB disponíveis, swap de 1.0 GiB com 512 MiB usados e `/` com 70% de ocupação (2.1 GB disponíveis). Journal confirma inicialização Spring/Tomcat e conexão Hikari/MySQL; também registrou o valor rejeitado de uma senha. A conta associada foi informada como excluída. `ApiExceptionHandler` foi adicionado ao código; o commit local `cdf7997` foi conferido. A integrante informa `BUILD SUCCESS` e push ao Git; ainda falta implantar na EC2 e confirmar no journal que a senha fictícia não aparece. Não anexar a captura bruta do journal. | Você |
| R11 — Incidente e recuperação | Pendente | Fazer interrupção temporária e reversível somente em horário combinado, registrar antes/durante/depois e validar recuperação. | Juntas, com colega administrando |
| R12 — Backup e restauração | Pendente | Fazer cópia externa de configuração e dados de teste; demonstrar restauração segura. Não usar `.env` nem dados sensíveis no relatório. | Juntas / colega administra |
| R13 — Acesso simultâneo | Executado; evidência recebida | Cinco consultas simultâneas de leitura a `GET /usuarios/me`, feitas do PowerShell local usando sessão autenticada; todas retornaram `HTTP 200`. A captura comprova os resultados. Registrar a data e a hora da execução no relatório; o horário não aparece na captura. É um teste leve, não teste de estresse. | Você registra |
| R14 — Relatório, apresentação e encerramento | Em andamento | Este checklist é a base. Falta relatório final com evidências, ensaio e inventário/plano de encerramento dos recursos. | Você organiza; todas apresentam |

## Registro do teste da API

**Objetivo:** verificar o fluxo de autenticação da API hospedada.  
**Ação:** pelo PowerShell local, enviar cadastro e login de uma conta fictícia pela URL pública; consultar `/usuarios/me` com o token de sessão; executar logout.  
**Resultado observado:** cadastro retornou o usuário de teste; login e consulta do perfil foram concluídos sem erro; logout foi chamado sem erro.  
**Interpretação:** o backend respondeu externamente e o perfil protegido foi consultado com sessão autenticada.  
**Evidência:** captura editada, ocultando totalmente a linha da senha e quaisquer tokens. Não incluir o e-mail se ele identificar uma pessoa.

Atualização da demonstração: o cliente foi executado no Expo Go com `EXPO_PUBLIC_API_URL` apontando para a API pública da EC2 via Nginx na porta 80. O cadastro abriu a tela de verificação de e-mail, mas o envio e a validação do código ainda não estão implementados. O login foi permitido mesmo sem verificação. Registrar como limitação funcional, não como falha de infraestrutura.

O access log confirmou os códigos do fluxo final: `POST /usuarios` **201**, `POST /login` **200**, `GET /usuarios/me` **200**, `POST /logout` **204**. As respostas anteriores `400` correspondem às tentativas de cadastro/login com JSON inválido ou senha curta; não são o resultado do fluxo final bem-sucedido.

## Coleta administrativa enviada em 23/11/2026

- `sudo nginx -t`: sintaxe ok; teste da configuração bem-sucedido.
- `sudo ss -lntp`: Nginx em `0.0.0.0:80`; SSH em `0.0.0.0:22` e `[::]:22`; Java em `*:8080`; MySQL em `127.0.0.1:3306` e `127.0.0.1:33060`; resolvedor DNS em loopback.
- Interpretação: MySQL está limitado ao próprio host. O backend Java aceita conexões em todas as interfaces segundo o listener; confirmar que o Security Group associado não permite 8080 e avaliar com a administradora limitar o bind a loopback, já que o Nginx está no mesmo servidor.
- `sudo systemctl status proex nginx mysql --no-pager`: os três serviços ativos e habilitados. O backend executa `/opt/proex/backend-1.0.0.jar` como Java, e o MySQL informa que está operacional.
- Memória indicada no status: Java 169.9 MB atuais, pico 259.3 MB; swap do Java 76.9 MB. MySQL 23.8 MB atuais, pico 452.9 MB; swap indicada 458.8 MB, pico 459 MB. A swap alta deve ser acompanhada, sobretudo pelo histórico de falta de memória.
- O status do backend inclui avisos/stack trace com conteúdo truncado. Não atribuir causa até examinar logs completos de forma sanitizada.
- `uptime` às 22:03 UTC: servidor ativo há 10h46; médias de carga 0.00, 0.01 e 0.00.
- `free -h`: RAM total 908 MiB, usada 486 MiB, disponível 422 MiB; swap total 1.0 GiB, usada 570 MiB. Registrar como fotografia pontual; acompanhar swap/RAM por causa do histórico de falta de memória.
- `df -h`: partição `/` com 6.7 GB, 4.6 GB usados, 2.1 GB disponíveis (70%). Não limpar arquivos sem identificar sua função.
- Nginx access log: além do fluxo bem-sucedido, houve respostas 401 sem autenticação, 404 na raiz e 400 em tentativas malformadas. Há também requisições externas não reconhecidas, incluindo payloads inválidos e sondagens para `/`; isso mostra tráfego não solicitado comum em serviço público, mas não permite concluir que houve comprometimento.
- Journal do `proex`: inicialização normal do Tomcat na porta 8080 e conexão do pool Hikari com MySQL. Avisos de JSON inválido coincidem com tentativas 400. Uma validação de senha curta gerou `MethodArgumentNotValidException` com o valor rejeitado incluído na mensagem; a senha deve ser tratada como exposta e essa captura bruta não pode ir ao relatório. Investigar a configuração/código de logging para não registrar valores de campos sensíveis.
- Correção iniciada no código local: `ApiExceptionHandler` intercepta `MethodArgumentNotValidException`, retorna mensagem genérica HTTP 400 e não registra a exceção nem inclui valores rejeitados. A API atualmente hospedada continua sem essa correção até uma nova compilação e implantação. Validar no servidor com uma senha fictícia e curta após publicar a nova versão; o valor digitado não deve aparecer no journal.

## Coleta de métricas enviada em 24/11/2026

- `date -u`: 24/11/2026 às 17:18:05 UTC.
- `uptime` às 17:18:09 UTC: uptime de 1 dia e 6 horas; 1 usuário conectado; load average de 0.00, 0.02 e 0.00.
- `free -h`: RAM total 908 MiB, usada 596 MiB, livre 114 MiB, buff/cache 314 MiB, disponível 311 MiB; swap total 1.0 GiB, usada 512 MiB e livre 511 MiB.
- `df -h /`: raiz `/dev/root`, tamanho 6.7 GB, usados 4.6 GB, disponíveis 2.1 GB, ocupação 70%.
- Interpretação: carga baixa no instante medido. Havia 311 MiB de RAM disponível e cerca de metade da swap ocupada; registrar como fotografia pontual e acompanhar, sem concluir falha ou apagar arquivos. A raiz tinha 2.1 GB disponíveis.
- Evidência: captura dos quatro comandos enviada pela integrante; ocultar o hostname privado se desejado. Legenda deve indicar horário UTC e que a coleta não representa desempenho contínuo.

## Coleta somente de leitura que você pode fazer por SSH

A primeira aula também solicitou `whoami`, `hostnamectl`, `free -h`, `df -h` e `ip -br addr`. Considere essa atividade concluída; recupere a captura original para o relatório, se ainda existir. A lista abaixo complementa a coleta para o roteiro maior.

Execute no servidor, sem `sudo`, e salve os resultados sem credenciais:

```bash
whoami
hostnamectl
uptime
free -h
df -h
ss -lnt
systemctl is-active proex nginx mysql
systemctl is-enabled proex nginx mysql
```

Não abra nem copie `.env`. Não pare ou reinicie serviços durante esta coleta.

## Como registrar a coleta de métricas e logs (R10)

Para cada evidência, anote: **objetivo**, **ação/comando**, **resultado observado**, **interpretação** e **nome da captura**. Separe o que o comando mostrou do que você concluiu a partir dele.

| Comando | O que mostra | O que registrar |
|---|---|---|
| `uptime` | Tempo desde o último boot, usuários conectados e médias de carga do sistema. | Horário da coleta, tempo ligado e médias de carga. É uma fotografia, não prova estabilidade ao longo do tempo. |
| `free -h` | RAM e swap em formato legível. | Total e disponível de memória, swap usada. Para disponibilidade, observe `available`, não apenas `free`. |
| `df -h` | Tamanho e ocupação dos sistemas de arquivos. | A linha montada em `/`: tamanho, usado, disponível e percentual. |
| `sudo tail -n 20 /var/log/nginx/access.log` | Requisições HTTP recentes atendidas pelo Nginx. | Linhas do horário do teste: método/caminho, status HTTP e tamanho. Oculte IP de origem antes de compartilhar. |
| `sudo journalctl -u proex -n 40 --no-pager` | Mensagens recentes do backend Java gerenciado pelo systemd. | Linhas relevantes com horário e nível (`INFO`, `WARN`, `ERROR`). Um `WARN` não prova falha; correlacione com a operação e o status HTTP. Oculte dados pessoais ou segredos. |

**Como interpretar o status HTTP:** `2xx` indica que a requisição foi concluída; `4xx` normalmente indica entrada inválida, falta de autenticação ou recurso inexistente; `5xx` indica falha no servidor ou em uma dependência. Interprete o código junto com o endpoint e a ação. Exemplos do projeto: `401` em `/usuarios/me` sem token é esperado; `201` no cadastro indica criação; `200` no login/perfil indica sucesso; `204` no logout indica conclusão sem corpo de resposta.

**Capturas recomendadas:** (1) terminal com `uptime`, `free -h` e `df -h`; (2) linhas do Nginx próximas aos testes; (3) apenas mensagens relevantes do journal. Use nomes como `R10-metricas.png`, `R10-nginx-acesso.png` e `R10-backend-log.png`. Não inclua senha, token, conteúdo do `.env`, IP pessoal ou linhas irrelevantes. Acrescente uma legenda em cada captura com objetivo, resultado e interpretação.

## Itens para os quais é necessário acesso ao console AWS ou coordenação do grupo

- Confirmar/recuperar capturas da 1ª aula de Free Plan e EC2; completar região, volume e status checks se ainda faltarem.
- Confirmar que as alterações do Security Group foram salvas, ocultar os IPs administrativos nas evidências e verificar todas as regras IPv4/IPv6; não expor 8080 nem a porta do MySQL.
- Executar `sudo nginx -t`, guardar configuração ativa do Nginx sem segredos e coletar logs necessários; agora a integrante pode fazer essa verificação com o `sudo` liberado.
- Executar teste de reboot, incidente reversível, backup e restauração apenas em horário seguro e combinado, para evitar interromper o uso do servidor compartilhado.
- Remover ou redefinir a conta de teste criada para a demonstração; a senha apareceu em captura compartilhada anteriormente.

