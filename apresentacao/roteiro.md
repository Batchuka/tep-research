# Roteiro da defesa

Meta: cerca de 27 min de fala (o limite é 30). Os tempos são acumulados. Se aos 16 min você ainda não chegou ao slide 19 (Resultados), encurte os slides 12 a 18.

| Slide | Tema | Tempo | Acumulado |
|---|---|---|---|
| 1 | Capa | 0:30 | 0:30 |
| 2–9 | O problema | 8:00 | 8:30 |
| 10–18 | O que foi construído | 7:00 | 15:30 |
| 19–26 | Resultados | 9:00 | 24:30 |
| 27–29 | Conclusões, futuros, obrigado | 2:30 | 27:00 |
| 30–41 | Reserva (só nas perguntas) | — | — |

Slides de reserva: 30 índice da reserva (clicável), 31 código original em FORTRAN, 32 `tep-plant` em Rust, 33 modos de operação, 34 distúrbios, 35 função de custo, 36 ponto de operação ótimo, 37 custo por parcela, 38 malhas P e PI, 39 índice de preditividade (erro previsível, figura do Cap. 2), 40 velocidade e janelas, 41 containers. Nos dois slides de código, a imagem e o endereço abrem o repositório no navegador.

Navegação:
- **Ir:** os botões verdes ao lado do número do slide (por exemplo "Ponto ótimo" no slide 5) levam à reserva.
- **Voltar:** volta ao slide de onde você saiu. Funciona no Foxit e no Acrobat; em outros visualizadores, use **Alt + ←**.
- **Reserva:** em todo slide de reserva, leva ao índice da reserva (slide 30), que tem um link para cada slide. Funciona em qualquer visualizador.
- **Marcadores:** o PDF abre com o painel de marcadores (Capa, O problema, O que foi construído, Resultados, Fechamento, Reserva), cada parte com os seus slides. Em tela cheia o painel some; saia da tela cheia para usar.
- Teste tudo no computador da sala antes de começar.

---

## O que falar em cada slide

**1. Capa** — Nome, título, orientador. Uma frase: "o trabalho é sobre acompanhar se uma planta está sendo operada do jeito que foi decidido".

**2. Divisor: O problema** — Só passe.

**3. O artigo de Downs e Vogel** — Mostre os destaques: é o modelo de um processo químico industrial, feito para estudar controle *plant-wide*, com 12 válvulas para manipular e 41 medições. As sub-rotinas em FORTRAN eram enviadas a pedido; hoje estão no GitHub (botão "Código FORTRAN").

**4. TEP** — Downs e Vogel publicaram em 1993 uma planta química como problema aberto. Cinco unidades, reciclo e 20 distúrbios. Por isso é tão usado em controle *plant-wide*. Uso o Modo 1.

**5. Camadas de controle** — A pergunta clássica é como sintonizar uma malha. Mas, segundo Larsson e Skogestad, operar uma planta envolve decidir em camadas: otimização em horas, supervisão em minutos, regulatório em segundos. Mostre onde fica cada parte do laboratório: as malhas P e PI rodam dentro do `tep-plant`, junto do processo; o *historian* e o supervisor no Kubernetes ficam na camada supervisória, que é o foco do trabalho; a IHM mostra o veredito; o `tep-optimizer` é um projeto paralelo, fora do escopo. A planta conversa com o resto por OPC UA.

**6. P1 a P5** — Separo cinco problemas que se confundem no TEP. P1 é o ponto ótimo, P2 é a estabilidade (dentro do P1), P3 é o que manter constante, P4 é o controle. As respostas deles juntas são o que chamo de política de operação. Para o TEP, a literatura já respondeu P1 a P4. O que falta é o P5: verificar, com a planta em operação, se a política está sendo cumprida. **Aqui o P5 só avalia, não age.**

