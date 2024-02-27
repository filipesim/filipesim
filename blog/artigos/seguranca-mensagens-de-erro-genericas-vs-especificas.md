# Segurança: Mensagens de erro genéricas VS específicas

Hoje me deparei com um [vídeo](https://www.youtube.com/watch?v=SVO3tNH5fTo) do Lucas Montano do canal Lucas Montano em que ele fazia um react de um vídeo de um sênior que comentava sobre a preguiça de muitos programadores ao desenvolver sistemas.

Sabe aquela situação em que você preenche um formulário inteiro e ao clicar no bendito botão **"salvar"**, ao invés de um **retorno claro do problema**, recebemos aquela clássica mensagem: “Erro ao salvar”? Então... mais ou menos sobre isso.

## Mensagens de erro específicas

Lá no [minuto 5:20 do vídeo](https://www.youtube.com/watch?v=SVO3tNH5fTo&t=330s), o Lucas Montano do canal Lucas Montano fala algo que me chamou a atenção:

> “Tentar evitar ao máximo mensagens de erro genéricas… Sempre que a gente não detalha o erro pro usuário, a gente tá gerando trafego pro suporte, então sempre que o usuário ficou com uma dúvida ele vai entrar em contato com o suporte e suporte vai ser sempre um centro de custo pra empresa.”

Pois é… **mensagens de erro genéricas**, uma hora ou outra, vão se tornar um problema, seja pro cliente, pro suporte, pro dev ou, como na maioria dos casos, pra todos.

Quando o dev recebe uma tarefa mal escrita, ele acha ruim... mas faz o mesmo na hora de descrever mensagens de erro pro usuário.

Você não deve assumir que todos vão entender perfeitamente o funcionamento por trás de um campo cheio de validações e regras de negócio que você cuspiu no seu código hadouken.

<img src="https://raw.githubusercontent.com/filipesim/filipesim/master/blog/imagens/codigo-hadouken.png" alt="">

> Cansei de ver formulário com validação e retorno mais ou menos como nesse exemplo ai...

## Mensagens de erro genéricas

Mas na hora me lembrei de uma situação. Há algum tempo trabalhei em algumas melhorias de vulnerabilidade de segurança em cima de um teste de intrusão (famoso Pentest) feito em uma API.

No endpoint de autenticação da API, eram retornadas mensagens de erro mais específicas, e, acredite se quiser, um dos pontos recomendados era justamente o uso de mensagens genéricas.

**MAS COMO ASSIM?**

Calma javascripto, vou explicar...

Se retornarmos mensagens de erro especificas em um método de autenticação como no exemplo que mencionei, podemos estar, na verdade, guiando um hacker numa tentativa de invasão a nossa API.

Se liga só...

Imaginem que num primeiro momento, ao tentar autenticar na nossa API todos os parâmetros enviados estão errados, então retornamos a mensagem de erro: “Sistema inválido”.

<img src="https://raw.githubusercontent.com/filipesim/filipesim/master/blog/imagens/request-sistema-invalido.png" alt="" width="400px">

> Exemplo de request com um sistema inválido

<img src="https://raw.githubusercontent.com/filipesim/filipesim/master/blog/imagens/response-sistema-invalido.png" alt="" width="400px">

> Exemplo de response com um sistema inválido
 
Um "bandindinho" poderia realizar um ataque de força bruta na API até conseguir descobrir um sistema válido.

Request 1: Tento sistema X
Request 2: Tento sistema Y
Request 3: Tento sistema Z

Após diversas tentativas, ele consegue fazer isso:

<img src="https://raw.githubusercontent.com/filipesim/filipesim/master/blog/imagens/request-usuario-invalido.png" alt="" width="400px">

> Exemplo de request com um sistema válido

Note que a mensagem de erro muda para: **“Usuário inválido”**.

<img src="https://raw.githubusercontent.com/filipesim/filipesim/master/blog/imagens/response-usuario-invalido.png" alt="" width="400px">

> Exemplo de response com um sistema válido

Logo, podemos assumir que o sistema é **válido**, o que indica que ele acertou o sistema.

> "Opa, passei uma fase..." – pensa o hacker maligno nesse momento.

Assim, ele poderia repetir o processo com o parâmetro de usuário, até encontrar um válido:

Request 1: Tento usuário X
Request 2: Tento usuário Y
Request 3: Tento usuário Z

<img src="https://raw.githubusercontent.com/filipesim/filipesim/master/blog/imagens/request-senha-invalida.png" alt="" width="400px">

> Exemplo de request com um usuário válido

Ao acertar um usuário, a mensagem de erro também muda, agora para: **“Senha incorreta”**.

<img src="https://raw.githubusercontent.com/filipesim/filipesim/master/blog/imagens/response-senha-invalida.png" alt="" width="400px">

> Exemplo de response com um usuário válido

Agora era só repetir o processo para a senha ou então usar as informações de usuários descobertos para qualquer outra atividade maquiavélica, na tentativa de dominar a API… Muá-ha-ha-ha… Perdão, me empolguei…

<img src="https://raw.githubusercontent.com/filipesim/filipesim/master/blog/imagens/villain-laugh.gif" alt="" width="400px">

## O veredicto

Devo retornar mensagens de erro especificas ou genéricas? 

Como toda boa resposta de dev sênior: depende!

Geralmente **mensagens de erro genéricas** vão apenas deixar o usuário perdido e sem saber o que fazer pra resolver um problema, ele vai acionar o suporte que muitas vezes também não sabe o que fazer pra resolver e o suporte vai acionar a área de TI pra você tentar entender o que significa aquela mensagem de erro debugando o código linguição que você escreveu e nem lembra mais. Por isso a importância de retornar **mensagens de erro específicas**.

Por outro lado, em alguns casos, como no exemplo que vimos, **mensagens de erro específicas** podem abrir brechas de segurança que nem imaginamos. Nesses casos, precisamos retornar **mensagens de erro genéricas**. 

Escrevi esse artigo para compartilhar essa experiência, de repente você também nunca tinha pensado que uma simples mensagem de erro poderia se transformar em uma falha de segurança...

Me conta aí... você já tinha pensado por esse lado?