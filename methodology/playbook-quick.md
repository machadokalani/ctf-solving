# Playbook Rápido — Metodologia de CTF

Bússola de raciocínio para qualquer máquina.

Princípio geral: cada fase produz a informação que orienta a próxima. Não
pular etapa, pois ter pressa na enumeração vira parede na exploração.

---

## 1. Preparação
Garantir que a base é confiável antes de tirar qualquer conclusão.
- Ambiente de ataque no ar e conectado ao alvo?
- Estado confiável? (relógio sincronizado, sessão limpa) — resultado tirado
  de base instável não vale nada.

## 2. Reconhecimento
Mapear toda a superfície exposta antes de formar hipótese.
- O que o alvo expõe? (portas, serviços, versões)
- Começar largo, depois aprofundar no que apareceu.
- Cada serviço encontrado é uma porta de entrada em potencial — e o conjunto
  deles é o roteiro do box. Não escolher um favorito cedo demais.

## 3. Enumeração
Para cada ponto exposto, extrair o máximo de informação antes de atacar.
- Qual a interface natural deste serviço? (como se "conversa" com ele)
- O que ele revela sem esforço? (versão, banners, conteúdo público,
  configuração padrão)
- Enumerar do geral para o específico — afunilar até o dado concreto.
- O que este achado me permite perguntar em seguida? Cada informação abre a
  próxima pergunta.

## 4. Exploração
Transformar informação em acesso.
- Qual o vetor mais barato primeiro? (acesso anônimo, credencial padrão,
  má configuração — antes de partir para o complexo)
- A entrada confia cegamente no que eu forneço? (todo campo/parâmetro que eu
  controlo é um vetor potencial)
- Não procurar a bala de prata: encadear fraquezas pequenas. Um achado que
  parece inútil sozinho pode ser o elo que liga os outros (attack chain).
- Nenhum indicador isolado é veredito — é a combinação que abre o caminho.

## 5. Pós-exploração
Consolidar e aprofundar o acesso obtido.
- Passar de "li/capturei algo" para "tenho uma sessão real e controlo".
- O que encontrei se reusa em outro lugar? (credenciais, chaves, padrões)
- Qual o próximo nível de privilégio, e o que me leva até ele?
- Localizar o objetivo (flag/dado-alvo) nos lugares prováveis.

## 6. Documentação
Registrar enquanto está fresco — parte do método, não um extra.
- Capturar o raciocínio, não só os comandos: por que cada passo foi o próximo.
- Evidência dos momentos decisivos.
- Fechar o ciclo: o que levo deste box para o próximo.

---

Lembrete de mão trocada: para cada técnica ofensiva aqui, existe o outro lado
— como ela seria detectada. Esse elo (o que vira alerta, que log gera) está
no apêndice blue team do `playbook-methodology.md`. Pensar nos dois lados é o
diferencial.