**7. De onde vem o Kubernetes** — Siga a figura da esquerda para a direita: primeiro uma aplicação por máquina; depois várias máquinas virtuais na mesma máquina; depois *containers*, mais leves. Com muitos *containers* em muitas máquinas, alguém precisa decidir onde cada um roda e recriá-lo quando cai: é o Kubernetes. Daí vem o mecanismo do próximo slide.

**8. Kubernetes** — A ideia central do Kubernetes é a mesma do P5: o usuário declara um estado desejado com o `kubectl`, o servidor de API guarda isso no etcd junto com o estado observado, e controladores comparam os dois; os nativos cuidam dos *pods*. Um tipo novo (CRD) pode ganhar um controlador próprio, o operador: é isso que o trabalho faz. Se o estado desejado for a política e o observado for a planta, temos o supervisor. O Kubernetes não é tempo real, mas a supervisão também não precisa ser: as malhas continuam na camada regulatória.

**9. Objetivo** — Leia o objetivo geral. Reforce a ênfase: Integração de Sistemas de Controle, ou seja, o foco é ligar as camadas. Aponte a caixa "Fora do escopo": escolher variáveis e sintonizar controladores não é o trabalho; P1 a P4 entram como dados; e o supervisor só avalia, não age. Cite rápido os cinco específicos; cada um vira uma parte da apresentação.

**10. Divisor** — Passe.

**11. Laboratório** — Siga os números da figura. A planta publica por OPC UA (1, 2); o *historian* coleta; o supervisor, dentro do Kubernetes, pede estatísticas ao *historian* a cada 30 s (3) e lê os recursos declarados (4); a IHM mostra o veredito (5). O UaExpert comanda velocidade e distúrbios (6). **A planta fica fora do Kubernetes; o cluster só observa.**

**12. Planta** — O FORTRAN original roda do início ao fim e só depois os dados são analisados. Supervisão precisa da planta em operação, então reescrevi em Rust, sobre um framework que desenvolvi (monjolo), para rodar de forma contínua e publicar os sinais por OPC UA. Dá para acelerar até cerca de 56 vezes.

**13. Historian** — Coleta a cada 100 ms e responde duas perguntas por HTTP: médias da janela e o índice de cada malha. O índice vem de um trabalho do CERN: se o erro da malha é previsível, a malha pode melhorar.

**14. Três recursos: `Plant`** — São três tipos de objeto novos no Kubernetes, ligados em cadeia. O primeiro é a planta: diz onde está o *historian*, qual política está ativa e de quanto em quanto tempo avaliar. Trocar de política é trocar o `policyRef`.

**15. Três recursos: `OperatingPolicy`** — A política do Modo 1: orçamento de custo, metas com tolerância, limites e os critérios de cada malha. Três avaliações ruins seguidas mudam o veredito.

**16. Três recursos: `CostFunction`** — O custo $J$ de Downs e Vogel escrito como 12 termos, cada um um coeficiente vezes as médias de sinais. **O ponto principal dos três slides: o código do supervisor não tem nada do TEP. Tudo o que é da planta está nestes arquivos.**

**17. Veredito** — Os quatro passos. Três avaliações ruins seguidas viram NonCompliant. O veredito fica no status da Plant, então o `kubectl` lê, a IHM lê, qualquer programa lê. O supervisor nunca escreve na planta.

**18. IHM** — A planta ao vivo e o painel do supervisor. Rápido.

**19. Divisor: Resultados** — "Agora, os resultados."

**20. A planta simulada e o modelo original** — Antes de supervisionar, confirmar que a planta é o TEP. No caso base, o custo dá 170,63 contra 170,61 publicado. No ótimo do Modo 1, o otimizador acha 114,32 contra 114,31 de Ricker. Como o ótimo depende do modelo inteiro, bater esse número é uma boa indicação de que a física foi reescrita corretamente.

**21. Os experimentos ao longo de 2026** — Explique a numeração: é um registro do projeto, em ordem cronológica, com observação, hipótese, intervenção, resultado e conclusão. De 1 a 11, a planta passou a funcionar (Cap. 3). De 12 a 24, versões anteriores da planta e a investigação da água de resfriamento; não entram na monografia. De 25 a 27, os únicos com o supervisor no Kubernetes: são os resultados que vêm agora.

