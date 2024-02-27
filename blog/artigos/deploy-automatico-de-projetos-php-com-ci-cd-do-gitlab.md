# Deploy Automático com CI/CD do GitLab

Há alguns meses, precisava melhorar o fluxo de deploy de alguns projetos na empresa. Passei um bom tempo batendo cabeça a respeito do tal do CI/CD. Já havia usado o Jenkins para subir uma aplicação automaticamente, mas não podia deixar de conferir a nova modinha de deploy. Em meio a tanto conteúdo, parecia que nada se aplicava a minha situação. Depois de fazer 1 milhão de perguntas pro nosso amigo GPT e consumir muito material na internet cheguei em uma solução inicial (provavelmente duvidosa, ou melhor, um work in progress, rs).

## Alguns detalhes sobre o meu cenário

Tinha em mãos um projeto legado feito em PHP 5 que utilizava o framework CakePHP 1.2.
O deploy era normalmente feito a moda antiga (Talvez não tão antiga quando arrastar os arquivos via FTP mas...) entrando no servidor, e dando o famoso git pull.

Sem inventar demais, precisava arrumar um jeito para que todo PR aceito na master, subisse automaticamente para o servidor.

Todo o processo está na documentação do GitLab, que é boa, mas muito confusa, sempre te levando pra lá e pra cá em artigos que parecem quase iguais. 

Aos trancos e barrancos, documentei a solução inicial para esse problema e quis compartilhar.

## Habilitar execução de ações GIT via token de acesso do GitLab

Pra começar, precisava conseguir executar comandos git no servidor, como **"git pull"**, sem que fosse solicitado usuario e senha. Para isso configurei um token de acesso que é usado pelo servidor para autenticação com o GitLab por OAuth2 ao invés de usuário e senha.

### Passo 1: Gerar um token de acesso

Dentro do projeto no GitLab, acesse:

> Configurações > Tokens de acesso

Siga o passo a passo da tela e guarde o token gerado.
> O token deve se parecer com: **bqSHTGgVgwdQXKPzetCV** (Token fictício)

### Passo 2: Configurar o token no servidor

No servidor em que deseja configurar o deploy automático, configure o token gerado.<br>
Se for um projeto que ainda não está clonado no servidor, use:<br>
```cmd
sudo git clone http://oauth2:bqSHTGgVgwdQXKPzetCV@sua.url/seu-projeto
```
Se for um projeto já existente, use:
```cmd
git remote set-url origin http://oauth2:bqSHTGgVgwdQXKPzetCV@sua.url/seu-projeto
```
Agora já deve ser possível executar qualquer comando GIT no repositório sem que seja exigido senha.

---

## Criar um runner para realizar tarefas CI/CD no GitLab

Agora, precisava configurar um tal de runner para executar os comandos que ia definir no arquivo de deploy automático. Falando de forma leiga, ele funciona como uma espécie de servidor, a partir dele as ações definidas no arquivo de configuração do deploy automático são executadas.

### Passo 1: Criar o Runner
O runner pode ser criado como um container Docker.
Defina onde esse container vai ficar, ele precisa estar ativo sempre que for feito o deploy, ele pode ficar numa máquina local, ou no próprio servidor em que o deploy acontecerá.

Crie o Runner em um container Docker com o comando abaixo:
```cmd
docker run -d --name gitlab-runner --restart always -v /var/run/docker.sock:/var/run/docker.sock -v gitlab-runner-config:/etc/gitlab-runner gitlab/gitlab-runner:latest
```

### Passo 2: Vincular o Runner ao GitLab
Existe um token que é disponibilizado pelo GitLab para vincular no runner criado. Nesse momento, será solicitado a URL do GitLab e o token de autenticação.

Para encontrá-lo, no GitLab acesse: 

> Área do Administrador > Visão geral > Runners

Então, execute o comando de registro:
```cmd
docker exec -it gitlab-runner gitlab-runner register
```

Doc: https://docs.gitlab.com/runner/register/index.html?tab=Docker#register-with-a-runner-authentication-token

Após o GitLab 16, esse processo será atualizado.

Doc: https://docs.gitlab.com/ee/ci/runners/new_creation_workflow.html

---

## Acesso ao servidor via SSH com Chave publica/privada

Agora precisava de acesso ao servidor de deploy via SSH. Precisamos configurar uma chave no runner e compartilhar essa chave com o servidor de deploy para que ele se autentique. Assim, não precisaremos informar senha nos deploys.

### Passo 1: Entrar no container do Runner
Execute o comando abaixo:
```cmd
docker exec -it gitlab-runner bash
```

### Passo 2: Trocar para o usuário do Runner
Execute o comando abaixo:
```cmd
su gitlab-runner
```

### Passo 3: Gerar a chave publica/privada para o usuário
Execute o comando abaixo:
```cmd
ssh-keygen -t rsa -b 4096 -C "seu_email@example.com"
```

### Passo 4: Enviar a chave publica para o servidor do deploy
Execute o comando abaixo:
```cmd
ssh-copy-id usuario.do.servidor@ip.do.servidor
```
> Informe a senha do usuario nesse momento para permitir o acesso.

Depois disso já deve ser possível acessar o servidor do deploy sem senha:
```cmd
ssh usuario.do.servidor@ip.do.servidor
```

---

## Criar a tarefa do CI/CD

### Passo 1: Criar o arquivo gitlab-ci
Criar um arquivo chamado .gitlab-ci.yml com a estrutura de exemplo abaixo:
```yml
stages:
  - deploy

deploy:
  stage: deploy
  script:
    - cat ~/.ssh/id_rsa.pub
    - ssh usuario.do.servidor@ip.do.servidor 'cd /pasta/do/seu/projeto && git pull'
  only:
    - master
```

Após isso, ao aceitar qualquer PR na master, todo o fluxo configurado deve ser executado e o deploy será feito automaticamente no servidor.

---

## Alguns links que consultei

**Gitlab Runner**
https://docs.gitlab.com/runner/install/
https://docs.gitlab.com/runner/configuration/

**Como configurar Chave PÚBLICA e Chave PRIVADA no SSH**
https://www.youtube.com/watch?v=7BEsfupYngE

**Gitlab CI/CD AutoDevops a nova modinha de deploy**
https://medium.com/marujosx/gitlab-ci-cd-autodevops-a-nova-modinha-de-deploy-274080ed35e5

---

## Conclusão

Quando vi tudo funcionando foi só alegria, mas é claro que essa é apenas uma primeira implementação do processo. No mesmo momento em que terminei, já surgiram dúvidas e pontos a aprimorar:

- Será que essa é a melhor forma de fazer o deploy de projetos PHP?
- Temos vários outros projetos, nesse caso, seria melhor criar um Runner para cada ou um para todos?
- Pensar em um fluxo de rollback...

De qualquer forma, queria compartilhar o que aprendi. Espero que ajude alguém com um problema parecido, ou que no mínimo de alguns insights de como tudo isso funciona.

Sinta-se a vontade para comentar!

Um abraço!