# ctf-solving

Writeups de máquinas CTF (Hack The Box e outras plataformas) com o raciocínio
por trás de cada passo, mais uma seção de detecção blue team: como cada ataque
seria identificado em um SIEM. 

## Propósito

Este repositório documenta minha prática de raciocínio ofensivo através de
CTFs. Mais do que registrar a solução de cada máquina, o foco está em mostrar
o processo de decisão — por que cada passo foi o próximo movimento, como li a
saída de cada ferramenta e que hipótese ela abriu.

Cada writeup fecha com uma perspectiva de blue team: como aquele ataque
geraria evidência e seria detectado do outro lado da mesa. Esse é o cruzamento
que me interessa — entender a ofensiva para saber defender melhor.

É um repositório separado do meu [homelab-soc](https://github.com/machadokalani/homelab-soc),
que documenta a infraestrutura de detecção que eu mesmo construo e opero
(Wazuh + Sysmon). Aqui, o objeto são simulações fornecidas por terceiros.

## Organização

- **`methodology/`** — playbooks de metodologia ofensiva, aplicáveis a
  qualquer máquina. `playbook-quick.md` é um checklist de campo para consultar
  durante um CTF; `playbook-methodology.md` é a metodologia completa, com o
  porquê de cada fase.
- **`writeups/`** — os writeups, organizados por plataforma. Cada máquina é
  uma pasta autocontida, com seu texto e suas imagens.

## Máquinas resolvidas

| Máquina | Plataforma | Dificuldade | Temas | Writeup |
|---|---|---|---|---|
| EM ANDAMENTO | [Ver](writeups/htb/appointment/appointment.md) |


## Ética e escopo

Todo o conteúdo aqui refere-se a máquinas de laboratório em plataformas de
treinamento, cujos termos permitem a publicação de writeups. As técnicas
documentadas destinam-se exclusivamente ao aprendizado em ambientes
autorizados. Nada neste repositório deve ser usado contra sistemas sem
autorização explícita.