**22. Experimento 25: IDV(6)** — Em operação normal, o veredito ficou Compliant em todas as avaliações. Liguei o IDV(6), a perda da alimentação de A. Com 0,98 h a vazão de produto sai da meta; com 1,15 h o custo estoura e o veredito vira NonCompliant. São cerca de 7 minutos de relógio. A ordem é a física (primeiro o produto, depois o custo) e a política não foi mudada.

**23. Experimento 26: seguimento do setpoint** — Mas olhando as malhas: o supervisor disse que estavam saudáveis com a pressão 131 kPa acima do setpoint. O índice do CERN supõe que a malha segue o setpoint; lá eles têm alarme de desvio, e aqui não tinha. Acrescentei o offset máximo e repeti. Aí a degradação aparece em ordem: níveis, produto, malhas, custo.

**24. A escala do índice das malhas** — Relendo os dados, vi que o código gravava 1 menos o índice. Uma deriva previsível parecia malha saudável. Refazendo o cálculo na escala certa, sem rodar de novo, o índice teria acusado o distúrbio em 0,6 h, antes do custo. Mas ele também acusa transitórios inofensivos. Conclusão: complemento, não substituto. **Diga isso com calma: o erro foi seu e quem o achou foi você.**

**25. Experimento 27: malhas PI em operação normal** — Abra com o motivo: com P, a planta não voltou do IDV(6), então passei as malhas para PI e rodei sem distúrbio, para achar o novo ponto. Depois siga os quatro passos do slide: (1) o PI segura a pressão no setpoint fechando a purga aos poucos, de 39 para 18 %; (2) sai menos gás, o inerte B se acumula e o nível do reator, que não tem malha, cai de 72 para 46 %; (3) abaixo de 45,8 %, o feixe de resfriamento fica descoberto e retira menos calor; (4) a temperatura sobe, a purga satura e a pressão passa de 3000 kPa, o limite de parada de emergência. Feche com a legenda: com P, a mesma planta converge em 40 h. Cuidados: a purga só satura no fim, não durante a deriva; não diga que a taxa de reação cai (a vazão de produto até sobe um pouco); e é parada de emergência, não explosão.

**26. Experimento 27: o que o supervisor viu** — **Este é o resultado mais importante.** Enquanto a planta se perdia, o custo caiu de 166 para 55 $/h, porque menos purga significa menos reagente perdido na conta de Downs e Vogel. Custo e malhas ficaram verdes até o fim. Só o limite de nível declarado na política percebeu, e tarde: o nível caiu 25 h dentro da faixa antes de cruzar. Um limite fixo avisa quando a variável chega lá, não quando ela começa a ir.

**27. Conclusões** — Leia a caixa: o objetivo foi atingido. Depois os quatro pontos, todos sobre o objetivo: (1) o Kubernetes observou e julgou, Compliant em operação normal e NonCompliant sob o IDV(6), com a política fixa; (2) as respostas de P1 a P4 que implementei tiveram problemas (a escala do índice, o orçamento calibrado num transitório, as malhas PI sem malha de nível); (3) o veredito só vale o que vale a política, e foi o que se viu no Experimento 27; (4) falta escolher, entre os métodos de P1 a P4, quais dão os melhores critérios. Diga o ponto 2 com calma e sem se diminuir: são problemas que os experimentos mostraram.

**28. Futuros** — Os quatro, na ordem: corrigir os erros encontrados; observar uma planta real do laboratório de controle do Ifes; avaliar sinais de sensores e atuadores, na linha da iniciação científica; e otimização em tempo real, recalculando o ponto de operação com a planta em operação.

**29. Obrigado** — Abra para perguntas.

---

## Perguntas prováveis

**Por que Kubernetes, e não um SCADA ou um CLP?**
O Kubernetes não substitui o CLP: as malhas ficam na camada regulatória. Ele entra na supervisão, que não precisa de tempo real. O ganho é o mecanismo de estado desejado contra observado: a política vira um objeto do cluster, editável com `kubectl`, e o veredito vira um status que qualquer programa lê. Johansson et al. (2022), da ABB, já usam Kubernetes com OPC UA em automação.

**E o tempo real?**
O supervisor avalia a cada 30 s e foi o bastante: pegou o IDV(6) em cerca de 7 min de relógio. Se a avaliação atrasar alguns segundos, a planta não sente, porque quem age são as malhas.

**Por que o supervisor não age sobre a planta?**
Foi uma escolha de escopo: neste trabalho o P5 só avalia. Agir exige salvaguardas e o operador no circuito; está nos trabalhos futuros, e a planta já expõe métodos OPC UA para isso.

**Funciona para outra planta?**
O código do supervisor não tem nada do TEP; bastaria escrever novos manifestos e apontar o historian para outro servidor OPC UA. O limite: um tipo novo de critério, como foi o offset máximo, exige mudar o código. A política é declarativa dentro do vocabulário que o supervisor conhece. E não testei com outra planta.

**Por que 30 s e janela de 60 s?**
É a escala de minutos da camada supervisória. Uma ressalva honesta: as janelas são medidas em tempo de relógio, e com a planta acelerada elas cobrem mais tempo de processo (slide 40). A correção é medir pelo `clock.t_h`.

**Por que Rust?**
A monografia não justifica isso de forma explícita; prepare a resposta com suas palavras. O que o texto sustenta: o monjolo foi feito para rodar modelos de forma contínua, quem escreve a planta trata só do processo e do controle, e a planta usa 15 MB de memória (slide 41).

**É mesmo um gêmeo digital?**
Não no sentido estrito, e o trabalho diz isso: gêmeo digital acompanha um sistema real, e o TEP não existe fisicamente. É uma versão digital que roda de forma contínua. Modelar uma planta do laboratório do curso e rodar o modelo ao lado dela é um dos trabalhos futuros.

**O gráfico da releitura é uma manipulação dos dados?**
Não: os dados gravados são os mesmos. Na escala do artigo, o índice é exatamente 1 menos o valor gravado, e o limite (0,5) foi calibrado do mesmo jeito que o 0,12 original. O gráfico mostra o que o supervisor teria visto com a escala certa, não o que ele fez. O que ele fez está no slide do Experimento 25 (22). Para explicar o que é erro previsível, use o slide 39 (botão "Erro previsível" no slide 24 ou no 13).

**Como o erro de escala passou?**
O limite (0,12) foi calibrado em cima dos valores que o código produzia, então tudo parecia coerente em operação normal. O erro só apareceu quando o índice não reagiu ao IDV(6) e fui comparar com o artigo. Como as rodadas estavam gravadas, deu para refazer o julgamento sem rodar de novo.

**Por que o orçamento é 170,6 $/h?**
É o custo do caso base. Em operação normal o custo medido foi 166,8 $/h, então ele precisa subir cerca de 2,3 % para estourar. Ressalva: a calibração foi feita no começo de um transitório lento, e o orçamento deve ser recalibrado com o custo de regime.

**Por que P funciona e PI não?**
Slide 38 (malhas P e PI na pressão). É uma leitura dos dados, não algo provado: com P a pressão tem folga e a planta se ajusta; com PI a purga absorve tudo, mas ela não controla o líquido no reator.

**O que é o ponto de operação ótimo?**
Slide 36. Entre todos os pontos que cumprem o Modo 1 e respeitam os limites, é o de menor custo J, em regime. Ricker (1995) achou esse ponto com o modelo completo: J cai de 170,6 para 114,3 $/h, principalmente porque se perde menos reagente na purga. No ótimo, 5 variáveis encostam num limite (a pressão vai até 2800 kPa); as outras 3 ficam num mínimo dentro da faixa. A política deste trabalho usa o caso base, e não o ótimo, porque perseguir o ótimo exigiria mexer na implementação e na sintonia das malhas.
